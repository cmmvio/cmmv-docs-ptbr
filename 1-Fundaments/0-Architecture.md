# Arquitetura

À primeira vista, a estrutura do projeto CMMV, com seus módulos interconectados e componentes orquestrados, pode parecer complexa. No entanto, detalharemos cada elemento abaixo para esclarecer sua função dentro da aplicação.

<img src="/assets/cmmv-core.png" />

# Core

O **Core** atua como o orquestrador de todo o sistema. Ele contém classes fundamentais necessárias para a operação do sistema, como **Singleton**, **Application**, e outras utilidades essenciais. Além disso, fornece tudo o que é necessário para a criação de contratos. Através do Core, é possível configurar a aplicação e integrar módulos complementares, transpiladores, providers e serviços, entre outros componentes.

Quando a aplicação é iniciada, o primeiro processo é carregar todos os módulos e submódulos configurados no sistema. Isso inclui seus serviços, configurações adicionais, validações e, principalmente, **transpiladores**. Os transpiladores desempenham um papel crucial no sistema, gerando arquivos com base nos contratos, views e formulários.

A aplicação requer apenas o módulo **Core** para funcionar. Para microserviços, módulos adicionais podem não ser necessários. No entanto, sem outros módulos, a aplicação só seria capaz de executar tarefas aplicadas diretamente no código. Por isso, o **sistema de módulos** é tão importante.

Além disso, a aplicação é responsável por criar **models, interfaces e schemas para fast-json-stringify**, bem como **DTOs**. Essas estruturas de dados são utilizadas para:
- **Tipagem automática** dentro da aplicação
- **Validação de entrada** usando class-validator
- **Transformação e serialização de dados**
- **Parsing JSON de alta performance** via fast-json-stringify

# HTTP

O módulo **HTTP** é responsável por:

* Criar serviços e controladores para operações CRUD básicas
* Manipular validação de dados, exportação e aplicação de filtros
* Expor uma **API RESTful**

Por padrão, o módulo HTTP usa `@cmmv/server` como camada de abstração para lidar com solicitações e servir arquivos, mas pode ser substituído por Express ou outros frameworks.

O módulo HTTP inclui suporte embutido para:

* Cache **ETag**
* **Compressão**
* **Cabeçalhos de segurança** (Helmet)
* **Validação de entrada JSON**
* **Manipulação de cookies**
* **Serviço de arquivos estáticos**
* **CORS (Cross-Origin Resource Sharing)**
* **Gerenciamento de sessões**

# Repository (Opcional)

O **módulo Repository** é fundamental para qualquer aplicação, pois fornece uma camada de abstração para o **TypeORM**. Ele gerencia a comunicação entre a aplicação e o banco de dados, com suporte a:

* SQLite
* MongoDB
* SQL Server
* MySQL
* PostgreSQL
* Oracle

Este módulo é responsável por **gerar entidades ORM**, **criar índices e relacionamentos** (para bancos relacionais). Além disso, os serviços anteriormente gerados pelo módulo HTTP serão atualizados para recuperar dados diretamente do banco de dados, eliminando a necessidade de integrações adicionais.

# Cache (Opcional)

O módulo **Cache** complementa HTTP e RPC fornecendo uma camada de cache em memória do servidor. Pode ser facilmente aplicado a controladores e gateways por meio de **decoradores**.

O módulo Cache suporta:

* Redis
* Memcached
* MongoDB
* Arquivos binários

Ele utiliza o **cache-manager** para seu funcionamento. As configurações de cache podem ser definidas diretamente nos contratos, com opções para **TTL (tempo de vida)**, **compressão** e **prefixação de chaves**.

## Protobuf (Opcional)

O CMMV oferece suporte nativo a **RPC**, e para isso, o **módulo Protobuf** é responsável por gerar contratos no padrão `.proto`. Ele também cria **interfaces e types** para uso no **TypeScript**, além de fornecer uma **estrutura JSON** para suportar Protobuf no frontend quando a comunicação ocorre via RPC.

Além disso, o **módulo `@cmmv/vue`** integra o **Protobuf com Vue 2 e Vue 3**, fornecendo **mixins e composables** para fácil implementação. Esse módulo também configura a **comunicação WebSocket binária**, garantindo uma integração perfeita entre backend e frontend.

## WebSocket (Opcional)

Para uma comunicação RPC eficiente e **com baixo overhead**, o **módulo WebSocket** gerencia a **transmissão bidirecional de dados binários** baseada em contratos. Como mencionado anteriormente, todas as implementações do frontend são estruturadas em torno de **contratos**, que podem ser usados de duas maneiras:

1. **Importando o bundle JavaScript gerado**, simplificando as interações baseadas em contrato.
2. **Usando o módulo `@cmmv/vue`**, que fornece uma abordagem simplificada para integrar a comunicação WebSocket no Vue.

Essa estrutura modular garante que as chamadas **RPC** sejam otimizadas para **desempenho, escalabilidade e facilidade de uso**, permitindo uma comunicação em tempo real leve e eficiente.

## Auth (Opcional)

O **módulo Auth** é responsável pelo gerenciamento de acesso ao sistema, fornecendo **autenticação local segura** com **usuário e senha criptografados**. Ele também suporta **autenticação via provedores externos**, como **Google, Facebook e outros**.

- **Autenticação e registro de usuários**
- **Controle de acesso baseado em funções (RBAC)**
- **Grupos de permissões para controle detalhado**
- **Integração automática com controladores e gateways** para proteção de rotas privadas
- **Geração e gerenciamento de tokens de acesso**
- **Suporte a OAuth para autenticação de terceiros**
- **Compatível com comunicação RESTful e RPC**, sem necessidade de implementações adicionais

Este módulo garante um **sistema de autenticação seguro e escalável**, simplificando o gerenciamento de usuários enquanto fornece medidas robustas de segurança.

## Módulos Oficiais do CMMV

Além de sua funcionalidade principal, o CMMV fornece **módulos oficiais** que expandem suas capacidades para diversos casos de uso. Esses módulos abrangem **criação de formulários, componentes de frontend, agendamento de tarefas, testes automatizados, filas de mensagens, busca full-text, criptografia, documentação de API e comunicação baseada em eventos**.

### Módulos Disponíveis:

<br/>

- **`@cmmv/form`** - Gera **schemas para dashboards** e elementos de formulário dinâmicos.
- **`@cmmv/ui`** - Fornece **componentes Vue 3** para **painéis administrativos** e páginas.
- **`@cmmv/scheduling`** - Suporte para **agendamento de tarefas pré-definidas**.
- **`@cmmv/testing`** - Habilita **testes automatizados** para aplicações CMMV.
- **`@cmmv/view`** - Fornece **renderização básica no servidor (SSR)**.
- **`@cmmv/queue`** - Implementa **filas de processamento**, com suporte para **Redis, RabbitMQ e Kafka**.
- **`@cmmv/elastic`** - Integração com **Elasticsearch** para **busca full-text**.
- **`@cmmv/email`** - Gerenciamento de **envio de e-mails** com configuração embutida.
- **`@cmmv/normalizer`** - Suporte para **normalização de dados** nos formatos **JSON, XML e CSV**.
- **`@cmmv/encryptor`** - Fornece **criptografia avançada** e **assinaturas digitais com curvas elípticas**.
- **`@cmmv/swagger`** - Gera **documentação de API** automaticamente.
- **`@cmmv/events`** - Implementa **comunicação interna baseada em eventos** para microsserviços e módulos.
- **`@cmmv/vue`** - Integração simplificada com **Vue 2 e Vue 3**.

Esses módulos oferecem **recursos prontos para uso**, permitindo que os desenvolvedores expandam suas aplicações sem complexidade adicional. Seja para **desenvolvimento de UI, processamento em segundo plano, segurança ou comunicação interna**, o **CMMV fornece soluções modulares e escaláveis** para aplicações modernas.
