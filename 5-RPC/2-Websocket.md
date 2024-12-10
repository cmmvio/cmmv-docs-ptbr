# Websocket

The ``@cmmv/ws`` module is a key component responsible for handling WebSocket communication in the CMMV framework. It is crucial for the functioning of **Remote Procedure Calls (RPC)** in the system, allowing for seamless client-server interactions. This module automatically generates WebSocket gateways based on the contracts defined in your project, enabling binary RPC request and response handling.

* **Automatic Gateway Generation:** When the ``@cmmv/ws`` module is added, it automatically generates WebSocket gateways from the contracts defined in your project. These gateways serve as the communication endpoints for RPC requests and responses.

* **Binary Protocol:** The module communicates using a binary format for all RPC requests and responses. This ensures efficient data transfer and supports complex data structures. It is important to note that simple text-based communication is not supported by this module.

* **Contract-based Communication:** The communication follows the structure defined in your contracts, ensuring type-safe, well-structured data handling. The contracts are written using the ``@Contract`` decorator from the ``@cmmv/core`` module.

Below is an example of a contract that defines the structure for task-related RPC operations. The contract is annotated with the @Contract and @ContractField decorators to specify the fields and types.


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

The contract above will generate a WebSocket gateway, responsible for handling RPC operations like adding, updating, deleting, and retrieving tasks. Below is an example of a generated gateway:

```typescript
// Generated automatically by CMMV
    
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

The gateway interacts with services (e.g., TaskService) to process the data. These services perform business logic and interact with the database or other back-end components.

The ``@cmmv/ws`` module is a powerful tool for handling binary WebSocket communication in the CMMV framework. It automatically generates WebSocket gateways based on contracts, handling RPC requests and responses efficiently using binary data. This ensures secure, performant communication and seamless integration with the rest of the CMMV system.

By utilizing this module, developers can easily implement real-time functionality in their applications, leveraging the built-in RPC system for tasks such as data synchronization, real-time updates, and more.