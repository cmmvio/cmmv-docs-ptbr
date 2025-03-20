# WebSocket

O módulo ``@cmmv/ws`` é um componente essencial responsável por gerenciar a comunicação WebSocket no framework CMMV. Ele é crucial para o funcionamento das **Chamadas de Procedimento Remoto (RPC)** no sistema, permitindo interações contínuas entre cliente e servidor. Este módulo gera automaticamente gateways WebSocket com base nos contratos definidos no seu projeto, possibilitando o manuseio de requisições e respostas RPC em formato binário.

* **Geração Automática de Gateway:** Quando o módulo ``@cmmv/ws`` é adicionado, ele gera automaticamente gateways WebSocket a partir dos contratos definidos no seu projeto. Esses gateways funcionam como pontos de extremidade de comunicação para requisições e respostas RPC.

* **Protocolo Binário:** O módulo utiliza um formato binário para todas as requisições e respostas RPC. Isso garante uma transferência de dados eficiente e suporta estruturas de dados complexas. É importante observar que a comunicação simples baseada em texto não é suportada por este módulo.

* **Comunicação Baseada em Contratos:** A comunicação segue a estrutura definida nos seus contratos, garantindo um manuseio de dados seguro e bem estruturado. Os contratos são escritos usando o decorador ``@Contract`` do módulo ``@cmmv/core``.

Abaixo está um exemplo de contrato que define a estrutura para operações RPC relacionadas a tarefas. O contrato é anotado com os decoradores ``@Contract`` e ``@ContractField`` para especificar os campos e tipos.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
    })
    removed: boolean;
}
```

O contrato acima gerará um gateway WebSocket, responsável por lidar com operações RPC como adicionar, atualizar, excluir e recuperar tarefas. Abaixo está um exemplo de um gateway gerado:

```typescript
// Gerado automaticamente pelo CMMV

import { Rpc, Message, Data, Socket, RpcUtils } from "@cmmv/ws";
import { plainToClass } from 'class-transformer';
import { TaskEntity } from '../entities/task.entity';

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
    async getAll(@Socket() socket) {
        try {
            const items = await this.taskservice.getAll();
            const response = await RpcUtils.pack(
                "task", "GetAllTaskResponse", items
            );

            if (response)
                socket.send(response);
        } catch (e) {
            // Lidar com erro
        }
    }

    @Message("AddTaskRequest")
    async add(@Data() data: AddTaskRequest, @Socket() socket) {
        try {
            const entity = plainToClass(TaskEntity, data.item);
            const result = await this.taskservice.add(entity);
            const response = await RpcUtils.pack(
                "task", "AddTaskResponse", { item: result, id: result.id }
            );

            if (response)
                socket.send(response);
        } catch (e) {
            // Lidar com erro
        }
    }

    @Message("UpdateTaskRequest")
    async update(@Data() data: UpdateTaskRequest, @Socket() socket) {
        try {
            const entity = plainToClass(TaskEntity, data.item);
            const result = await this.taskservice.update(data.id, entity);
            const response = await RpcUtils.pack(
                "task", "UpdateTaskResponse", { item: result, id: result.id }
            );

            if (response)
                socket.send(response);
        } catch (e) {
            // Lidar com erro
        }
    }

    @Message("DeleteTaskRequest")
    async delete(@Data() data: DeleteTaskRequest, @Socket() socket) {
        try {
            const result = (await this.taskservice.delete(data.id)).success;
            const response = await RpcUtils.pack(
                "task", "DeleteTaskResponse", { success: result, id: data.id }
            );

            if (response)
                socket.send(response);
        } catch (e) {
            // Lidar com erro
        }
    }
}
```

O gateway interage com serviços (por exemplo, ``TaskService``) para processar os dados. Esses serviços executam a lógica de negócios e interagem com o banco de dados ou outros componentes do backend.

O módulo ``@cmmv/ws`` é uma ferramenta poderosa para gerenciar a comunicação WebSocket binária no framework CMMV. Ele gera automaticamente gateways WebSocket com base em contratos, lidando com requisições e respostas RPC de forma eficiente usando dados binários. Isso garante uma comunicação segura, performática e uma integração perfeita com o restante do sistema CMMV.

Ao utilizar este módulo, os desenvolvedores podem facilmente implementar funcionalidades em tempo real em suas aplicações, aproveitando o sistema RPC integrado para tarefas como sincronização de dados, atualizações em tempo real e muito mais.
