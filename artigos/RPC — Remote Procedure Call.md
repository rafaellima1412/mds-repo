
O RPC (Remote Procedure Call) é um protocolo utilizado para executar chamadas de procedimento ou métodos em um sistema computacional diferente daquele em que o código está sendo executado. Esse protocolo permite que um programa em um dispositivo cliente envie uma solicitação de execução de procedimento para um software em outro dispositivo servidor, que executa o procedimento e retorna o resultado.

O RPC abstrai a complexidade da comunicação em rede, permitindo aos desenvolvedores se concentrarem na lógica de negócios, em vez dos detalhes de como os dados são transmitidos e recebidos. Existem vários tipos de protocolos RPC, como SOAP, Thrift e CORBA, entre outros. Mais à frente, iremos abordar uma alternativa moderna desse tipo de protocolo: o gRPC.

## Exemplo de um servidor RPC

Ao contrário do gRPC, chamadas RPC convencionais não precisam necessariamente de um contrato forte. Isso pode ser bom pela flexibilidade e velocidade de implementação, mas ruim para manter um padrão e a consistência dos dados.

Nesse exemplo, a ideia é implementar uma chamada para um sistema que calcula a quantidade diária recomendada de ingestão de proteína com base no peso informado. A implementação consiste em criar um método e registrá-lo em um servidor RPC alocado em uma porta do host — neste caso, a porta 1234.

## Exemplo de um cliente RPC

A implementação de um cliente tende a ser bem mais simples. Basta criar uma conexão com a porta onde o servidor e o método RPC foram alocados, informando uma lista de argumentos no formato esperado pelo servidor.

Como o RPC convencional não exige um contrato forte, os prós e contras tendem a ser os mesmos: velocidade e flexibilidade em troca de consistência.

## gRPC — Google Remote Procedure Call

O gRPC é um framework de chamada de procedimento remoto (RPC) de código aberto, desenvolvido pelo Google. Ele permite que os desenvolvedores conectem serviços de maneira performática e escalável, possibilitando a comunicação entre serviços construídos de forma distribuída.

Seu design baseia-se no uso de HTTP/2 como protocolo de transporte e Protocol Buffers (ProtoBuf) como linguagem de descrição de interface de serviço (IDL). Além do conceito de chamadas RPC, oferece recursos como autenticação, balanceamento de carga e validações.

Essa arquitetura é ideal para construir arquiteturas de microsserviços, nas quais serviços leves e performáticos são importantes para o desempenho e a escalabilidade. Com o HTTP/2, é possível fazer múltiplas chamadas RPC em paralelo sobre uma única conexão TCP, melhorando a eficiência da rede e a latência.

O gRPC suporta streaming bidirecional, permitindo que tanto o cliente quanto o servidor enviem uma sequência de mensagens um para o outro usando uma única conexão. Isso é útil para casos como chat e monitoramento em tempo real, além de outros cenários que exigem comunicação contínua e persistente.

Implementar e gerenciar um sistema baseado em gRPC pode ser mais complexo do que usar alternativas mais simples, como REST, especialmente em projetos menores ou com requisitos menos rigorosos de desempenho.

A utilização de contratos via ProtoBuf para manter a consistência implica a necessidade de distribuir e versionar esse contrato entre o cliente e o servidor. Quando o contrato precisa ser modificado para adicionar, alterar ou remover uma variável, pode haver o desafio de garantir a atualização de todos os clientes desse serviço.

## Protocol Buffers (ProtoBuf)

O Protocol Buffers, ou ProtoBuf, é a linguagem de descrição de interface preferida pelo gRPC. É usada para definir os serviços e a estrutura de dados compartilhados entre cliente e servidor por meio de um contrato forte.

O ProtoBuf é um sistema de serialização binária que, além de ser mais eficiente em termos de espaço do que formatos como JSON, oferece uma maneira clara de especificar a interface do serviço de forma independente de linguagens e frameworks.

## Exemplo de Protobuf

Nesse contrato de exemplo, descrevemos a assinatura de comunicação gRPC entre cliente e servidor usando a sintaxe proto3. O serviço é chamado `IMCService` e implementa um método chamado `Calcular`, que recebe uma mensagem no formato `IMCRequest` e retorna uma mensagem no formato `IMCResponse`.

A mensagem de solicitação contém peso e altura; a mensagem de resposta contém o resultado do cálculo do IMC. A linguagem é descritiva e relativamente simples.

Após descrever o contrato, precisamos gerar os pacotes Go que o implementam. Normalmente, usamos o compilador `protoc`. Ao gerar os arquivos para Go, são produzidos arquivos como `imc_grpc.pb.go` e `imc.pb.go`, que precisam estar disponíveis tanto no cliente quanto no servidor.

## Exemplo de servidor gRPC

Com os contratos gerados pelo Protobuf importados, podemos iniciar a implementação de um servidor gRPC. É necessário respeitar os contratos definidos e implementar um método chamado `Calcular`, que recebe e responde aos objetos com a assinatura definida.

Depois de implementar as funções necessárias com a lógica do serviço, basta alocar uma porta — neste exemplo, a 50051 — e registrar esse serviço no servidor gRPC.

## Exemplo de cliente gRPC

Para implementar o cliente de um servidor gRPC, precisamos utilizar o mesmo contrato, criar uma conexão com o endereço e a porta do servidor e chamar o método definido na assinatura, `Calcular`, informando os dados no formato acordado e recebendo a resposta em seguida.
