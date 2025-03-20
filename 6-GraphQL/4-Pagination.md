# Paginação

Para manter a consistência com **APIs RESTful**, o **`@cmmv/graphql`** implementa **suporte integrado à paginação**.
Como os resolvers GraphQL utilizam os mesmos **serviços que REST e RPC**, a paginação é **aplicada automaticamente** às **consultas de busca**.

Isso garante:
✅ **Integração perfeita** entre GraphQL, REST e RPC.
✅ **Parâmetros de paginação padronizados** em todas as APIs.
✅ **Opções flexíveis de filtragem, ordenação e busca**.

## Como Funciona a Paginação

O sistema define dois tipos relacionados à paginação:

- **`PaginationArgs`** → Usado em consultas para solicitar dados paginados.
- **`PaginationResponse`** → Define a estrutura de resposta para resultados paginados.

**Todos os resolvers de busca (`find`) gerados automaticamente** **suportam paginação automaticamente**, o que significa que os desenvolvedores não precisam configurá-la manualmente.

## Parâmetros de Requisição de Paginação

As consultas GraphQL usam o tipo **`PaginationArgs`**, que fornece parâmetros padrão para controlar os resultados.

```typescript
import { ArgsType, Field, Int } from "@cmmv/graphql";
import GraphQLJSON from "graphql-type-json";

@ArgsType()
export class PaginationArgs {
    @Field(() => Int, {
        description: "Máximo de itens por página.",
        nullable: true,
        defaultValue: 10
    })
    limit?: number;

    @Field(() => Int, {
        description: "Número de itens pulados.",
        nullable: true,
        defaultValue: 0
    })
    offset?: number;

    @Field({
        description: "Campo usado para ordenação.",
        nullable: true,
        defaultValue: "id"
    })
    sortBy?: string;

    @Field({
        description: "Ordem de ordenação: 'asc' ou 'desc'.",
        nullable: true,
        defaultValue: "asc"
    })
    sort?: string;

    @Field({
        description: "Termo de busca.",
        nullable: true
    })
    search?: string;

    @Field({
        description: "Campo onde a busca é aplicada.",
        nullable: true
    })
    searchField?: string;

    @Field(() => GraphQLJSON, {
        description: "Opções de filtragem flexíveis.",
        nullable: true
    })
    filters?: Record<string, any>;
}
```

## Explicação dos Parâmetros de Consulta

| Parâmetro    | Tipo    | Padrão  | Descrição |
|-------------|--------|---------|-----------|
| `limit`   | Int    | **10**  | Número máximo de itens por página. |
| `offset`  | Int    | **0**   | Número de itens a serem pulados antes de retornar os resultados. |
| `sortBy`  | String | **id**  | Campo usado para ordenar os resultados. |
| `sort`    | String | **asc** | Ordem de ordenação (**asc** ou **desc**). |
| `search`  | String | _null_  | Termo geral de busca para filtrar resultados. |
| `searchField` | String | _null_ | Campo específico onde a busca deve ser aplicada. |
| `filters` | JSON   | _null_  | Objeto JSON com opções de filtragem flexíveis. |

O campo **`filters`** suporta **filtragem complexa**, permitindo **múltiplas condições e campos dinâmicos** usando **`graphql-type-json`**.

## Estrutura de Resposta de Paginação

Consultas paginadas retornam um objeto **`PaginationResponse`**, contendo os itens solicitados e metadados.

```typescript
import { ObjectType, Field, Int } from "@cmmv/graphql";
import GraphQLJSON from "graphql-type-json";

@ObjectType()
export class PaginationResponse {
    @Field(() => Int, { description: "Máximo de itens por página." })
    limit!: number;

    @Field(() => Int, { description: "Número de itens pulados." })
    offset!: number;

    @Field({ description: "Campo de ordenação." })
    sortBy!: string;

    @Field({ description: "Ordem de ordenação: 'asc' ou 'desc'." })
    sort!: string;

    @Field({ description: "Termo de busca aplicado.", nullable: true })
    search?: string;

    @Field({ description: "Campo onde a busca foi aplicada.", nullable: true })
    searchField?: string;

    @Field(() => GraphQLJSON, { description: "Filtros aplicados.", nullable: true })
    filters?: Record<string, any>;
}
```

## Explicação dos Campos de Resposta

| Campo       | Tipo    | Descrição |
|------------|--------|-----------|
| `limit`   | Int    | Número de itens por página. |
| `offset`  | Int    | Número de itens pulados. |
| `sortBy`  | String | Campo usado para ordenação. |
| `sort`    | String | Ordem de ordenação (**asc** ou **desc**). |
| `search`  | String | Termo de busca aplicado (se houver). |
| `searchField` | String | Campo onde a busca foi realizada. |
| `filters` | JSON   | Objeto JSON com condições de filtro aplicadas. |

## Consulta GraphQL Paginada

Os desenvolvedores podem usar **consultas GraphQL** para buscar resultados paginados com ordenação e filtragem.

```graphql
query {
    userFind(limit: 5, offset: 0, sortBy: "createdAt", sort: "desc", search: "João") {
        count
        data {
            id
            name
            email
        }
        pagination {
            limit
            offset
            sortBy
            sort
            search
            searchField
            filters
        }
    }
}
```

**Resposta**

```json
{
    "data": {
        "userFind": {
            "count": 100,
            "data": [
                { "id": "1", "name": "João Silva", "email": "joao@exemplo.com" },
                { "id": "2", "name": "João Santos", "email": "santos@exemplo.com" }
            ],
            "pagination": {
                "limit": 5,
                "offset": 0,
                "sortBy": "createdAt",
                "sort": "desc",
                "search": "João",
                "searchField": null,
                "filters": null
            }
        }
    }
}
```

### Filtragem Avançada com JSON

O campo **`filters`** permite filtragem avançada, suportando **múltiplas condições** e **campos dinâmicos**.

```graphql
query {
    userFind(filters: { role: "admin", status: "ativo" }) {
        count
        data { id name role status }
    }
}
```

**Resposta**

```json
{
    "data": {
        "userFind": {
            "count": 3,
            "data": [
                { "id": "1", "name": "Alice", "role": "admin", "status": "ativo" },
                { "id": "2", "name": "Bob", "role": "admin", "status": "ativo" }
            ]
        }
    }
}
```

A paginação GraphQL no `@cmmv/graphql` garante:
- **Suporte unificado entre REST, RPC e GraphQL**.
- **Recuperação eficiente de dados com limite, offset, ordenação e filtragem**.
- **Filtros baseados em JSON flexíveis para consultas avançadas**.
- **Suporte automático para consultas `find`** sem configuração adicional.

Ao incorporar a paginação diretamente no GraphQL, o **CMMV proporciona uma experiência fluida**, reduzindo a complexidade enquanto maximiza **desempenho e escalabilidade**.
