# Arquitetura

À primeira vista, a estrutura do projeto CMMV, com seus módulos interconectados e componentes orquestrados, pode parecer complexa. No entanto, vamos detalhar cada elemento abaixo para esclarecer sua função dentro da aplicação.

<img src="/assets/cmmv-core.png" />

## Core

O **Core** atua como **orquestrador** de todo o sistema. Ele contém classes fundamentais para o funcionamento da aplicação, como **Singleton, Application** e outras utilidades essenciais. Além disso, fornece todos os recursos necessários para a **criação de contratos**. Através do Core, é possível **configurar a aplicação** e integrar **módulos complementares, transpilers, providers e serviços**, entre outros componentes.

Ao iniciar a aplicação, o primeiro processo é **carregar todos os módulos e submódulos configurados no sistema**. Isso inclui **serviços, configurações adicionais, validações** e, principalmente, os **transpilers**. Os **transpilers** têm um papel crucial no sistema, pois geram **arquivos do sistema** baseados em **contratos, views e formulários**.

A aplicação **precisa apenas do Core** para funcionar. Para **microsserviços**, módulos adicionais podem não ser necessários. No entanto, sem outros módulos, a aplicação só será capaz de executar **tarefas aplicadas diretamente via código**. Por isso, o **sistema modular é essencial**.

Além disso, a aplicação é responsável por **gerar modelos, interfaces e schemas para fast-json-stringify, bem como DTOs**. Essas estruturas são utilizadas para **tipagem automática, validação de entrada de dados com class-validator, transformação e serialização de dados** e **parseamento de JSON de alta performance** usando `fast-json-stringify`.

## HTTP

O módulo **HTTP** é responsável por:

* Criar **serviços e controladores** para operações CRUD básicas.
* Lidar com **validação de dados, exportação e aplicação de filtros**.
* Expor uma **API RESTful**.

Por padrão, o módulo HTTP usa `@cmmv/server` como camada de abstração para gerenciar requisições e servir arquivos. No entanto, ele pode ser substituído por **Express** ou outros frameworks.

O módulo HTTP inclui suporte nativo para:

* **ETag caching**
* **Compressão de resposta**
* **Segurança com headers HTTP (Helmet)**
* **Validação de entrada de dados em JSON**
* **Manipulação de cookies**
* **Serviço de arquivos estáticos**
* **Suporte a CORS (Cross-Origin Resource Sharing)**
* **Gerenciamento de sessão**

## Repository (Opcional)

O módulo **Repository** é **fundamental** para qualquer aplicação, pois fornece uma camada de abstração para **TypeORM**. Ele gerencia a comunicação entre a aplicação e o banco de dados, com suporte para:

* **SQLite**
* **MongoDB**
* **SQL Server**
* **MySQL**
* **PostgreSQL**
* **Oracle**

Esse módulo **gera automaticamente entidades ORM, índices e relacionamentos** em bancos de dados relacionais. Além disso, os **serviços previamente gerados pelo módulo HTTP** serão atualizados para buscar dados diretamente no banco de dados, eliminando a necessidade de integrações adicionais.

## Cache (Opcional)

O módulo **Cache** complementa o **HTTP e RPC**, fornecendo uma **camada de cache em memória** para o servidor. Ele pode ser facilmente aplicado em **controladores e gateways** através de **decoradores**.

O módulo Cache suporta:

* **Redis**
* **Memcached**
* **MongoDB**
* **Armazenamento em arquivos binários**

O funcionamento do cache é baseado no **cache-manager**. As configurações de cache podem ser **definidas diretamente nos contratos**, incluindo **TTL (time-to-live), compressão e prefixação de chaves**.

## Protobuf (Opcional)

O CMMV **suporta RPC nativamente**, e para isso, o **módulo Protobuf** é responsável por **gerar contratos no formato `.proto`**. Ele também cria **interfaces e tipos para TypeScript** e uma **estrutura JSON** para suportar **Protobuf no frontend** quando a comunicação ocorre via RPC.

Além disso, o módulo **`@cmmv/vue`** integra o **Protobuf com Vue 2 e Vue 3**, fornecendo **mixins e composables** para implementação simplificada. Esse módulo também configura **WebSocket binário para comunicação**, garantindo **integração perfeita entre backend e frontend**.

## WebSocket (Opcional)

Para comunicação **RPC eficiente com baixa latência**, o **módulo WebSocket** gerencia a **transmissão bidirecional de dados binários** baseada em contratos. O CMMV suporta duas formas principais de utilizar contratos no frontend:

- **Importando o bundle JavaScript gerado**, facilitando a comunicação baseada em contratos.
- **Usando o módulo `@cmmv/vue`**, que simplifica a integração do WebSocket com aplicações Vue.

Essa estrutura modular garante que **as chamadas RPC sejam otimizadas** para **desempenho, escalabilidade e facilidade de uso**, permitindo **comunicação leve e em tempo real**.

## Auth (Opcional)

O **módulo Auth** gerencia o acesso ao sistema, fornecendo **autenticação segura** com **criptografia de credenciais**. Ele também suporta **autenticação de terceiros** via **Google, Facebook e outros**.

- **Autenticação e registro de usuários**
- **Controle de acesso baseado em funções (RBAC)**
- **Grupos de permissões para autorização granular**
- **Proteção automática de rotas privadas em controladores e gateways**
- **Geração e gerenciamento de tokens de acesso**
- **Suporte a OAuth para autenticação externa**
- **Compatibilidade com comunicação RESTful e RPC sem necessidade de implementação adicional**

Esse módulo garante um **sistema de autenticação seguro e escalável**, simplificando o gerenciamento de usuários com **medidas robustas de segurança**.

## Módulos Oficiais do CMMV

Além de suas funcionalidades principais, o CMMV fornece **módulos oficiais** que **expandem suas capacidades** para diferentes casos de uso. Esses módulos abrangem **criação de schemas de formulários, componentes UI para frontend, agendamento de tarefas, testes, renderização server-side (SSR), filas de mensagens, busca, criptografia, documentação de API e comunicação baseada em eventos**.

### Módulos Disponíveis:

<br/>

- **`@cmmv/form`** - Gera **schemas para dashboards** e elementos dinâmicos de formulários.
- **`@cmmv/ui`** - Fornece **componentes frontend baseados em Vue 3** para construção de **painéis administrativos** e páginas.
- **`@cmmv/sandbox`** - Interface interativa para **criação e testes de contratos** com suporte a **RESTful e GraphQL**.
- **`@cmmv/scheduling`** - Suporte a **agendamentos de tarefas predefinidos**.
- **`@cmmv/testing`** - Permite **testes automatizados** em aplicações CMMV.
- **`@cmmv/queue`** - Implementa **filas de tarefas**, com suporte a **Redis, RabbitMQ e Kafka**.
- **`@cmmv/elastic`** - Integração com **Elasticsearch** para **busca full-text**.
- **`@cmmv/email`** - Gerenciamento de **envio de e-mails** com configuração embutida.
- **`@cmmv/normalizer`** - Suporte à **normalização de dados** para **JSON, XML e CSV**.
- **`@cmmv/encryptor`** - Fornece **criptografia avançada** e **assinaturas digitais com curvas elípticas**.
- **`@cmmv/events`** - Comunicação **event-driven** para microservices e módulos.
- **`@cmmv/vue`** - Integra **Vue 2 e Vue 3** com aplicações CMMV.
- **`@cmmv/vault`** - Armazenamento seguro de dados usando **criptografia ECC e AES-256-GCM**.

Esses módulos permitem **extensões plug-and-play**, tornando o **CMMV modular e escalável** para aplicações modernas.
