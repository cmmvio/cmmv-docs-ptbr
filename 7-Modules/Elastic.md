# Elastic

Repositório: [https://github.com/cmmvio/cmmv-elastic](https://github.com/cmmvio/cmmv-elastic)

O módulo ``@cmmv/elastic`` oferece uma integração perfeita com o [Elastic](https://www.elastic.co/pt/) para gerenciar índices, manipular documentos e executar consultas de busca. Este módulo foi projetado para ser escalável e eficiente, integrando-se perfeitamente ao framework ``@cmmv`` para suportar aplicações modernas orientadas a dados.

## Recursos

* **Gerenciamento de Índices:** Simplifica a criação, exclusão, verificação e gerenciamento de índices Elasticsearch.
* **Operações com Documentos:** Realiza operações CRUD em documentos Elasticsearch.
* **Execução de Buscas e Consultas:** Executa consultas de busca avançadas com suporte a pontuação e filtragem.
* **Suporte a Alias e Rollover:** Gerencia aliases para roteamento flexível de índices e realiza rollovers para armazenamento escalável de dados.
* **Integração com o Framework CMMV:** Suporte integrado para contratos e modelos do ``@cmmv``.
* **Registro e Tratamento de Erros:** Garante um gerenciamento robusto de erros e depuração.

## Instalação

Para instalar o módulo ``@cmmv/elastic``:

```bash
$ pnpm add @cmmv/elastic
```

## Configuração

O módulo ``@cmmv/elastic`` requer uma configuração do Elasticsearch, que pode ser definida no arquivo ``.cmmv.config.cjs``:

```javascript
import * as fs from "node:fs";

module.exports = {
    env: process.env.NODE_ENV,

    elastic: {
        node: 'http://localhost:9200', // URL do nó Elasticsearch
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

No seu arquivo ``index.ts``, inclua o ``ElasticModule`` e seus serviços para uma integração perfeita com a aplicação:

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

Permite criar um novo índice Elasticsearch com configurações ajustáveis, como o número de shards e réplicas. Use isso para estruturar o armazenamento de dados.

```typescript
await ElasticService.createIndex('meu-indice', {
    number_of_shards: 1,
    number_of_replicas: 0,
});
```

### Verificando a Existência de um Índice

Verifica se um índice específico existe no Elasticsearch. Útil para operações condicionais para evitar a criação duplicada de índices.

```typescript
const existe = await ElasticService.checkIndexExists('meu-indice');
console.log(`Índice existe: \${existe}`);
```

### Excluindo um Índice

Exclui um índice existente do Elasticsearch. Use isso para remover índices desatualizados ou desnecessários.

```typescript
await ElasticService.deleteIndex('meu-indice');
```

### Inserindo Documentos

Adiciona um novo documento a um índice especificado. Ideal para armazenar e indexar dados estruturados para recuperação e análise.

```typescript
await ElasticService.insertDocument('meu-indice', {
    id: 1,
    nome: 'Documento de Teste'
});
```

### Atualizando Documentos

Atualiza um documento existente em um índice especificado. Use isso para modificar dados armazenados sem criar duplicatas.

```typescript
await ElasticService.updateDocument('meu-indice', 'id-do-documento', {
    nome: 'Nome Atualizado'
});
```

### Buscando Documentos

Realiza uma consulta de busca em um índice especificado. Use isso para recuperar documentos que correspondam a critérios específicos.

```typescript
const resultados = await ElasticService.searchDocuments('meu-indice', {
    query: { match: { nome: 'Documento de Teste' } },
});
console.log(resultados);
```

### Gerenciamento de Alias

Cria ou verifica a existência de um alias para um índice. Aliases são úteis para abstrair o acesso a índices, permitindo um gerenciamento de dados flexível.

```typescript
await ElasticService.createAlias('meu-indice');
const aliasExiste = await ElasticService.checkAliasExists('meu-alias');
```

### Rollover de Índice

Realiza uma operação de rollover em um alias, criando um novo índice quando condições específicas são atendidas (por exemplo, número máximo de documentos). Útil para gerenciar armazenamento de dados em grande escala.

```typescript
await ElasticService.performRollover('meu-alias', { max_docs: 1000 });
```
