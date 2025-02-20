# Elastic

O módulo ``@cmmv/elastic`` oferece uma integração completa com o [Elastic](https://www.elastic.co/pt/) para gerenciamento de índices, manipulação de documentos e execução de consultas de busca. Este módulo foi projetado para ser escalável e eficiente, integrando-se perfeitamente ao framework ``@cmmv`` para suportar aplicações modernas orientadas a dados.

## Recursos

* **Gerenciamento de Índices:** Simplifica a criação, exclusão, verificação e gerenciamento de índices do Elasticsearch.
* **Operações com Documentos:** Realize operações de CRUD em documentos no Elasticsearch.
* **Busca e Execução de Consultas:** Execute consultas avançadas de busca com suporte a pontuação e filtros.
* **Suporte a Alias e Rollover:** Gerencie aliases para roteamento flexível de índices e execute rollovers para armazenamento escalável de dados.
* **Integração com o Framework CMMV:** Suporte integrado para contratos e modelos do ``@cmmv``.
* **Registro de Erros e Tratamento:** Garante um gerenciamento robusto de erros e depuração.

## Instalação

Para instalar o módulo ``@cmmv/elastic``:

```bash
$ pnpm add @cmmv/elastic
```

## Configuração

O módulo ``@cmmv/elastic`` exige uma configuração do Elasticsearch, que pode ser definida no arquivo ``.cmmv.config.cjs``:

```javascript
import * as fs from "node:fs";

module.exports = {
    env: process.env.NODE_ENV,

    elastic: {
        node: 'http://localhost:9200', // URL do nó do Elasticsearch
        //cloud: { id: '<cloud-id>' }, // Opcional para Elastic Cloud
        /*
        tls: {
            ca: fs.readFileSync('./http_ca.crt'),
            rejectUnauthorized: false
        },
        auth: {
            bearer: process.env.ELASTIC_BEARER || "",
            apiKey: process.env.ELASTIC_APIKEY || "",
            username: process.env.ELASTIC_USERNAME || "",
            password: process.env.ELASTIC_PASSWORD || ""
        }
        */
    }
};
```

## Configurando a Aplicação

No seu ``index.ts``, inclua o ``ElasticModule`` e seus serviços para integração com a aplicação:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { ElasticModule, ElasticService } from "@cmmv/elastic";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        ElasticModule,
    ],
    services: [ElasticService],
});
```

## Uso

### Criando um Índice

Permite criar um novo índice no Elasticsearch com configurações personalizáveis, como o número de shards e réplicas. Use isso para estruturar seu armazenamento de dados.

```typescript
await ElasticService.createIndex('my-index', {
    number_of_shards: 1,
    number_of_replicas: 0,
});
```

### Verificando Existência de Índice

Verifica se um índice específico existe no Elasticsearch. Útil para operações condicionais para evitar a criação de índices duplicados.

```typescript
const exists = await ElasticService.checkIndexExists('my-index');
console.log(`O índice existe: \${exists}`);
```

### Excluindo um Índice

Exclui um índice existente no Elasticsearch. Use isso para remover índices desatualizados ou desnecessários.

```typescript
await ElasticService.deleteIndex('my-index');
```

### Inserindo Documentos

Adiciona um novo documento a um índice especificado. Ideal para armazenar e indexar dados estruturados para recuperação e análise.

```typescript
await ElasticService.insertDocument('my-index', { 
    id: 1, 
    name: 'Documento de Teste' 
});
```

### Atualizando Documentos

Atualiza um documento existente em um índice especificado. Use isso para modificar dados armazenados sem criar duplicatas.

```typescript
await ElasticService.updateDocument('my-index', 'document-id', { 
    name: 'Nome Atualizado' 
});
```

### Buscando Documentos

Executa uma consulta de busca em um índice especificado. Use isso para recuperar documentos que correspondam a critérios específicos.

```typescript
const results = await ElasticService.searchDocuments('my-index', {
    query: { match: { name: 'Documento de Teste' } },
});
console.log(results);
```

### Gerenciamento de Alias

Cria ou verifica a existência de um alias para um índice. Aliases são úteis para abstrair o acesso aos índices, permitindo um gerenciamento flexível de dados.

```typescript
await ElasticService.createAlias('my-index');
const aliasExists = await ElasticService.checkAliasExists('my-alias');
```

### Rollover de Índice

Executa uma operação de rollover em um alias, criando um novo índice quando condições específicas são atendidas (por exemplo, número máximo de documentos). Útil para gerenciar o armazenamento de dados em grande escala.

```typescript
await ElasticService.performRollover('my-alias', { max_docs: 1000 });
```