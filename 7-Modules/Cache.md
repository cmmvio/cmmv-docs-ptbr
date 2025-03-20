# Cache

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/cache](https://github.com/cmmvio/cmmv/tree/main/packages/cache)

O módulo ``@cmmv/cache`` integra-se ao ``cache-manager`` para fornecer gerenciamento de cache em memória. Por padrão, inclui suporte ao Redis usando o pacote ``@tirke/node-cache-manager-ioredis``, mas também suporta outros armazenamentos de cache compatíveis com ``cache-manager``.

Para instalar o módulo ``@cmmv/cache``, use o npm:

```bash
$ pnpm add @cmmv/cache
```

Se você planeja usar Redis ou outros armazenamentos, será necessário instalar o pacote apropriado com base no armazenamento de cache que deseja usar.

**Armazenamentos de Cache Suportados e Drivers**

O ``cache-manager`` suporta vários armazenamentos de cache, e você pode configurar seu armazenamento com base em suas necessidades. Aqui estão os armazenamentos suportados e seus respectivos drivers:

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
        ttl: 600 // TTL (Time-to-Live) do cache em segundos
    },
}
```
<br/>

* **store:** Define o tipo de armazenamento de cache (``cache-manager-redis`` neste caso).
* **host:** Especifica o host do Redis.
* **port:** Especifica a porta do Redis.
* **ttl:** Define o tempo de vida padrão para entradas de cache (em segundos).

Para descobrir todas as configurações possíveis, visite [NPM](https://www.npmjs.com/package/cache-manager)

## Decorador de Cache

Neste exemplo, demonstramos como implementar manualmente o cache usando o módulo ``@cmmv/cache`` em um controlador gerado pelo ``@cmmv/http``. O cache pode ser adicionado em vários níveis, incluindo endpoints específicos ou manualmente após certas operações, como adicionar ou atualizar registros.

* **Controlador:** ``TaskController`` é responsável por lidar com requisições relacionadas a tarefas.
* **Decorador de Cache:** O decorador ``@Cache`` é usado para armazenar automaticamente em cache as respostas para endpoints GET. Neste caso, ``task:getAll`` e ``task:{id}`` são armazenados em cache por 300 segundos com compressão.
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

    // GET Todas as Tarefas com Cache
    @Get()
    @Cache("task:getAll", { ttl: 300, compress: true })
    async getAll(@Queries() queries: any, @Request() req): Promise<Task[]> {
        Telemetry.start('TaskController::GetAll', req.requestId);
        let result = await this.taskservice.getAll(queries, req);
        Telemetry.end('TaskController::GetAll', req.requestId);
        return result;
    }

    // GET Tarefa por ID com Cache
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

        // Armazena em cache o resultado da tarefa recém-criada
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

        // Atualiza o cache com a tarefa modificada
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

        // Remove a tarefa excluída do cache
        CacheService.del(`task:\${id}`);

        Telemetry.end('TaskController::Delete', req.requestId);
        return result;
    }
}
```

**Cache em Requisições GET:**

* **Cache Automático:** O decorador ``@Cache`` é aplicado aos métodos ``getAll`` e ``getById``. Os resultados em cache são armazenados e recuperados automaticamente com base no padrão de chave, reduzindo a carga no banco de dados e melhorando o desempenho.
* **Chave de Cache:** As chaves de cache são definidas como ``task:getAll`` para recuperar todas as tarefas e ``task:{id}`` para recuperar uma tarefa por ID.
* **TTL (Time-to-Live):** Os resultados em cache são válidos por 300 segundos.

**Cache Manual para ``POST``, ``PUT`` e ``DELETE``:**

* Para ``POST`` (criar uma nova tarefa) e ``PUT`` (atualizar uma tarefa), o método ``CacheService.set`` é usado para atualizar manualmente o cache com a tarefa nova ou modificada.
* Para ``DELETE`` (remover uma tarefa), o método ``CacheService.del`` é usado para remover a entrada correspondente do cache após a exclusão da tarefa.
* **Telemetria:** Registra e monitora a execução de cada método, útil para monitoramento de desempenho.

Essa configuração otimiza o tempo de resposta ao armazenar em cache dados acessados frequentemente, garantindo que as entradas de cache sejam atualizadas ou removidas quando os dados são modificados.

## Configurações do Contrato

O módulo ``@cmmv/cache`` fornece o decorador ``@Cache``, que pode ser aplicado a qualquer controlador, gateway ou métodos específicos. Este decorador é útil para armazenar automaticamente em cache as respostas de rotas específicas.

No exemplo a seguir, propriedades de cache são adicionadas a um contrato usando o decorador ``@Contract``. Isso habilita o cache para as rotas de controlador e gateway geradas automaticamente pelo transpiler.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    cache: {
        key: "task:",   // Prefixo da chave de cache
        ttl: 300,       // Tempo de vida (em segundos)
        compress: true  // Habilita compressão para dados armazenados
    }
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [
            {
                type: 'IsString',
                message: 'Rótulo inválido',
            },
            {
                type: 'IsNotEmpty',
                message: 'Rótulo inválido',
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
                message: 'Tipo de verificação inválido',
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
                message: 'Tipo de remoção inválido',
            },
        ],
    })
    removed: boolean;
}
```

## Propriedades de Cache

* **key:** Um prefixo para a chave de cache. No exemplo acima, é ``"task:"``. Para uma requisição ``getAll``, o resultado em cache será armazenado como ``task:getAll``.
* **ttl:** O tempo de vida dos dados em cache. Após 300 segundos, a entrada de cache expira.
* **compress:** Habilita a compressão dos dados antes de armazená-los no cache para reduzir o uso de memória.

Se você precisar de estratégias de cache mais avançadas ou integrações com armazenamentos personalizados, pode personalizar as configurações de cache no arquivo ``.cmmv.config.cjs`` ou estender as funcionalidades de cache existentes dentro do seu projeto. Por exemplo, você pode adicionar estratégias de cache personalizadas como cache em memória, cache em banco de dados ou até cache multinível.

## Controladores

Quando o módulo ``@cmmv/cache`` está presente no projeto, os controladores gerados automaticamente pelo ``@cmmv/http`` são modificados para incluir funcionalidades de cache. A configuração de cache é definida diretamente no contrato usando a propriedade ``cache``. Veja como o controlador é alterado:

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    cache: {
        key: "task:",
        ttl: 300,
        compress: true
    }
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [
            {
                type: 'IsString',
                message: 'Rótulo inválido',
            },
            {
                type: 'IsNotEmpty',
                message: 'O rótulo não pode estar vazio',
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
                message: 'O valor de verificação deve ser um booleano',
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
                message: 'O valor de remoção deve ser um booleano',
            },
        ],
    })
    removed: boolean;
}
```

Com base no contrato acima com a configuração de cache, o controlador gerado incluirá a lógica de cache usando o decorador ``@Cache`` do ``@cmmv/cache``. Aqui está um exemplo de como o controlador é gerado:

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

* **Decorador de Cache:** O decorador ``@Cache`` é adicionado a métodos como ``getAll`` e ``getById`` com base na configuração de cache fornecida no contrato. Este decorador armazena automaticamente os resultados em cache pelo tempo especificado (``ttl``) e aplica compressão (``compress``) se configurado.

* **``@Cache("task:getAll", { ttl: 300, compress: true })``:** Esta linha configura o cache para o método ``getAll``, onde a chave de cache é ``task:getAll``, com um TTL de 300 segundos, e comprime os dados em cache.

* **Gerenciamento Manual de Cache:** Nos métodos ``add``, ``update`` e ``delete``, o gerenciamento de cache é aplicado manualmente. Os métodos ``CacheService.set`` e ``CacheService.del`` são usados para atualizar ou excluir entradas de cache.

* **``CacheService.set("task:{id}", JSON.stringify(result), 300)``:** Define manualmente o cache para uma tarefa específica após adicioná-la ou atualizá-la.
* **``CacheService.del("task:{id}")``:** Exclui o cache de uma tarefa específica após sua remoção.
* **Chaves de Cache Dinâmicas:** As chaves de cache são dinâmicas, como mostrado nos métodos ``getById``, ``add``, ``update`` e ``delete``, onde o ID da tarefa faz parte da chave de cache.

O módulo ``@cmmv/cache`` aprimora os controladores gerados adicionando capacidades de cache automáticas e manuais com base na configuração de cache do contrato. O decorador ``@Cache`` é aplicado onde necessário, e o ``CacheService`` é usado para gerenciar entradas de cache para tarefas ou recursos específicos. Isso garante melhor desempenho ao reduzir consultas redundantes ao banco de dados ou chamadas de serviço.

## Gateways

Quando o módulo ``@cmmv/cache`` está presente no projeto, os gateways gerados pelo módulo ``@cmmv/ws`` são modificados para incluir funcionalidades de cache. A configuração de cache pode ser definida no contrato e impacta como as mensagens RPC são tratadas.

* **Decorador de Cache:** O decorador ``@Cache`` é aplicado aos manipuladores de mensagens no gateway, habilitando o cache automático das respostas. A chave de cache, TTL (tempo de vida) e opções de compressão são baseados na configuração do contrato.

* **Gerenciamento Manual de Cache:** As entradas de cache são atualizadas ou excluídas manualmente quando novos dados são adicionados ou modificados, garantindo que o cache permaneça consistente com os dados mais recentes.

```typescript
// Gerado automaticamente pelo CMMV

import { Rpc, Message, Data, Socket, RpcUtils } from "@cmmv/ws";
import { plainToClass } from 'class-transformer';
import { TaskEntity } from '../entities/task.entity';
import { Cache, CacheService } from "@cmmv/cache"; // Módulo de cache

import {
    AddTaskRequest,
    UpdateTaskRequest,
    DeleteTaskRequest
} from "../protos/task";

import { TaskService } from '../services/task.service';

@Rpc("task")
export class TaskGateway {
    constructor(private readonly taskservice: TaskService) {}

    @Message("GetAllTaskRequest")
    @Cache("task:getAll", { ttl: 300, compress: true }) // Decorador de cache
    async getAll(@Socket() socket) {
        const items = await this.taskservice.getAll();

        const response = await RpcUtils.pack(
            "task", "GetAllTaskResponse", items
        );

        socket.send(response);
    }

    @Message("AddTaskRequest")
    async add(@Data() data: AddTaskRequest, @Socket() socket) {
        const entity = plainToClass(TaskEntity, data.item);
        const result = await this.taskservice.add(entity);

        const response = await RpcUtils.pack(
            "task", "AddTaskResponse",
            { item: result, id: result.id }
        );

        CacheService.set(// Atualização manual do cache
            `task:\${result.id}`,
            JSON.stringify(result),
            300
        );

        socket.send(response);
    }

    @Message("UpdateTaskRequest")
    async update(@Data() data: UpdateTaskRequest, @Socket() socket) {
        const entity = plainToClass(TaskEntity, data.item);
        const result = await this.taskservice.update(data.id, entity);

        const response = await RpcUtils.pack(
            "task", "UpdateTaskResponse",
            { item: result, id: result.id }
        );

        CacheService.set(// Atualização manual do cache
            `task:\${result.id}`,
            JSON.stringify(result),
            300
        );

        socket.send(response);
    }

    @Message("DeleteTaskRequest")
    async delete(@Data() data: DeleteTaskRequest, @Socket() socket) {
        const result = (await this.taskservice.delete(data.id)).success;

        const response = await RpcUtils.pack(
            "task",
            `
            "DeleteTaskResponse",
            { success: result, id: data.id }
        );

        CacheService.del(`task:\${data.id}`); // Exclusão do cache

        socket.send(response);
    }
}
```
<br/>

* **Cache Automático:** O decorador ``@Cache`` é aplicado aos manipuladores de mensagens para armazenar respostas em cache com base na configuração do contrato.
* **Gerenciamento Manual de Cache:** As entradas de cache são atualizadas ou excluídas manualmente quando dados são adicionados, atualizados ou excluídos, garantindo consistência.
* **Melhoria de Desempenho:** Respostas em cache reduzem a necessidade de chamadas repetidas ao serviço, melhorando o desempenho das interações RPC.
