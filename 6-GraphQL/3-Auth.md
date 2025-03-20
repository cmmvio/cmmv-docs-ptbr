# Autenticação

Quando o módulo **`@cmmv/auth`** está presente no projeto, o **`@cmmv/graphql`** integra-se perfeitamente aos recursos de autenticação e controle de acesso. Essa integração garante **operações GraphQL seguras** sem exigir configuração adicional.

## Recursos de Autenticação Automática

Uma vez que ambos os módulos estão incluídos, o GraphQL **suporta automaticamente** todas as funcionalidades de autenticação disponíveis no `@cmmv/auth`, incluindo:

**Decorador `@Authorized`**:
   - Validação integrada de papéis e permissões para consultas e mutações.
   - Garante que os usuários só possam acessar endpoints GraphQL autorizados.

**Validação de Papéis do Usuário**:
   - Restringe o acesso aos resolvers GraphQL com base nos papéis do usuário.
   - Impede consultas e mutações não autorizadas.

**Gerenciamento de Tokens e Sessões**:
   - O **manuseio de tokens e refresh tokens** é totalmente automatizado.
   - Os usuários podem **fazer login e atualizar tokens** diretamente através do GraphQL.

**Suporte a Todos os Provedores de Autenticação**:
   - OAuth2 (Google, Facebook, GitHub, etc.).
   - Autenticação Multi-Fator (MFA).
   - Validação reCAPTCHA para maior segurança.

**Resolvers de Autenticação Integrados**:
   - O `@cmmv/graphql` fornece internamente resolvers de autenticação para login de usuário, atualização de token e gerenciamento de sessão.

## Usando o Decorador @Authorized

Com o `@cmmv/auth` habilitado, todos os resolvers suportam automaticamente **controle de autorização**.
Você pode restringir o acesso usando o **decorador `@Authorized`**, especificando os papéis necessários.

```typescript
import {
    Resolver, Query, Mutation,
    Arg, Authorized
} from "@cmmv/graphql";

@Resolver()
class UserResolver {
    @Query(() => String)
    @Authorized() // Apenas usuários autenticados podem acessar
    getSecretData(): string {
        return "Esta é uma consulta protegida!";
    }

    @Mutation(() => Boolean)
    @Authorized("admin") // Restringe a usuários com o papel "admin"
    deleteUser(@Arg("id") id: string): boolean {
        return true; // Operação de exclusão simulada
    }
}
```

## Gerenciamento de Tokens

O `@cmmv/graphql` **gerencia automaticamente a autenticação por token**, permitindo que os usuários façam login e atualizem seus tokens **diretamente através do GraphQL**.

```graphql
mutation {
    login(username: "admin", password: "password123") {
        token
        refreshToken
    }
}
```

## Controle de Acesso Baseado em Papéis (RBAC)

Por padrão, o `@cmmv/auth` fornece **controle de acesso baseado em papéis (RBAC)** dentro do GraphQL.
Os **papéis e permissões de cada usuário** são automaticamente **validados** ao chamar consultas ou mutações.

```typescript
import { Resolver, Query, Authorized, Ctx, GraphQLContext } from "@cmmv/graphql";

@Resolver()
class SecureResolver {
    @Query(() => String)
    @Authorized(["moderator", "admin"]) // Permite apenas papéis específicos
    getModeratorData(@Ctx() ctx: GraphQLContext): string {
        return `Olá, \${ctx.auth?.user?.username}! Você tem acesso.`;
    }
}
```

## Suporte a OAuth2, MFA e reCAPTCHA

Todos os **métodos de autenticação** do `@cmmv/auth` estão **totalmente disponíveis** no GraphQL **sem necessidade de configuração adicional**.
Isso significa que você pode **fazer login usando Google, GitHub e outros provedores OAuth2** ou até mesmo **exigir verificação MFA**.

| Recurso       | Disponível no GraphQL? |
|--------------|-----------------------|
| **Login OAuth2 (Google, GitHub, etc.)** | ✅ Sim |
| **Autenticação Multi-Fator (MFA)** | ✅ Sim |
| **Validação reCAPTCHA** | ✅ Sim |
| **Manuseio de JWT & Refresh Token** | ✅ Sim |
| **Controle de Acesso Baseado em Papéis (RBAC)** | ✅ Sim |

Ao integrar o **`@cmmv/graphql`** com o **`@cmmv/auth`**, as operações GraphQL tornam-se **totalmente autenticadas** com **zero configuração adicional**.
Isso garante que:

- **Todas as consultas e mutações são seguras** usando `@Authorized()`.
- **Os tokens são gerenciados automaticamente**, permitindo autenticação e atualização contínuas.
- **OAuth2, MFA e controle de acesso baseado em papéis (RBAC) estão integrados**.
- **O GraphQL herda todos os recursos de autenticação** disponíveis no `@cmmv/auth`.
