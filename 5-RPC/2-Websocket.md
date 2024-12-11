# WebSocket

O módulo `@cmmv/ws` é um componente-chave responsável por gerenciar a comunicação WebSocket no framework CMMV. Ele é crucial para o funcionamento de **Remote Procedure Calls (RPC)** no sistema, permitindo interações cliente-servidor sem interrupções. Este módulo gera automaticamente gateways WebSocket com base nos contratos definidos no seu projeto, habilitando o manuseio eficiente de solicitações e respostas RPC binárias.

* **Geração Automática de Gateways:** Ao adicionar o módulo `@cmmv/ws`, ele gera automaticamente gateways WebSocket a partir dos contratos definidos no seu projeto. Esses gateways atuam como endpoints de comunicação para solicitações e respostas RPC.

* **Protocolo Binário:** O módulo utiliza um formato binário para todas as solicitações e respostas RPC. Isso garante transferência de dados eficiente e suporte a estruturas de dados complexas. É importante observar que comunicação baseada em texto simples não é suportada por este módulo.

* **Comunicação Baseada em Contratos:** A comunicação segue a estrutura definida nos seus contratos, garantindo tratamento de dados bem estruturado e com tipagem segura. Os contratos são escritos utilizando o decorador `@Contract` do módulo `@cmmv/core`.

Abaixo está um exemplo de contrato que define a estrutura para operações RPC relacionadas a tarefas. O contrato é anotado com os decoradores `@Contract` e `@ContractField` para especificar os campos e tipos.

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

O contrato acima gerará um gateway WebSocket responsável por gerenciar operações RPC como adicionar, atualizar, deletar e recuperar tarefas. Abaixo está um exemplo de gateway gerado:

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
            // Handle error
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
            // Handle error
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
            // Handle error
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
            // Handle error
        }
    }
}
```

O gateway interage com serviços (ex.: `TaskService`) para processar os dados. Esses serviços realizam a lógica de negócio e interagem com o banco de dados ou outros componentes do backend.

O módulo `@cmmv/ws` é uma ferramenta poderosa para gerenciar comunicação WebSocket binária no framework CMMV. Ele gera automaticamente gateways WebSocket com base nos contratos, lidando com solicitações e respostas RPC de forma eficiente usando dados binários. Isso garante uma comunicação segura, performática e integração fluida com o restante do sistema CMMV.

Ao utilizar este módulo, os desenvolvedores podem facilmente implementar funcionalidades em tempo real em suas aplicações, aproveitando o sistema RPC embutido para tarefas como sincronização de dados, atualizações em tempo real e muito mais.
