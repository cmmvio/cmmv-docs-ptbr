# Servidor

O módulo **@cmmv/graphql** inclui um **Apollo Server** totalmente configurado, que opera em **modo standalone** por padrão.

- **Geração Automática de Resolvers** → Consultas e mutações são criadas a partir dos contratos do sistema.
- **Autenticação Integrada** → Funciona perfeitamente com `@cmmv/auth` (se habilitado).
- **Geração Automática de Esquema** → Um arquivo de esquema `.graphql` é armazenado em `.generated/schema.graphql`.
- **Apollo Server Standalone** → Executa na **porta 4000** por padrão.
- **GraphQL Playground** → Disponível no **modo de desenvolvimento** para testar consultas.

## Como o Servidor Funciona

Quando o `@cmmv/graphql` é incluído em um projeto, os seguintes passos ocorrem:

1️⃣ **Resolvers são gerados automaticamente** → Os contratos são transformados em consultas/mutações GraphQL.
2️⃣ **Integração de autenticação** → Se o `@cmmv/auth` estiver habilitado, a autenticação é tratada automaticamente.
3️⃣ **Esquema é gerado** → O sistema salva um arquivo de esquema `.graphql` em `.generated/`.
4️⃣ **Apollo Server é iniciado** → Uma **API GraphQL** é lançada na **porta 4000** (padrão).
5️⃣ **GraphQL Playground fica disponível** → No **modo de desenvolvimento**, uma interface integrada permite testar consultas.

Isso garante uma **API GraphQL totalmente funcional** **sem exigir configuração manual**.

## Configuração do Servidor

Por padrão, o **Apollo Server** opera em **modo standalone** na **porta 4000**, mas isso pode ser personalizado.

### ✅ Exemplo: Alterando a Porta Padrão

Modifique seu arquivo **`.cmmv.config.cjs`**:

```javascript
module.exports = {
    server: {
        host: process.env.HOST || "0.0.0.0",
        port: process.env.PORT || 3000,
    },

    graphql: {
        port: process.env.GRAPHQL_PORT || 4000,
    },
};
```

| Opção | Padrão | Descrição |
|-------|--------|-----------|
| **`graphql.port`** | **4000** | Define a porta onde o servidor GraphQL é executado. |

## Acessando a API GraphQL

Uma vez que o servidor está em execução, a API GraphQL fica acessível em:

```
http://localhost:4000/graphql
```

Durante o desenvolvimento, o Apollo fornece um **GraphQL Playground interativo**, onde você pode:

✅ **Executar Consultas e Mutações**
✅ **Testar Cabeçalhos de Autenticação**
✅ **Visualizar o Esquema da API**

## Futuro em Produção

Em **atualizações futuras**, o Apollo Server será **incorporado como middleware** dentro do servidor HTTP padrão.
Isso significa que, no **modo de produção**, a API GraphQL estará acessível em:

```
http://localhost:3000/graphql
```

Isso garante que o **GraphQL se integre perfeitamente** à principal **API REST**.

- **Apollo Server é integrado** e opera na **porta 4000** por padrão.
- **Resolvers e Esquema são gerados automaticamente** a partir dos contratos do sistema.
- **Autenticação é totalmente integrada** se o `@cmmv/auth` estiver habilitado.
- **No modo de desenvolvimento**, o GraphQL Playground está acessível para testes.
- **Atualizações futuras em produção** incorporarão o Apollo dentro do servidor HTTP em `/graphql`.
