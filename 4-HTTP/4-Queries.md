# Consultas

A função ``findAll`` na camada de repositório foi aprimorada para suportar filtros dinâmicos de consulta e é compatível com bancos de dados SQL e MongoDB. O sistema de cache fornecido por ``@cmmv/cache`` também foi atualizado para suportar múltiplos filtros, melhorando a flexibilidade e o desempenho.

Busca todos os registros de uma entidade especificada, aplicando filtros, paginação e ordenação dinamicamente com base nas ``queries`` fornecidas. Este método é compatível com SQL e MongoDB, adaptando seu comportamento com base no tipo de banco de dados.

| **Parâmetro**       | **Tipo**    | **Descrição**                                                                                   | **Padrão**  |
|---------------------|-------------|--------------------------------------------------------------------------------------------------|-------------|
| `limit`           | *(número)*  | Especifica o número máximo de resultados a serem retornados.                                    | `10`      |
| `offset`          | *(número)*  | Especifica o número de resultados a serem ignorados.                                            | `0`       |
| `sortBy`          | *(string)*  | Especifica o campo pelo qual os resultados devem ser ordenados.                                 | `'id'`    |
| `sort`            | *(string)*  | Especifica a ordem de classificação. Valores possíveis são `'asc'` e `'desc'`.              | `'asc'`   |
| `search`          | *(string)*  | Realiza uma pesquisa case-insensitive no campo especificado.                                    | -           |
| `searchField`     | *(string)*  | Especifica o campo onde a pesquisa deve ser realizada.                                          | -           |
| **Filtros Adicionais** | *(variado)* | Quaisquer parâmetros adicionais de consulta são aplicados dinamicamente como filtros baseados nos nomes dos campos da entidade. | -           |

1. Manipulação de Consultas SQL:

    * Converte ``search`` e ``searchField`` em condições ``LIKE``.
    * Constrói dinamicamente a cláusula ``WHERE`` com base nos ``filters``.
    * Utiliza ``LIMIT``, ``OFFSET`` e ``ORDER BY`` para paginação e ordenação.

2. Manipulação de Consultas MongoDB:

    * Utiliza ``$regex`` para buscas case-insensitive.
    * Aplica filtros dinamicamente como parte da consulta ``find`` do MongoDB.
    * Suporta ``skip``, ``limit`` e ordenação utilizando a sintaxe compatível com o MongoDB.

### Exemplo

```typescript
const tasks = await Repository.findAll(TaskEntity, {
    limit: 20,
    offset: 0,
    sortBy: 'createdAt',
    sort: 'desc',
    search: 'John',
    searchField: 'name',
    status: 'active',
}); 
```

## Cache

Se o módulo ``@cmmv/cache`` estiver configurado no sistema, o mecanismo de cache ajusta dinamicamente a chave de cache para incluir os parâmetros de consulta, como ``search``, garantindo que os resultados filtrados sejam armazenados em cache sob chaves únicas. Isso permite que o sistema armazene e recupere resultados de maneira eficiente, mesmo quando as consultas envolvem filtros dinâmicos como ``limit``, ``offset`` ou ``searchField``.

Quando um método do controlador é decorado com ``@Cache``, os seguintes passos ocorrem:

1. **Chave Base do Cache:**
A chave base fornecida no decorador (por exemplo, ``"task:getAll"``) é utilizada.

2. **Parâmetros de Consulta:**
Se a requisição inclui parâmetros de consulta, como ``search`` ou ``searchField``, eles são adicionados à chave de cache. Por exemplo:

* Requisição:
``GET /api/tasks?search=John&searchField=name&limit=10``
* Chave de Cache:
``task:getAll:search=John&searchField=name&limit=10``

### Exemplo

```typescript
@Get()
@Cache("task:getAll", { ttl: 300, compress: true, schema: TaskFastSchema })
async getAll(@Queries() queries: any, @Request() req) {
    Telemetry.start('TaskController::GetAll', req.requestId);
    const result = await this.taskservice.getAll(queries, req);
    Telemetry.end('TaskController::GetAll', req.requestId);
    return result;
}
```

## Exemplo de Fluxo

O cliente envia uma requisição GET com parâmetros de consulta:

```
GET /api/tasks?search=John&searchField=name&limit=5
```

**Controlador**

```typescript
@Get()
@Cache("task:getAll", { ttl: 300, compress: true, schema: TaskFastSchema })
async getAll(@Queries() queries: any, @Request() req) {
    return await this.taskservice.getAll(queries, req);
}
```

**SQL**

```sql
SELECT * FROM tasks WHERE LOWER(name) LIKE LOWER('%John%') LIMIT 5 OFFSET 0;
```

**MongoDB**

```javascript
{ name: { $regex: /John/i } }
```

**Chave de Cache**

```
task:getAll:search=John&searchField=name&limit=5
```