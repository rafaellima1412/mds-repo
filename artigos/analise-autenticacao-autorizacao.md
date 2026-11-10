# Análise do Sistema de Autenticação e Autorização
**Projeto:** `61310_backend_b2b_distribuidor_revenda` (FastAPI)

---

## 1. Visão geral da arquitetura

O microsserviço **não implementa login próprio**. A autenticação é delegada a um
serviço externo chamado **"Login Único"**, que roda sobre **AWS Cognito**
(confirmado pela URL `*.auth.us-east-1.amazoncognito.com` em `.env_local`).
Este backend é responsável apenas pela **autorização** (RBAC), a partir das
informações contidas no JWT emitido pelo Login Único.

```
Usuário → Login Único (Cognito) → JWT (access_data com roles)
                                        │
                                        ▼
                        Este backend (valida token e roles)
```

### Componentes principais

| Componente               | Arquivo                                          | Responsabilidade                                                   |
| ------------------------ | ------------------------------------------------ | ------------------------------------------------------------------ |
| Endpoint de refresh      | `src/infra/router/endpoint/auth.py`              | Repassa `refresh-token` ao Login Único                             |
| Decorator de autorização | `src/infra/middleware/ValidateRolesDecorator.py` | Decodifica o JWT, monta `CurrentUser`, valida permissão            |
| Regras de acesso         | `src/domain/CurrentUser.py`                      | Lógica de RBAC (`validate_access`)                                 |
| Enums de RBAC            | `src/domain/enum/Role/*.py`                      | `RoleModule`, `RolePermission`, `RoleActions`, `RoleSystem`        |
| Chamada M2M              | `src/infra/gateway/UserGatewayLoginUnico.py`     | OAuth2 client_credentials para consultar usuários                  |
| API Key                  | `src/infra/middleware/ValidadeKeyMiddleware.py`  | Valida `x-api-key` via AWS API Gateway (chamadas server-to-server) |

### Fluxo de autorização (`validate_role`)

1. Extrai `Authorization: Bearer <token>`.
2. Decodifica o JWT.
3. Descompacta o campo `access_data` (base64 + zlib) → contém `roles` por
   ambiente (`b2b_acer_canais_local/dev/qas/...`) e dados da empresa (CNPJ).
4. Monta `CurrentUser` e injeta em `request.state.current_user`.
5. Verifica se o usuário tem alguma associação (`GetAssociationByEmailUsecase`).
6. Chama `CurrentUser.validate_access(permission, action, module)`:
   - `roles["*"] == "*"` → admin geral;
   - `"*:*"` no módulo → admin do módulo;
   - senão, procura a permissão (`"person:gpud"`) e checa se a ação está
     contida na string de ações liberadas.

### Formato de permissão no token

```
"<recurso>:<ações>"     ex: "person:g p u d", "product:d g notifies p u"
```
Ações: `g`=GET, `p`=POST, `u`=UPDATE, `d`=DELETE, `*`=admin, além de ações de
negócio (`aprovar`, `importar`, `gestor`, `distribuidor`, `revendedor`).

---

## 2. Erros/problemas encontrados

### 🔴 2.1 [CRÍTICO] JWT decodificado sem verificar assinatura

**Onde:** `ValidateRolesDecorator.py` e `GetRoles.py`

```python
payload = jwt.decode(token, options={"verify_signature": False})
```

**Problema:** o token era aceito **sem checar assinatura, emissor (`iss`),
audiência (`aud`) ou expiração (`exp`)**. Isso significa que qualquer JWT
com o formato esperado (payload em Base64 + campo `access_data` válido)
seria aceito como legítimo, **mesmo que não tivesse sido emitido pelo
Cognito**. Um atacante que soubesse o formato de `access_data` (base64 +
zlib de um JSON) poderia montar um token próprio, se auto-atribuir a role
`"*": "*"` (admin) e ter acesso total à API, sem nunca ter passado pelo
Login Único.

Além disso, o bloco `except jwt.ExpiredSignatureError` nunca era realmente
efetivo, pois sem verificação de assinatura o PyJWT não valida `exp` (a
menos que isso seja pedido explicitamente).

**Correção aplicada:** criado `src/infra/security/TokenService.py`, que
busca a chave pública real do Cognito via **JWKS** (com cache) e valida
assinatura (`RS256`), `iss`, `aud` (quando configurados) e `exp`
(via `options={"require": [...]}`).

```python
class TokenService:
    _jwks_client: PyJWKClient | None = None

    @classmethod
    def _get_jwks_client(cls) -> PyJWKClient:
        if cls._jwks_client is None:
            if not settings.COGNITO_JWKS_URL:
                raise ServiceError(status_code=HTTPStatus.INTERNAL_SERVER_ERROR,
                                    message="COGNITO_JWKS_URL não configurado")
            cls._jwks_client = PyJWKClient(settings.COGNITO_JWKS_URL, cache_keys=True)
        return cls._jwks_client

    @classmethod
    def decode(cls, token: str) -> dict:
        signing_key = cls._get_jwks_client().get_signing_key_from_jwt(token)
        decode_kwargs = {"algorithms": ["RS256"], "options": {"require": ["exp", "iss", "sub"]}}
        if settings.COGNITO_CLIENT_ID:
            decode_kwargs["audience"] = settings.COGNITO_CLIENT_ID
        if settings.COGNITO_ISSUER:
            decode_kwargs["issuer"] = settings.COGNITO_ISSUER
        return jwt.decode(token, signing_key.key, **decode_kwargs)
```

**Novas variáveis de ambiente necessárias** (`Settings.py` / `.env`):

```
COGNITO_JWKS_URL=https://cognito-idp.<regiao>.amazonaws.com/<user_pool_id>/.well-known/jwks.json
COGNITO_ISSUER=https://cognito-idp.<regiao>.amazonaws.com/<user_pool_id>
COGNITO_CLIENT_ID=<app_client_id_do_login_unico>
```

> ⚠️ **Ação necessária do time:** eu não tenho acesso ao `user_pool_id`/região
> reais do Cognito do "Login Único", então preenchi essas variáveis como
> **opcionais** para não quebrar o boot da aplicação. É preciso obter esses
> valores com o time responsável pelo Login Único e configurá-los em todos
> os ambientes (`local`, `dev`, `qas`, `hml`, `prd`) antes de considerar essa
> correção completa em produção. Enquanto `COGNITO_JWKS_URL` não estiver
> configurado, `TokenService.decode` falha com erro claro (500) em vez de
> aceitar token inválido silenciosamente — ou seja, "falha fechada", não
> "aberta".

---

### 🟠 2.2 Código duplicado de decode/descompactação

**Onde:** a mesma lógica de `base64.b64decode` + `zlib.decompress` +
`json.loads` existia, quase idêntica, em `ValidateRolesDecorator.py`
(`__decrypt_and_decompress`) **e** em `GetRoles.py`
(`__decrypt_and_decompress`).

**Correção:** extraída para `TokenService.decompress_access_data`, usada
nos dois lugares. Além disso, agora qualquer falha ao descompactar
`access_data` gera um `ServiceError` claro (422) em vez de um erro genérico
não tratado.

---

### 🟡 2.3 Loop redundante/confuso em `CurrentUser.validate_access`

**Onde:** `src/domain/CurrentUser.py`

```python
# user comum
for module in self.roles:
    user_has_module = any(role_module in module for module in self.roles)
```

**Problema:** o `for module in self.roles:` externo não tem efeito algum —
a variável `module` dele nunca é usada dentro do `any(...)`, que declara
sua **própria** variável de loop, também chamada `module` (shadowing). Ou
seja, a mesma expressão `any(role_module in module for module in self.roles)`
era recalculada `len(self.roles)` vezes à toa. Funcionalmente não gerava
resultado errado, mas era ineficiente e confuso de manter/ler.

**Correção:**

```python
# user comum
user_has_module = any(role_module in module_key for module_key in self.roles)
```

> Nota: a checagem usa `in` (substring), não `==`. Isso significa que, por
> exemplo, `role_module="geral"` também "casaria" com uma chave de módulo
> `"geral_x"`. Mantive o comportamento original (não alterei a semântica),
> mas registro aqui como ponto a validar com o time: o ideal seria checar
> igualdade exata (`role_module == module_key`) caso não exista a intenção
> de correspondência parcial entre módulos.

---

### 🟡 2.4 API Key validada contra o AWS a cada requisição

**Onde:** `ValidadeKeyMiddleware.py`

```python
response = client.get_api_key(apiKey=config.API_KEY_ID, includeValue=True)
```

**Observação (não corrigida neste momento, apenas sinalizada):** a cada
requisição autenticada por API Key, o serviço faz uma chamada síncrona
(`boto3`, bloqueante) ao API Gateway para buscar o valor atual da chave.
Isso adiciona latência de rede e um ponto de falha externo em toda
requisição, além de misturar chamada **síncrona** (`boto3`) dentro de um
`handler` **assíncrono** do FastAPI (pode bloquear o event loop sob carga).
Sugestão para evolução futura: cachear o valor da API Key por alguns
minutos (ex.: `functools.lru_cache` com TTL ou Redis) e/ou mover a
validação para o próprio API Gateway (usage plans), evitando repetir essa
checagem na aplicação.

---

## 3. Resumo das alterações aplicadas

| Arquivo | Alteração |
|---|---|
| `src/infra/security/TokenService.py` | **Novo.** Decodificação/validação segura do JWT via JWKS + descompactação do `access_data` |
| `src/infra/settings/Settings.py` | Adicionadas `COGNITO_JWKS_URL`, `COGNITO_ISSUER`, `COGNITO_CLIENT_ID` |
| `src/infra/middleware/ValidateRolesDecorator.py` | Usa `TokenService` em vez de `jwt.decode(..., verify_signature=False)`; remove função duplicada de descompactação |
| `src/application/usecase/auth/GetRoles.py` | Idem acima |
| `src/domain/CurrentUser.py` | Remove loop externo redundante em `validate_access` |

## 4. Pendências / próximos passos recomendados

1. **Obter e configurar** `COGNITO_JWKS_URL`, `COGNITO_ISSUER` e
   `COGNITO_CLIENT_ID` reais do Login Único em todos os ambientes.
2. Rodar a suíte de testes (`test/unit/test_roles_valid.py`) — notei que
   esse arquivo importa `CurrentUser` de
   `src.application.dto.CurrentUserDTO`, um caminho que **não existe** mais
   no código atual (a classe está em `src.domain.CurrentUser`). Esse teste
   provavelmente já está quebrado/desatualizado independentemente desta
   correção e merece ser revisado pelo time.
3. Validar com o time de infraestrutura se todas as chamadas a este
   microsserviço passam obrigatoriamente por uma camada (API Gateway/
   Cognito Authorizer) que já validava assinatura — se sim, a correção
   deste documento adiciona uma segunda camada de defesa (defesa em
   profundidade); se não, ela **fecha uma brecha real de autenticação**.
4. Avaliar cache/TTL para a validação de API Key (item 2.4).
