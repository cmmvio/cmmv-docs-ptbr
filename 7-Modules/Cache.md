# Cache

O módulo ``@cmmv/cache`` integra-se ao ``cache-manager`` para fornecer gerenciamento de cache na memória. Por padrão, ele inclui suporte ao Redis usando o pacote ``@tirke/node-cache-manager-ioredis``, mas também suporta outras stores de cache compatíveis com o ``cache-manager``.

Para instalar o módulo ``@cmmv/cache``, use o seguinte comando:

```bash
$ pnpm add @cmmv/cache cache-manager
$ pnpm add -D @types/cache-manager
```

Se você planeja usar Redis ou outras stores, precisará instalar o pacote apropriado com base na store de cache que pretende usar.

**Stores de Cache e Drivers Suportados**

O ``cache-manager`` suporta várias stores de cache, e você pode configurar sua store com base nos seus requisitos. Aqui estão as stores suportadas e seus respectivos drivers:

**Redis:** [Github](https://github.com/Tirke/node-cache-manager-stores/tree/main)

```bash
$ pnpm add @tirke/node-cache-manager-ioredis
```

**Memcached:** [Github](https://github.com/theogravity/node-cache-manager-memcached-store)

```bash
$ pnpm add cache-manager-memcached-store
```

**MongoDB:** [Github](https://github.com/v4l3r10/node-cache-manager-mongodb)

```bash
$ pnpm add cache-manager-mongodb
```

**Filesystem Binary:** [Github](https://github.com/sheershoff/node-cache-manager-fs-binary)

```bash
$ pnpm add cache-manager-fs-binary
```

## Configuração

```javascript
module.exports = {
   // Outras configurações

   cache: {
        store: "@tirke/node-cache-manager-ioredis",
        getter: "ioRedisStore",
        host: "localhost",
        port: 6379,
        ttl: 600 // Tempo de vida (TTL) do cache em segundos
    },
}
```

<br/>

* **store:** Define o tipo de store de cache (neste caso, ``cache-manager-redis``).
* **host:** Especifica o host do Redis.
* **port:** Especifica a porta do Redis.
* **ttl:** Define o tempo de vida padrão para entradas de cache (em segundos).

Para mais detalhes sobre as configurações possíveis, visite [NPM](https://www.npmjs.com/package/cache-manager).

## Decorador de Cache

Neste exemplo, demonstramos como implementar manualmente o cache usando o módulo ``@cmmv/cache`` em um controlador gerado pelo ``@cmmv/http``. O cache pode ser adicionado em vários níveis, incluindo endpoints específicos ou manualmente após certas operações, como adicionar ou atualizar registros.

* **Controller:** O ``TaskController`` é responsável por gerenciar requisições relacionadas a tarefas.
* **Cache Decorator:** O decorador ``@Cache`` é usado para armazenar respostas em cache automaticamente para endpoints ``GET``. Neste caso, ``task:getAll`` e ``task:{id}`` são armazenados em cache por 300 segundos com compressão.
* **CacheService:** Para operações ``POST``, ``PUT`` e ``DELETE``, o cache é atualizado ou removido manualmente usando a API do ``CacheService``.

```typescript
// Gerado automaticamente pelo CMMV
    
import { Telemetry } from "@cmmv/core";  
import { Cache, CacheService } from "@cmmv/cache"; 

import { 
    Controller, Get, Post, Put, Delete, 
    Queries, Param, Body, Request 
} from '@cmmv/http';

import { TaskService } from '../services/task.service';
import { Task } from '../models/task.model';

@Controller('task')
export class TaskController {
    constructor(private readonly taskservice: TaskService) {}

    // GET All Tasks com Cache
    @Get()
    @Cache("task:getAll", { ttl: 300, compress: true })
    async getAll(@Queries() queries: any, @Request() req): Promise<Task[]> {
        Telemetry.start('TaskController::GetAll', req.requestId);
        let result = await this.taskservice.getAll(queries, req);
        Telemetry.end('TaskController::GetAll', req.requestId);
        return result;
    }

    // GET Task por ID com Cache
    @Get(':id')
    @Cache("task:{id}", { ttl: 300, compress: true })
    async getById(@Param('id') id: string, @Request() req): Promise<Task> {
        Telemetry.start('TaskController::GetById', req.requestId);
        let result = await this.taskservice.getById(id, req);
        Telemetry.end('TaskController::GetById', req.requestId);
        return result;
    }

    // POST (Adicionar Tarefa) e Atualizar Cache
    @Post()
    async add(@Body() item: Task, @Request() req): Promise<Task> {
        Telemetry.start('TaskController::Add', req.requestId);
        let result = await this.taskservice.add(item, req);
        
        // Armazena o resultado da nova tarefa criada no cache
        CacheService.set(`task:\${result.id}`, JSON.stringify(result), 300);
        
        Telemetry.end('TaskController::Add', req.requestId);
        return result;
    }

    // PUT (Atualizar Tarefa) e Atualizar Cache
    @Put(':id')
    async update(
        @Param('id') id: string, @Body() item: Task, @Request() req
    ): Promise<Task> {
        Telemetry.start('TaskController::Update', req.requestId);
        let result = await this.taskservice.update(id, item, req);
        
        // Atualiza o cache com a tarefa atualizada
        CacheService.set(`task:\${result.id}`, JSON.stringify(result), 300);
        
        Telemetry.end('TaskController::Update', req.requestId);
        return result;
    }

    // DELETE (Remover Tarefa) e Limpar Cache
    @Delete(':id')
    async delete(
        @Param('id') id: string, @Request() req
    ): Promise<{ success: boolean, affected: number }> {
        Telemetry.start('TaskController::Delete', req.requestId);
        let result = await this.taskservice.delete(id, req);
        
        // Remove a tarefa deletada do cache
        CacheService.del(`task:\${id}`);
        
        Telemetry.end('TaskController::Delete', req.requestId);
        return result;
    }
}
```

**Armazenamento em Cache em Requisições GET:**

* **Cache Automático:** O decorador ``@Cache`` é aplicado aos métodos ``getAll`` e ``getById``. Os resultados armazenados em cache são recuperados automaticamente com base no padrão de chave, reduzindo a carga no banco de dados e melhorando o desempenho.
* **Chave de Cache:** As chaves de cache são definidas como ``task:getAll`` para recuperar todas as tarefas e ``task:{id}`` para recuperar uma tarefa pelo ID.
* **TTL (Tempo de Vida):** Os resultados armazenados em cache são válidos por 300 segundos.

**Gerenciamento Manual de Cache para ``POST``, ``PUT`` e ``DELETE``:**

* Para ``POST`` (criando uma nova tarefa) e ``PUT`` (atualizando uma tarefa), o método ``CacheService.set`` é usado para atualizar manualmente o cache com a tarefa nova ou atualizada.
* Para ``DELETE`` (removendo uma tarefa), o método ``CacheService.del`` é usado para remover a entrada correspondente do cache após a exclusão da tarefa.
* **Telemetry:** Rastreamento e registro da execução de cada método, útil para monitoramento de desempenho.

Essa configuração otimiza o tempo de resposta armazenando em cache dados frequentemente acessados, enquanto garante que as entradas no cache sejam atualizadas ou limpas quando os dados são modificados.

## Propriedades do Cache

O módulo ``@cmmv/cache`` permite configurar propriedades diretamente no contrato usando a propriedade ``cache``. Essas propriedades determinam como o cache será gerenciado em controladores, gateways e métodos.

### Exemplo de Configuração no Contrato

Abaixo está um exemplo de como adicionar propriedades de cache em um contrato utilizando o decorador ``@Contract``.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    cache: { 
        key: "task:",   // Prefixo da chave do cache
        ttl: 300,       // Tempo de vida (em segundos)
        compress: true  // Ativar compressão para dados armazenados
    }
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [
            {
                type: 'IsString',
                message: 'Etiqueta inválida',
            },
            {
                type: 'IsNotEmpty',
                message: 'Etiqueta não pode estar vazia',
            },
        ],
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [
            {
                type: 'IsBoolean',
                message: 'O valor de checked deve ser booleano',
            },
        ],
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [
            {
                type: 'IsBoolean',
                message: 'O valor de removed deve ser booleano',
            },
        ],
    })
    removed: boolean;
}
```

### Propriedades Disponíveis para Cache

* **key:** Define o prefixo usado para gerar a chave do cache. Por exemplo, no contrato acima, o prefixo ``task:`` será usado para todas as operações relacionadas a tarefas.
* **ttl:** Especifica o tempo de vida do cache (em segundos). Após esse período, a entrada no cache expira.
* **compress:** Ativa a compressão dos dados antes de armazená-los no cache, reduzindo o uso de memória.

### Exemplo de Controlador Gerado com Cache

Com base no contrato acima, o controlador gerado incluirá lógica para gerenciar o cache automaticamente usando o decorador ``@Cache`` e o serviço ``CacheService``.

```typescript
// Gerado automaticamente pelo CMMV

import { Telemetry } from "@cmmv/core";  
import { Cache, CacheService } from "@cmmv/cache"; 

import { 
    Controller, Get, Post, Put, Delete, 
    Queries, Param, Body, Request 
} from '@cmmv/http';

import { TaskService } from '../services/task.service';
import { Task } from '../models/task.model';

@Controller('task')
export class TaskController {
    constructor(private readonly taskservice: TaskService) {}

    @Get()
    @Cache("task:getAll", { ttl: 300, compress: true })
    async getAll(@Queries() queries: any, @Request() req): Promise<Task[]> {
        Telemetry.start('TaskController::GetAll', req.requestId);
        let result = await this.taskservice.getAll(queries, req);
        Telemetry.end('TaskController::GetAll', req.requestId);
        return result;
    }

    @Get(':id')
    @Cache("task:{id}", { ttl: 300, compress: true })
    async getById(@Param('id') id: string, @Request() req): Promise<Task> {
        Telemetry.start('TaskController::GetById', req.requestId);
        let result = await this.taskservice.getById(id, req);
        Telemetry.end('TaskController::GetById', req.requestId);
        return result;
    }

    @Post()
    async add(@Body() item: Task, @Request() req): Promise<Task> {
        Telemetry.start('TaskController::Add', req.requestId);
        let result = await this.taskservice.add(item, req);
        CacheService.set(`task:\${result.id}`, JSON.stringify(result), 300);
        Telemetry.end('TaskController::Add', req.requestId);
        return result;
    }

    @Put(':id')
    async update(
        @Param('id') id: string, @Body() item: Task, @Request() req
    ): Promise<Task> {
        Telemetry.start('TaskController::Update', req.requestId);
        let result = await this.taskservice.update(id, item, req);
        CacheService.set(`task:\${result.id}`, JSON.stringify(result), 300);
        Telemetry.end('TaskController::Update', req.requestId);
        return result;
    }

    @Delete(':id')
    async delete(
        @Param('id') id: string, @Request() req
    ): Promise<{ success: boolean, affected: number }> {
        Telemetry.start('TaskController::Delete', req.requestId);
        let result = await this.taskservice.delete(id, req);
        CacheService.del(`task:\${id}`);
        Telemetry.end('TaskController::Delete', req.requestId);
        return result;
    }
}
```

Este exemplo mostra como o ``@Cache`` e o ``CacheService`` são utilizados para otimizar as operações em um controlador. Respostas para métodos ``GET`` são armazenadas em cache automaticamente, enquanto o cache é atualizado ou removido manualmente para métodos ``POST``, ``PUT`` e ``DELETE``.

## Benefícios do Uso de Cache

* **Desempenho Melhorado:** Reduz chamadas redundantes ao banco de dados e melhora o tempo de resposta.
* **Gerenciamento Centralizado:** Configuração de cache em contratos facilita a manutenção e o gerenciamento.
* **Flexibilidade:** Suporte para várias stores e gerenciamento manual de entradas de cache permite personalização.

Para mais informações, consulte a [documentação oficial do CMMV](https://github.com/andrehrferreira/cmmv).
