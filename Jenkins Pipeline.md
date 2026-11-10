Jenkins Pipeline (ou simplesmente "Pipeline" com um "P" maiúsculo") é um conjunto de plugins que suporta a implementação e integração de pipelines de entrega contínua em Jenkins. 

Um pipeline de entrega contínua (CD) é uma expressão automatizada do seu processo para obter software do controle de versão através de seus usuários e clientes.

Cada alteração no seu software (comprometido no controle de origem) vai através de um processo complexo a caminho de ser lançado. Este processo envolve construir o software de maneira confiável e repetível, bem como progredindo o software construído (chamado de "construir") através de vários estágios de teste e implantação.


### Fundamentos de Declarativos do Pipeline[](https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com#declarative-pipeline-fundamentals)

Na sintaxe de Declarativa Pipeline, o `pipeline`bloco define todo o trabalho realizado ao longo de todo o seu Pipeline.

Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any // 1
    stages {
        stage('Build') { // 2
            steps {
                // 3
            }
        }
        stage('Test') { // 4
            steps {
                // 5
            }
        }
        stage('Deploy') { // 6
            steps {
                // 7
            }
        }
    }
}
```

| etapa | ação                                                                                  |
| ----- | ------------------------------------------------------------------------------------- |
| 1     | Execute este Pipeline ou qualquer um de seus estágios, em qualquer agente disponível. |
| 2     | Define o estágio "Construir".                                                         |
| 3     | Realize algumas etapas relacionadas ao estágio "Construir".                           |
| 4     | Define o estágio "Teste".                                                             |
| 5     | Realize algumas etapas relacionadas ao estágio "Teste".                               |
| 6     | Define o estágio "Deploy".                                                            |
| 7     | Execute algumas etapas relacionadas ao estágio "Implantar".                           |
### Fundamentos do Pipeline Roteirizado[](https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com#scripted-pipeline-fundamentals)

Na sintaxe do Scripted Pipeline, um ou mais `node`blocos fazem o trabalho principal ao longo de todo o Pipeline. Embora este não seja um requisito obrigatório de Sintaxe do Pipeline com script, confinando o trabalho do seu Pipeline dentro de um `node`bloco faz duas coisas:

1. Agenda as etapas contidas no bloco para executar adicionando um item para a fila de Jenkins. Assim que um executor é livre em um nó, o passos serão executados.
    
2. Cria um espaço de trabalho (um diretório específico para esse particular Pipeline) onde o trabalho pode ser feito em arquivos verificados a partir do controle de origem.  
    **Cuidado:** Dependendo da sua configuração do Jenkins, alguns espaços de trabalho podem não ser automaticamente limpo após um período de inatividade. Consulte os bilhetes e discussão vinculada de [JENKINS-2111 para](https://issue-redirect.jenkins.io/issue/2111) mais informações.
    

Jenkinsfile (Roteirizado Pipeline)

```groovy
node {  // 1
    stage('Build') { // 2
        // 3
    }
    stage('Test') { // 4
        // 5
    }
    stage('Deploy') { // 6
        // 7
    }
}
```

|     |                                                                                                                                                                                                                                                                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Execute este Pipeline ou qualquer um de seus estágios, em qualquer agente disponível.                                                                                                                                                                                      |
| 2   | Define o estágio "Construir". `stage`blocos são opcionais no Scripted Pipeline sintaxe. No entanto, a implementação `stage`blocos em um Scripted Pipeline fornece visualização mais clara de cada `stage`Subconjunto de tarefas/passos na interface do usuário do Jenkins. |
| 3   | Realize algumas etapas relacionadas ao estágio "Construir".                                                                                                                                                                                                                |
| 4   | Define o estágio "Teste".                                                                                                                                                                                                                                                  |
| 5   | Realize algumas etapas relacionadas ao estágio "Teste".                                                                                                                                                                                                                    |
| 6   | Define o estágio "Deploy".                                                                                                                                                                                                                                                 |
| 7   | Execute algumas etapas relacionadas ao estágio "Implantar".                                                                                                                                                                                                                |

executar um comando shell --> sh 
O comando `sh 'make'` varre o projeto em busca de arquivos modificados e os compila de forma otimizada.
`sh 'make clean'` preparam o espaço removendo resquícios de compilações anteriores antes de iniciar uma nova build.
`sh 'make check'` ou `sh 'make test'` para disparar a suíte de testes automatizados.

`Jenkinsfile`é um arquivo de texto que contém a definição de um Jenkins Pipeline e é verificado no controle de origem. Considere o seguinte Pipeline que implementa um pipeline básico de entrega contínua de três estágios.

pipeline {
    agent any 
    stages {
        stage('Build') {
            steps {
                // Compila a aplicação usando as regras do seu Makefile
                sh 'make' 
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
               }
           }
        stage('Test') {
            steps {
                // Executa a suíte de testes do projeto
                sh 'make test' 
				junit 'reports/**/*.xml'
               }
           }
		stage('Deploy') {
		    steps {
		        sh 'make publish' //
		       }
		   }
	   }
   }

|Tipo|O que faz?|Exemplo no Jenkins|
|---|---|---|
|**Debug Flags**|Aumentam a verbosidade de logs e expõem dados técnicos para **encontrar erros de lógica e infraestrutura**.|`jenkins.util.Timer.showStructure=true`|
|**Feature Flags**|Funcionam como um "interruptor" para **ligar ou desligar uma funcionalidade** sem a necessidade de novos deploys.|
```groovy
environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
```
Armazenar chaves estáticas da AWS nas credenciais do Jenkins não é muito seguro. Se você puder executar o próprio Jenkins na AWS (pelo menos o agente), é preferível usar funções do IAM para uma [conta de serviço](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) [de computador](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) ou [EKS](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). Também é possível usar a [federação de identidade web](https://github.com/jenkinsci/oidc-provider-plugin#accessing-aws).


### Fundamentos do Pipeline Roteirizado[](https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com#scripted-pipeline-fundamentals)

Na sintaxe do Scripted Pipeline, um ou mais `node`blocos fazem o trabalho principal ao longo de todo o Pipeline. Embora este não seja um requisito obrigatório de Sintaxe do Pipeline com script, confinando o trabalho do seu Pipeline dentro de um `node`bloco faz duas coisas:

1. Agenda as etapas contidas no bloco para executar adicionando um item para a fila de Jenkins. Assim que um executor é livre em um nó, o passos serão executados.
    
2. Cria um espaço de trabalho (um diretório específico para esse particular Pipeline) onde o trabalho pode ser feito em arquivos verificados a partir do controle de origem.  
    **Cuidado:** Dependendo da sua configuração do Jenkins, alguns espaços de trabalho podem não ser automaticamente limpo após um período de inatividade. Consulte os bilhetes e discussão vinculada de [JENKINS-2111 para](https://issue-redirect.jenkins.io/issue/2111) mais informações.
    

Jenkinsfile (Roteirizado Pipeline)

```groovy
node {  // 1
    stage('Build') { // 2
        // 3
    }
    stage('Test') { //4
        // 5
    }
    stage('Deploy') { // 6
        // 7
    }
}
```

|     |                                                                                                                                                                                                                                                                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Execute este Pipeline ou qualquer um de seus estágios, em qualquer agente disponível.                                                                                                                                                                                      |
| 2   | Define o estágio "Construir". `stage`blocos são opcionais no Scripted Pipeline sintaxe. No entanto, a implementação `stage`blocos em um Scripted Pipeline fornece visualização mais clara de cada `stage`Subconjunto de tarefas/passos na interface do usuário do Jenkins. |
| 3   | Realize algumas etapas relacionadas ao estágio "Construir".                                                                                                                                                                                                                |
| 4   | Define o estágio "Teste".                                                                                                                                                                                                                                                  |
| 5   | Realize algumas etapas relacionadas ao estágio "Teste".                                                                                                                                                                                                                    |
| 6   | Define o estágio "Deploy".                                                                                                                                                                                                                                                 |
| 7   | Execute algumas etapas relacionadas ao estágio "Implantar".                                                                                                                                                                                                                |