# Consultas

A função ``findAll`` na camada de repositório foi aprimorada para suportar filtragem dinâmica de consultas e é compatível com bancos de dados SQL e MongoDB. O sistema de cache fornecido pelo ``@cmmv/cache`` também foi atualizado para suportar múltiplos filtros, melhorando a flexibilidade e o desempenho.

Busca todos os registros de uma entidade especificada, aplicando filtros, paginação e ordenação de forma dinâmica com base nas ``queries`` fornecidas. Este método é compatível com SQL e MongoDB, adaptando seu comportamento conforme o tipo de banco de dados.

| **Parâmetro**      | **Tipo**   | **Descrição**                                                                                  | **Padrão** |
|---------------------|------------|------------------------------------------------------------------------------------------------|------------|
| `limit`            | *(number)* | Especifica o número máximo de resultados a serem retornados.                                   | `10`       |
| `offset`           | *(number)* | Especifica o número de resultados a serem pulados.                                             | `0`        |
| `sortBy`           | *(string)* | Especifica o campo pelo qual os resultados devem ser ordenados.                                | `'id'`     |
| `sort`             | *(string)* | Especifica a ordem de classificação. Valores possíveis são `'asc'` e `'desc'`.                 | `'asc'`    |
| `search`           | *(string)* | Realiza uma busca insensível a maiúsculas e minúsculas no campo especificado.                  | -          |
| `searchField`      | *(string)* | Especifica o campo no qual a operação de busca será realizada.                                 | -          |
| **Filtros Adicionais** | *(variado)* | Quaisquer parâmetros de consulta adicionais são aplicados dinamicamente como filtros com base nos nomes dos campos da entidade. | -          |

1. **Manuseio de Consultas SQL:**

    * Converte ``search`` e ``searchField`` em condições ``LIKE``.
    * Constrói dinamicamente a cláusula ``WHERE`` com base em ``filters``.
    * Usa ``LIMIT``, ``OFFSET`` e ``ORDER BY`` para paginação e ordenação.

2. **Manuseio de Consultas MongoDB:**

    * Usa ``$regex`` para buscas insensíveis a maiúsculas e minúsculas.
    * Aplica filtros dinamicamente como parte da consulta ``find`` do MongoDB.
    * Suporta ``skip``, ``limit`` e ordenação usando sintaxe compatível com MongoDB.

### Exemplo

<br/>

```typescript
const tasks = await Repository.findAll(TaskEntity, {
    limit: 20,
    offset: 0,
    sortBy: 'createdAt',
    sort: 'desc',
    search: 'João',
    searchField: 'nome',
    status: 'ativo',
});
```

## Cache

Se o módulo ``@cmmv/cache`` estiver configurado no sistema, o mecanismo de cache ajusta dinamicamente a chave de cache para incluir parâmetros de consulta, como ``search``, garantindo que os resultados filtrados sejam armazenados em cache sob chaves únicas. Isso permite que o sistema armazene e recupere resultados de forma eficiente, mesmo quando as consultas envolvem filtros dinâmicos como ``limit``, ``offset`` ou ``searchField``.

Quando um método de controlador é decorado com ``@Cache``, os seguintes passos ocorrem:

1. **Chave de Cache Base:**
A chave base fornecida no decorador (ex.: ``"task:getAll"``) é usada.

2. **Parâmetros de Consulta:**
Se a requisição incluir parâmetros de consulta como ``search`` ou ``searchField``, eles são anexados à chave de cache. Por exemplo:

* Requisição:
``GET /api/tasks?search=João&searchField=nome&limit=10``
* Chave de Cache:
``task:getAll:search=João&searchField=nome&limit=10``

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

## Fluxo de Exemplo

O cliente envia uma requisição GET com parâmetros de consulta:

```
GET /api/tasks?search=João&searchField=nome&limit=5
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
SELECT * FROM tasks WHERE LOWER(nome) LIKE LOWER('%João%') LIMIT 5 OFFSET 0;
```

**MongoDB**

```javascript
{ nome: { $regex: /João/i } }
```

**Chave de Cache**

```
task:getAll:search=João&searchField=nome&limit=5
```
