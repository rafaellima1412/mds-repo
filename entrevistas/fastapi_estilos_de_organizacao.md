# Três formas de organizar um projeto FastAPI

Mesmo projeto fictício (uma API de "empresas" e "contatos", tipo CRM) organizado
de 3 jeitos diferentes, para comparar lado a lado.

---

## 1. FLAT — tudo em um pacote só, um arquivo por tipo

Estrutura mais simples: sem camadas, sem separação por feature. Cada arquivo
guarda **um tipo de coisa** (todos os models juntos, todos os routers juntos etc).

```
meu_projeto/
├── main.py              # cria o FastAPI(), inclui os routers
├── .env
└── app/
    ├── __init__.py
    ├── config.py         # <- config do servidor
    ├── database.py        # engine/session do SQLAlchemy
    ├── models.py          # TODOS os modelos ORM num arquivo só
    ├── schemas.py          # TODOS os DTOs Pydantic num arquivo só
    ├── routers.py          # TODAS as rotas num arquivo só (ou routers/company.py, routers/contact.py)
    └── crud.py             # funções de acesso ao banco
```

**`app/config.py`**
```python
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    environment: str = "dev"
    port: int = 8000
    database_url: str

    model_config = SettingsConfigDict(env_file=".env", extra="ignore")


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

**Uso (`app/database.py`)** — import direto, sem indireção:
```python
from app.config import get_settings

engine = create_engine(get_settings().database_url)
```

**Quando faz sentido:** protótipo, MVP, script que virou API, projeto de 1 dev,
poucas entidades. Cresce mal — `models.py` e `routers.py` viram arquivos de
milhares de linhas.

---

## 2. EM CAMADAS — Clean Architecture (`api-desafio`)

Organização **horizontal**: cada pasta é uma camada técnica. A regra de
dependência aponta sempre para dentro (`domain` não conhece ninguém).

```
api-desafio/
├── main.py                              # entrypoint (uvicorn main:app)
└── src/api_desafio/
    ├── domain/                              # regras de negócio puras
    │   ├── entities/weather_record.py
    │   ├── repositories/weather_repository.py   # interface (porta)
    │   └── exceptions.py
    │
    ├── application/                         # casos de uso
    │   ├── ports/weather_provider.py            # interface (porta)
    │   └── use_cases/
    │       ├── fetch_and_store_weather.py
    │       └── list_weather_records.py
    │
    ├── infrastructure/                      # implementações concretas
    │   ├── config/settings.py                   # <- config do servidor (escondida aqui)
    │   ├── database/session.py
    │   ├── database/models.py
    │   ├── database/repositories/sqlalchemy_weather_repository.py
    │   ├── external/openweather_provider.py
    │   └── di/container.py                       # injeção de dependência
    │
    └── presentation/                        # HTTP e CLI
        ├── api/routers/weather_router.py
        ├── api/schemas/weather_schemas.py
        └── cli/fetch_weather_cli.py
```

**`infrastructure/config/settings.py`**
```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    database_url: str
    openweather_api_key: str

    class Config:
        env_file = ".env"
```

**Uso** — nunca é importado direto por `domain`/`application`; só `infrastructure`
o lê, e entrega os valores já prontos via `container.py` (injeção de dependência):
```python
# infrastructure/di/container.py
from dependency_injector import containers, providers
from api_desafio.infrastructure.config.settings import Settings

class Container(containers.DeclarativeContainer):
    settings = providers.Singleton(Settings)
    weather_provider = providers.Factory(OpenWeatherProvider, api_key=settings.provided.openweather_api_key)
```

**Quando faz sentido:** domínio complexo, regras de negócio que precisam ser
testadas isoladamente de banco/HTTP, times grandes, projeto de vida longa,
alta chance de trocar peças (ex: trocar Postgres por outro banco).

---

## 3. MEIO-TERMO — organização por feature (`crm_back`)

Organização **vertical**: cada feature tem sua própria pasta com tudo que
precisa (`routers.py`, `schemas.py`, `actions.py`, `validators.py`,
`exceptions.py`). Não existe separação `domain/application/infra` global —
só um padrão de porta/adapter pontual pra integrações externas.

```
crm_back/
├── main.py                          # entrypoint, FastAPI(), CORS, Sentry, inclui routers
└── rental/
    ├── config.py                        # <- config do servidor (na raiz do pacote)
    ├── dependencies/db.py                   # engine/session, get_db()
    ├── models/                              # um arquivo por entidade
    │   ├── company.py
    │   └── user.py
    ├── application/gateway/                 # SÓ as interfaces de integrações externas
    │   ├── ITokenGateway.py
    │   └── ICalendarGateway.py
    ├── infra/gateway/                        # implementações dessas interfaces
    │   ├── TokenGateway.py
    │   └── CalendarGateway.py
    └── apis/                                 # <- uma pasta por feature
        ├── company/
        │   ├── routers.py
        │   ├── schemas.py
        │   ├── actions.py
        │   ├── validators.py
        │   └── exceptions.py
        ├── contact/
        │   ├── routers.py
        │   ├── schemas.py
        │   └── ...
        └── opportunity/
            └── ...
```

**`rental/config.py`**
```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class RentalSettings(BaseSettings):
    ENVIRONMENT: str = "dev"
    PORT: int = 8000
    DATABASE_URL: str

    model_config = SettingsConfigDict(env_file=".env", extra="ignore")


settings = RentalSettings()   # instância única, criada no import
```

**Uso** — import direto do singleton, de qualquer lugar do projeto, sem `Depends`:
```python
# rental/dependencies/db.py
from rental.config import settings
engine = create_engine(settings.DATABASE_URL)
```

**Quando faz sentido:** projeto com várias entidades/features que crescem em
paralelo (times diferentes mexendo em `company` e `opportunity` ao mesmo
tempo), mas sem a complexidade de manter 4 camadas globais. É o estilo mais
comum em APIs FastAPI de produção de porte médio.

---

## Resumo visual

|                       | Flat                                    | Em camadas (Clean Arch)                                  | Meio-termo (por feature)                      |
| --------------------- | --------------------------------------- | -------------------------------------------------------- | --------------------------------------------- |
| Eixo de organização   | por **tipo de arquivo**                 | por **camada técnica**                                   | por **feature/domínio**                       |
| Onde mora o config    | `app/config.py`                         | `infrastructure/config/settings.py`                      | `rental/config.py` (raiz do pacote)           |
| Como é acessado       | `Depends(get_settings)`                 | injetado via `container.py` (DI)                         | import direto do singleton `settings`         |
| Isolamento do domínio | nenhum                                  | total (`domain` não importa nada)                        | parcial (só gateways externos têm interface)  |
| Onde crescer dói      | `models.py`/`routers.py` viram gigantes | curva de aprendizado, mais arquivos p/ 1 feature simples | pode duplicar lógica entre features parecidas |
| Bom para              | protótipo, MVP                          | domínio complexo, times grandes, longa vida              | maioria dos projetos reais de porte médio     |
