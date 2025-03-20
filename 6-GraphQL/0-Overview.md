# Visão Geral

O módulo `@cmmv/graphql` oferece uma integração perfeita com GraphQL usando `type-graphql` e `Apollo Server`. Ele lê automaticamente os contratos do sistema e gera os tipos e resolvers necessários, garantindo uma configuração sem esforço para aplicações GraphQL. Além disso, inclui mecanismos de autenticação que complementam o `@cmmv/auth`.

Por padrão, o servidor GraphQL é executado separadamente da API REST na porta **4000**, mas isso pode ser configurado conforme necessário. Atualizações futuras visam integrar o Apollo Server como middleware dentro do `@cmmv/http`.

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso Importante</p>
    <p>
        A versão atual do <strong>@cmmv/graphql</strong> ainda está em beta e ainda não suporta a personalização de <strong>escalares, diretivas, middlewares ou plugins</strong>, nem permite substituir o servidor GraphQL subjacente. Esses recursos estão planejados para versões futuras.
    </p>
</div>

## Instalação

Para instalar o `@cmmv/graphql`, execute:

```bash
$ pnpm add @cmmv/graphql apollo-server type-graphql graphql
```

## Configuração

Para habilitar o suporte ao GraphQL em sua aplicação CMMV, adicione o ``GraphQLModule`` à sua configuração:

```typescript
import { Application } from "@cmmv/core";
import { GraphQLModule } from "@cmmv/graphql";
import { DefaultAdapter } from "@cmmv/http";
import { AuthModule } from "@cmmv/auth";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        GraphQLModule,
        AuthModule
    ],
    contracts: [...],
});
```

## Opções de Configuração

O módulo GraphQL permite personalizar suas configurações por meio do arquivo de configuração da aplicação.

Para personalizar as configurações do GraphQL, atualize seu arquivo `.cmmv.config.cjs`:

```javascript
module.exports = {
    server: {
        host: process.env.HOST || "0.0.0.0",
        port: process.env.PORT || 3000,
    },

    graphql: {
        port: process.env.GRAPHQL_PORT || 4000,
    },

    ...
};
```

Com essa configuração, o servidor GraphQL escutará na porta configurada, mantendo a compatibilidade com o restante do framework CMMV.
