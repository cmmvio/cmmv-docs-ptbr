# Introdução

CMMV (Contract-Model-Model-View) é um **ecossistema full-stack** que automatiza a criação de toda a estrutura de backend e frontend com base nos **princípios SOLID**. Ao aproveitar **contratos TypeScript**, o CMMV gera:

- **Backend**: Modelos, entidades de banco de dados, serviços, controladores, gateways RPC e serialização (otimizada com \`json-fast-stringify\`).
- **Frontend**: Esquemas de formulários, configurações de tabelas de dados e componentes UI baseados nos contratos.
- **Painel Administrativo**: Um painel totalmente funcional alimentado por \`@cmmv/admin\`, gerado automaticamente a partir dos contratos e views.

O CMMV **suporta APIs RESTful, RPC e GraphQL**, permitindo **integração perfeita** com aplicativos externos. Atualizações futuras introduzirão **transpilação de contratos para outras linguagens de programação**, garantindo compatibilidade entre plataformas.

Além disso, o CMMV fornece **integrações nativas** para:
- **Vue 3, React, Angular** (para SPAs e dashboards modernos)
- **Flutter & Electron** (para aplicativos móveis e desktop multiplataforma)

## Por que CMMV?

O CMMV foi projetado para **resolver os gargalos** do desenvolvimento moderno de backend e frontend. Enquanto frameworks tradicionais exigem **configuração manual de controladores, serviços e views**, o CMMV **automatiza esse processo**, eliminando redundâncias e melhorando a manutenção.

### 🔥 Benefícios Principais
<br/>

* 🚀 **Geração Automática de Código**: Defina contratos uma vez e gere todo o resto.
* 🔄 **Arquitetura Padronizada**: Impõe as melhores práticas.
* 🔌 **Integrações Perfeitas**: Funciona com bancos de dados, autenticação, cache e muito mais.
* 📡 **Suporte a Múltiplos Protocolos**: REST, RPC, GraphQL em um único sistema.
* ⚡ **Desempenho Otimizado**: Usa \`json-fast-stringify\` para serialização de alta velocidade.

## Como Funciona

O CMMV **gera** todos os componentes necessários de backend e frontend com base em **contratos TypeScript**.

### **Fluxo do Backend**
<br/>

1. **Definição de Contratos**: Criação de modelos com decorators.
2. **Geração Automática**:
   - **Modelos** (Entidades ORM)
   - **Serviços** (Lógica de Negócio)
   - **Controladores** (API REST)
   - **Gateways** (WebSocket RPC)
   - **Serialização** (Otimizada para desempenho)
3. **API Pronta**: Exposição de endpoints via **REST, RPC ou GraphQL**.

### **Fluxo do Frontend**
<br/>

1. **Esquemas de Formulário e Tabela**: Baseados nos contratos.
2. **Componentes UI**: Utiliza \`@cmmv/ui\` para dashboards.
3. **Painel Administrativo**: \`@cmmv/admin\` gera views automaticamente.

## Painel Administrativo Gerado Automaticamente

Assim que contratos e views são definidos, o \`@cmmv/admin\` gerará automaticamente:

* Gerenciamento de Usuários
* Controle de Acesso Baseado em Função (RBAC)
* Operações CRUD
* Suporte a GraphQL, REST e RPC

## Integração
<br/>

* Vue 3 (recomendado)
* React (futuro)
* Angular (futuro)
* Flutter (futuro)
* Electron (futuro)

### Integração com Banco de Dados
<br/>

* MongoDB, PostgreSQL, MySQL, SQL Server
* Indexação automática e mapeamento de relacionamentos
* APIs CRUD com filtros, ordenação e paginação

### Cache & Desempenho
<br/>

* Suporte embutido para Redis & Memcached
* Decorators de auto-cache para serviços e controladores

### Filas de Tarefas & Mensageria
<br/>

* Integração com Kafka, RabbitMQ, Redis Queue
* Trabalhos assíncronos e arquitetura baseada em eventos

## Roadmap Futuro

O CMMV continuará expandindo suas capacidades de automação full-stack:

* 🔄 Transpilação para Outras Linguagens (C#, Go, Python, Java)
* 📡 Expansão de Recursos GraphQL
* 📲 Suporte para Aplicações Mobile com Flutter
* 🖥️ Suporte para Aplicações Desktop com Electron
* 🌎 Melhorias em Internacionalização (i18n & l10n)

💡 **Você sabia?** Muitas linguagens modernas adotam o conceito de contratos para garantir **tipagem forte e segurança em tempo de execução**. Ferramentas como Protocol Buffers (Protobuf) e GraphQL Schema cumprem um papel similar ao CMMV, permitindo que sistemas diferentes se comuniquem sem ambiguidades.
