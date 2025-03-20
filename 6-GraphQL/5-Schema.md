# Esquema

O módulo **@cmmv/graphql** gera automaticamente um **esquema GraphQL** no formato padrão `.graphql`.
Esse esquema é salvo no **diretório `.generated/`** e é **recriado toda vez que o sistema é reiniciado**.

⚠️ **Importante:**
Esse arquivo é **regenerado automaticamente** — quaisquer modificações manuais serão **sobrescritas** na próxima recarga.
Por causa disso, o desenvolvimento **Schema-First não é possível**, e todos os tipos ou resolvers personalizados devem ser definidos usando **Code-First** ou por meio de **contratos**.

## Como o Esquema é Gerado

Durante a inicialização do sistema, o **@cmmv/graphql** realiza os seguintes passos:

1️⃣ Lê todos os **contratos do sistema** e **resolvers personalizados**.
2️⃣ Gera um **esquema GraphQL** no formato `.graphql`.
3️⃣ Salva o arquivo de esquema no **diretório `.generated/`**.
4️⃣ Toda vez que o sistema **reinicia**, o esquema é **atualizado automaticamente**.

Isso garante **consistência** entre os serviços, permitindo que ferramentas externas ou microsserviços **consumam o esquema** quando necessário.

## Localização do Esquema

O arquivo de esquema gerado está localizado em:

```
.generated/schema.graphql
```

## Exemplo

Aqui está um exemplo de como o **esquema GraphQL** pode parecer:

```graphql
type User {
    id: ID!
    username: String!
    email: String!
    roles: [String!]!
}

type Query {
    userFind(limit: Int = 10, offset: Int = 0, sortBy: String = "id", sort: String = "asc", search: String, searchField: String, filters: JSON): PaginationUserReturn
    userById(id: ID!): User
}

type Mutation {
    createUser(username: String!, password: String!, profile: JSON): User
    updateUser(id: ID!, username: String!, password: String!, profile: JSON): Boolean
    deleteUser(id: ID!): Boolean
}

scalar JSON

type PaginationUserReturn {
    count: Int!
    data: [User!]!
    pagination: PaginationResponse!
}

type PaginationResponse {
    limit: Int!
    offset: Int!
    sortBy: String!
    sort: String!
    search: String
    searchField: String
    filters: JSON
}
```

### Desenvolvimento Schema-First Não é Suportado

Como o esquema é gerado **automaticamente**, o **desenvolvimento Schema-First no GraphQL não é viável** no CMMV.
Em vez disso, os desenvolvedores devem usar **Code-First** ou **Desenvolvimento Baseado em Contratos**.

### ❌ Schema-First (Não Suportado)
Definir esquemas manualmente em arquivos `.graphql` **não é possível**, pois o sistema **sobrescreve** o esquema a cada reinício.

### ✅ Code-First (Suportado)
Usando decoradores e TypeScript, os desenvolvedores podem criar **resolvers e tipos personalizados** dinamicamente.

```typescript
import { Resolver, Query, ObjectType, Field } from "@cmmv/graphql";

@ObjectType()
class CustomUser {
    @Field() id: string;
    @Field() username: string;
}

@Resolver()
class CustomResolver {
    @Query(() => CustomUser)
    getUser(): CustomUser {
        return { id: "1", username: "cmmv-user" };
    }
}
```

## Baseado em Contratos (Recomendado)

A **abordagem preferida** no **CMMV** é **definir contratos**, que são automaticamente convertidos em **tipos GraphQL, endpoints REST e eventos WebSocket**.

Exemplo de contrato:

```typescript
import { Contract, ContractField } from "@cmmv/core";

@Contract({ controller: "User" })
export class UserContract {
    @ContractField(() => String)
    username!: string;

    @ContractField(() => String, { nullable: true })
    email?: string;
}
```

Esse contrato será **incluído automaticamente no esquema gerado**.

Embora o **desenvolvimento Schema-First não seja possível**, o arquivo `schema.graphql` gerado ainda pode ser útil:

✅ **Documentação da API:** Serviços externos podem ler o esquema para **introspecção GraphQL**.
✅ **Integração de Microsserviços:** Outras aplicações podem usar o esquema como referência.
✅ **Clientes GraphQL:** Aplicações frontend podem **buscar o esquema dinamicamente** para consultas no lado do cliente.

Para explorar o esquema, você pode usar **ferramentas de introspecção GraphQL** como:

- **Apollo Sandbox** ([sandbox.apollo.dev](https://sandbox.apollo.dev))
- **GraphiQL** ([GraphQL Playground](https://www.graphql.com/graphiql))
- **Cliente GraphQL do Postman**

## Conclusão

- O **`@cmmv/graphql` gera automaticamente um esquema** em `.generated/schema.graphql`.
- **Modificações manuais não são permitidas**, pois o arquivo é sobrescrito a cada reinício.
- **Desenvolvimento Schema-First não é suportado** — os desenvolvedores devem usar abordagens **Code-First** ou **Baseadas em Contratos**.
- O esquema gerado pode ser usado para **documentação, introspecção e integração**.
