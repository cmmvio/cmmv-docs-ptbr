# Dicionário

A estrutura base dos contratos no CMMV foi inspirada no Protocol Buffers (Protobuf), o que pode fazer com que algumas nomenclaturas pareçam diferentes das tradicionalmente usadas em aplicações baseadas em SOLID. Este "Dicionário" tem como objetivo esclarecer os termos utilizados no CMMV, explicando o que eles representam na aplicação como um todo e como se conectam aos diferentes módulos e transpiladores do framework.

## Contratos vs. Schemas

O termo **"contratos"** é frequentemente associado a "schemas" em outros contextos, mas no CMMV há uma distinção importante. Enquanto um schema geralmente abrange todas as informações de uma aplicação, os contratos no CMMV focam em uma segmentação específica relacionada a uma estrutura de dados, como usuários, grupos, comentários, etc. Assim, um conjunto de contratos pode formar um schema completo neste cenário. Diferentemente do Protobuf puro, que foca apenas na estrutura de dados para comunicação, os contratos no CMMV vão além: eles contêm informações complementares que outros módulos utilizam para transpilar arquivos, como controllers, services, entidades e mais. Isso torna o contrato uma peça central que conecta e orienta a geração de código em toda a aplicação.

## Estrutura e Fluxo dos Contratos

### Models
O primeiro arquivo gerado a partir de um contrato é o **model**. Em SOLID, isso poderia ser comparado a uma "entidade", mas no CMMV o model tem um papel mais amplo. Além de definir uma interface simples com a estrutura de dados (como `id`, `nome`, etc.), o model também inclui uma classe que recebe decoradores do `class-validator` e do `class-transformer`. Esses decoradores são responsáveis por validar e transformar os dados de entrada (*input*) e saída (*output*), funcionando como uma ponte na serialização e desserialização. O fluxo típico é:
- Dados são recebidos via RESTful, RPC ou GraphQL.
- Passam pelo model para validação e transformação, se necessário.
- São empacotados e enviados ao repositório ou retornados como resposta.

Para melhorar a performance no retorno de APIs baseadas em JSON (como RESTful), o CMMV implementa suporte ao `json-fast-stringify`. O model é o ponto de partida para a derivação de todos os outros arquivos gerados no sistema.

### Módulo @cmmv/http
O módulo `@cmmv/http` é responsável pela implementação RESTful. Quando o contrato possui geração automática de controllers ativada, este módulo cria:
- **Services:** Inicialmente, implementa um CRUD simples em um array como *placeholder*. Funciona, mas sem persistência de dados.
- **Controllers:** Responsáveis por receber as rotas do servidor HTTP, repassar os dados ao service no formato correto e resolver a resposta no formato adequado para o protocolo HTTP.

### Módulo @cmmv/protobuf
O módulo `@cmmv/protobuf` utiliza os contratos para criar os arquivos `.proto`, compatíveis com RPC ou outros sistemas que suportam Protobuf. Ele suporta tipos, serviços e mensagens definidos no contrato, mas não utiliza *boilerplates* gerados diretamente pelo `protoc`. Esses arquivos `.proto` podem ser usados para comunicação eficiente em sistemas distribuídos.

### Módulo @cmmv/ws
O módulo `@cmmv/ws` implementa comunicação binária via WebSocket usando Protobuf. Ele abstrai os dados enviados e recebidos em formato binário, convertendo-os para o formato dos models. Além disso, cria **Gateways**, que funcionam como pontos de comunicação baseados em eventos, semelhante à abordagem de eventos do NestJS.

### Módulo @cmmv/repository
O módulo `@cmmv/repository` atualiza os services gerados pelo `@cmmv/http`, substituindo a implementação inicial em memória por uma integração completa com o banco de dados. Ele cria **entidades** no padrão do TypeORM, que possuem integração direta com os models, permitindo conversão fácil entre os formatos. O contrato pode incluir informações adicionais para personalizar essas entidades, como:
- Nome da tabela ou coleção.
- Registro automático de datas de criação e atualização.
- Registro do ID do usuário que criou ou editou o registro.
- Suporte a *soft delete* (exclusão lógica), que apenas marca o registro como excluído em vez de removê-lo.
- Definição de índices simples e compostos no cabeçalho do contrato.

### Módulo @cmmv/openapi
O módulo `@cmmv/openapi` gera documentação no padrão OpenAPI (usado pelo Swagger) com base nos contratos. Ele também cria a rota `/docs` com a interface do Swagger configurada para interagir com as rotas do sistema, incluindo as estruturas de dados e DTOs definidas nos contratos.

### Módulo @cmmv/graphql
O módulo `@cmmv/graphql` utiliza os contratos para criar os **resolvers** necessários ao funcionamento do GraphQL. Ele implementa o Apollo Server como servidor de comunicação e gera o arquivo `schema.graphql`, contendo os tipos, *queries*, *mutations* e resolvers da aplicação.

## Tabela de Termos

Abaixo está uma tabela que resume os termos utilizados no CMMV, comparando-os com conceitos equivalentes ou semelhantes em outras abordagens, como SOLID:

| **Termo no CMMV**       | **Equivalente ou Significado**                                                                                   |
|--------------------------|-----------------------------------------------------------------------------------------------------------------|
| **Contratos**           | Schemas (embora com uma diferença conceitual: contratos são segmentados, enquanto schemas abrangem tudo).         |
| **Models**              | Interface, Model ou Entity em SOLID – define a estrutura de dados e gerencia validação/transformação.           |
| **Services**            | Camada de lógica da aplicação – contém a implementação dos métodos, como CRUD ou funções personalizadas.         |
| **Controllers**         | Rotas REST – gerenciam as requisições HTTP e interagem com os services.                                         |
| **Gateways**            | Eventos RPC/WebSocket – pontos de comunicação baseados em eventos, semelhantes ao NestJS.                       |
| **Resolvers**           | Resolvedores do GraphQL – lidam com as *queries* e *mutations* no Apollo Server.                               |
| **Entities**            | Entidades do banco de dados – representam os dados persistidos, integradas ao TypeORM.                         |
| **Protos**              | Arquivos de Protobuf – definições `.proto` geradas para comunicação via RPC ou outros sistemas compatíveis.      |
| **Transpilers**         | Funções que, com base nos contratos, geram arquivos como models, services, controllers, entities, etc.          |

## Considerações Finais

Os contratos no CMMV são mais do que simples estruturas de dados: eles são o núcleo da geração de código e da integração entre os módulos do framework. Inspirados no Protobuf, mas expandidos para atender às necessidades de uma aplicação moderna, eles fornecem uma abordagem unificada para definir dados, lógica e comunicação, resultando em um sistema coeso, escalável e altamente automatizado.
