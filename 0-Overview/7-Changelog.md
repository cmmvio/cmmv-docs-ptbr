# Changelog

## Versão 0.9.0
20 de março de 2025

### Novos Recursos

- **Projeto `@cmmv/sandbox`**
  - Adicionada uma **interface interativa** para criação de **contratos complexos** no padrão CMMV.
  - Permite **testes integrados** das rotas **RESTful** e **GraphQL** diretamente na interface.
  - Fornece suporte para **visualização e manipulação** de contratos antes da implementação no sistema.

- **Carregamento Dinâmico de Contratos Públicos no `@cmmv/core`**
  - Agora o **núcleo do CMMV** pode carregar automaticamente **contratos públicos**, facilitando a modularização e o uso dinâmico de contratos sem necessidade de importação manual.

- **Correções e Melhorias nas Interfaces de Contratos**
  - As interfaces dos contratos foram **refatoradas** para melhor compatibilidade e estruturação.
  - **Inclusão de `paramType` nas mensagens**, garantindo **tipagem correta** nas chamadas RPC e GraphQL.

- **Melhorias no Parser de Contratos**
  - Várias **correções no parsing** de contratos foram aplicadas, garantindo **compatibilidade e padronização** dos contratos gerados.

- **Nova Classe de Compilação para Contratos**
  - Criada uma classe que **recebe um contrato em formato JSON** e **gera automaticamente o contrato em TypeScript** no formato correto.
  - Melhoria na **automatização da criação de contratos**, permitindo integração com sistemas externos.

### Melhorias e Correções no `@cmmv/auth`

- **Correções e Inclusão de Novos Dados em Contratos de Autenticação**
  - Ajustes nos contratos do **módulo de autenticação**, garantindo **compatibilidade total** com os novos modelos de contratos.
  - Melhorias na **gestão de usuários, permissões e tokens**, permitindo maior controle na definição de regras de acesso.

### Recomendações

- **Aproveite o `@cmmv/sandbox` para Criar e Testar Contratos**
  - Utilize a **nova interface interativa** para facilitar a criação de contratos **complexos** e testar suas APIs antes da implementação.

- **Utilize o Novo Carregamento Dinâmico de Contratos**
  - Agora o CMMV **importa automaticamente contratos públicos**, permitindo **menor esforço manual** ao definir novos contratos no sistema.

- **Adapte os Contratos Existentes para `paramType`**
  - Atualize **suas mensagens e contratos** para incluir **`paramType`**, garantindo melhor validação de dados e compatibilidade com **REST, GraphQL e RPC**.

A atualização para **0.9.0** traz **grandes melhorias no desenvolvimento e gerenciamento de contratos**, além de **facilitar testes e integração** com APIs, tornando o CMMV **mais eficiente e modular**.

## Versão 0.8.34
13 de março de 2025

### Novos Recursos

- **Módulo GraphQL (`@cmmv/graphql`)**
  - Introdução do suporte nativo para **GraphQL** com `type-graphql` e `Apollo Server`.
  - Geração automática de **tipos e resolvers** a partir dos contratos do sistema.
  - Integração nativa com `@cmmv/auth`, oferecendo **autenticação**, **controle de acesso baseado em funções** e **refresh de tokens**.
  - Suporte a **paginação e filtragem**, garantindo compatibilidade com as funcionalidades REST e RPC.

- **Módulo OpenAPI (`@cmmv/openapi`)**
  - Geração automática de **documentação OpenAPI/Swagger** para todos os controladores e rotas.
  - Criação dos arquivos `openapi.json` e `openapi.yml` no diretório `/public` para integração externa.
  - Implementação da rota `/docs` para acesso à documentação pública da API.
  - Suporte a **autenticação via Bearer Token e OAuth2** quando `@cmmv/auth` estiver ativado.

- **Transpilador de Schemas de Contrato**
  - Novo **transpilador de geração de schemas** nos formatos **JSON e YAML**.
  - Todos os contratos agora geram **schemas estruturados**, permitindo melhor interoperabilidade com outros serviços.

### Melhorias no Core

- **Extensão da API de `Application`**
  - Adição de `getResolver` e `getResolvers` para recuperar resolvers GraphQL gerados dinamicamente.
  - Introdução do método `resolveProvider`, permitindo a resolução dinâmica de **provedores com dependências** em outros provedores.

- **Aprimoramento do suporte a OpenAPI e GraphQL no `@cmmv/core`**
  - Os contratos agora **suportam metadados** para a geração de schemas OpenAPI.
  - Mecanismo embutido para **registro automático de resolvers** a partir dos contratos quando GraphQL está ativado.

### Correções e Melhorias

- **Atualizações na Autenticação (`@cmmv/auth`)**
  - **Melhoria na segurança** com validação e refresh de tokens aprimorados.
  - Correção de inconsistências no gerenciamento de sessões ao utilizar **controle de acesso baseado em funções**.
  - **Suporte a autenticação GraphQL**, garantindo integração perfeita entre todos os níveis da API.

- **Otimizações de Desempenho**
  - **Melhoria na geração de schemas de contratos**.
  - Otimização na **compilação dos schemas OpenAPI**, reduzindo **tempo de build**.

### Recomendações

- **Adote OpenAPI para Documentação**
  - Se você precisa de documentação para sua API, **ative `@cmmv/openapi`** para gerar automaticamente os arquivos OpenAPI.

- **Migre para GraphQL para Consultas Flexíveis**
  - Aplicações que precisam de **consultas personalizadas** devem considerar a migração para `@cmmv/graphql` para aproveitar resolvers **tipados**.

- **Atualize os Fluxos de Autenticação**
  - Utilize os novos recursos de autenticação do `@cmmv/auth`, incluindo **refresh de JWT, OAuth2 e controle de acesso baseado em funções**.

A atualização para **0.8.34** traz **grandes melhorias** em **GraphQL, OpenAPI, geração de schemas e autenticação**, tornando o CMMV **mais extensível e interoperável**.

---

## Versão 0.8.30
5 de março de 2025

### Correções Críticas

- **Correção na Inclusão de Arquivos do Transpilador em `@cmmv/core`**
  - Resolução de um problema onde arquivos incluídos nos transpiladores **não eram corretamente processados**, garantindo consistência na saída gerada.

- **Correção de Tipos de Parâmetros Vinculados no `@cmmv/http`**
  - Ajuste em **tipos de parâmetros incorretos** que estavam afetando a confiabilidade do manuseio de parâmetros em rotas HTTP.

- **Correção de Schemas do `fast-json-stringify` no `@cmmv/http`**
  - Correção na geração de schemas usando `fast-json-stringify`, garantindo **serialização correta e melhor desempenho**.

### Adições

- **Novos Hooks na Aplicação (`@cmmv/core`)**
  - Introdução de hooks de ciclo de vida (`awaitModule`, `awaitService`) para melhor controle da inicialização de serviços.

- **Relacionamento entre Contratos (`@cmmv/core`)**
  - Suporte para **definição de relacionamentos** entre contratos, melhorando a modularidade e reutilização.

- **Indexação Personalizada nos Contratos**
  - Agora é possível definir **índices personalizados** diretamente nos contratos para otimizar consultas no banco de dados.

- **Suporte a Alias em Modelos**
  - Modelos agora podem ter **aliases**, permitindo identificadores alternativos em consultas ao banco de dados.

- **Carregamento Recursivo de Provedores**
  - Agora os **provedores são carregados recursivamente**, garantindo a resolução correta de dependências durante a inicialização.

- **`execAsyncFn` para Processamento Assíncrono**
  - Novo método `execAsyncFn` para execução **não bloqueante** de funções específicas de provedores.

- **Novos Decoradores em `@cmmv/http`**
  - **`@Redirect` e `@HTTPCode`** para definir redirecionamentos e códigos de status HTTP diretamente nos controladores.

- **Novas Funções para Modelos em `@cmmv/http`**
  - Métodos `fromPartial`, `fromEntity` para criação facilitada de modelos.
  - Suporte para `getByIdRaw` para busca direta de entidades no banco de dados.

- **Melhorias na Autenticação (`@cmmv/auth`)**
  - **Gerenciamento de funções e grupos** para permissões avançadas de usuários.
  - **Verificação de impressão digital de JWT** para evitar sequestro de tokens.
  - **Autenticação em Dois Fatores (2FA)** com suporte a geração de QR Code.
  - **Registro de sessões** com impressão digital de dispositivos e dados de navegação.
  - **Validação de ReCAPTCHA** para melhorar a segurança contra bots.
  - **Criptografia AES-256-GCM para tokens JWT**, garantindo armazenamento seguro.
  - **Suporte a Refresh Token**, permitindo sessões contínuas para usuários autenticados.

- **Funções de Paginação e Contagem em `@cmmv/repository`**
  - Introdução de **`pagination`, `count` e `insertIfNotExists`** para melhor gerenciamento de dados.
  - Novos métodos **`updateOne`, `exists` e `count`** para consultas mais eficientes.

- **Novo Destino de Arquivos Protobuf em `@cmmv/protobuf`**
  - **Mudança na organização** dos arquivos gerados `.proto`, melhorando a estrutura do projeto.

- **Organização de Gateways WebSocket e Modelos em `@cmmv/ws`**
  - **WebSocket Gateways e Modelos movidos para `.generated`**, mantendo **placeholders no diretório `src`** para melhor organização do código.

- **Novo Módulo Vault (`@cmmv/vault`)**
  - **Geração segura de chaves e criptografia de payloads**.
  - Inserção, recuperação e exclusão de dados criptografados via **ECC e AES-256-GCM**.

### Melhorias

- **Suporte a Múltiplos Formatos de Configuração**
  - `@cmmv/core` agora suporta arquivos `.js`, `.cjs` e `.ts` para configuração.

- **Otimização de Modelos Baseados em Contratos**
  - Agora contratos, modelos e entidades **são carregados dinamicamente**, reduzindo importações manuais.

### Atualizações

- **Documentação Atualizada**
  - Guias detalhados para **autenticação avançada, indexação de contratos, segurança e Vault**.

A versão **0.8.30** melhora **autenticação, indexação, segurança e gerenciamento de contratos**, tornando o CMMV **mais robusto e escalável** para aplicações empresariais.
