# Contracts 

CMMV's contract system allows defining structured models that are used to auto-generate APIs, RPCs, and WebSocket routes. Contracts are defined using decorators applied to classes and their fields.

Contracts are defined using the `@Contract` and `@ContractField` decorators. These decorators specify the contract name, file path for protocol buffers, and the type of the contract fields.

```typescript
import { AbstractContract, Contract, ContractField } from "@cmmv/core";

@Contract({
    controllerName: "Task",
    protoPath: "src/protos/task.proto",
    protoPackage: "task"
})
export class TasksContract extends AbstractContract {
    @ContractField({ 
        protoType: 'string', 
        unique: true 
    })
    label: string;

    @ContractField({ 
        protoType: 'bool', 
        defaultValue: false 
    })
    checked: boolean;

    @ContractField({ 
        protoType: 'bool', 
        defaultValue: false 
    })
    removed: boolean;
}
```

## @Contract
Defines a contract class with the following options:
- **controllerName**: Specifies the contract name.
- **controllerCustomPath**: defines a custom path for the Rest controller.
- **protoPath**: Specifies the path to the protocol buffer file.
- **protoPackage**: defines the namespace of the contract generated in the protocol buffer.
- **directMessage**: defines whether RPC calls are direct or whether a crud structure should be created.
- **generateController**: defines whether or not transpilers should automatically generate drivers for this contract.
- **generateController**: defines access to data that requires authentication.

## @ContractField
Defines a contract field with options such as:
- **protoType**: Field type (`string`, `number`, etc.).
- **protoRepeated**: defines whether the field is a list.
- **defaultValue**: sets a default value when creating the record.
- **unique**: Ensures uniqueness.
- **index**: defines that the field in question is an index.

## Supported Field Types
CMMV supports a variety of field types:
- **Basic Types**: `string`, `boolean`, `int`, `float`, `double`, `bytes`, `uuid`
- **Numeric Types**: `int32`, `int64`, `uint32`, `uint64`, `sint32`, `sint64`, `fixed32`, `fixed64`, `sfixed32`, `sfixed64`
- **Advanced Types**: `json`, `jsonb`, `simpleArray`, `simpleJson`, `any`
- **Date and Time Types**: `date`, `time`, `timestamp`

When the application starts, the system automatically generates controllers, services, and generic entities based on the contract definitions. This process simplifies development by ensuring that common components like CRUD operations are pre-built and ready to use. Each entity and service is created dynamically to match the contract specifications, allowing the developer to focus on custom logic without having to manually define the basic structures. This auto-generation enhances efficiency and consistency throughout the application.

## HTTP Module

To start a basic REST application using the ``@cmmv/http`` and ``@cmmv/view`` modules, follow these steps:

```
$ pnpm add @cmmv/http @cmmv/view
```

Set up the application:

```typescript
import { Application } from "@cmmv/core";
import { ExpressAdapter, ExpressModule } from "@cmmv/http";
import { ViewModule } from "@cmmv/view";

Application.create({
    httpAdapter: ExpressAdapter,
    modules: [ 
        ExpressModule,
        ViewModule
    ]
});
```

To correctly implement a basic CMMV application using either the Express or Fastify adapters, you can use the following structure in your ``src/index.ts`` file. The system supports both Express and Fastify for handling HTTP requests, and the correct adapter is selected in the configuration.

By default, when you start the application, it will be hosted at ``http://localhost:3000``. This can be easily modified by configuring the port and bind address in the application's configuration file. The configuration settings allow you to change the host, port, and other related properties for the HTTP server. These options will be covered in more detail later when we show the configuration file setup, providing flexibility to adjust your server's runtime environment.

## Model

The system will automatically generate the model for your entity in the following format:

```typescript
// Generated automatically by CMMV

export interface Task {
    id?: any;
    label: string;
    checked: boolean;
    removed: boolean;
}
```

This Task interface defines the structure of the model, including optional and required fields. The id is optional, and fields such as label, checked, and removed are mandatory with defined types. You can further customize or extend this model based on your application’s needs.

This base interface, such as the Task interface, will be utilized by other modules, including the repository module, to define entities in the database. By sharing the same interface structure across modules, it ensures consistency and type safety throughout the application. For example, the repository module will use this Task interface to map and handle database operations, making it easier to maintain and scale your application while ensuring that the data model aligns across different layers of the system.

## Controller

The example contract provided above will automatically generate a controller at the path ``/src/controllers/task.controller.ts``. This controller will be structured as follows:

```typescript
// Generated automatically by CMMV
    
import { Telemetry } from "@cmmv/core";

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
    async getAll(@Queries() queries: any, @Request() req): Promise<Task[]> {
        Telemetry.start('TaskController::GetAll', req.requestId);
        let result = await this.taskservice.getAll(queries, req);
        Telemetry.end('TaskController::GetAll', req.requestId);
        return result;
    }

    @Get(':id')
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
        Telemetry.end('TaskController::Add', req.requestId);
        return result;
    }

    @Put(':id')
    async update(
        @Param('id') id: string, 
        @Body() item: Task, 
        @Request() req
    ): Promise<Task> {
        Telemetry.start('TaskController::Update', req.requestId);
        let result = await this.taskservice.update(id, item, req);
        Telemetry.end('TaskController::Update', req.requestId);
        return result;
    }

    @Delete(':id')
    async delete(
        @Param('id') id: string, 
        @Request() req
    ): Promise<{ success: boolean, affected: number }> {
        Telemetry.start('TaskController::Delete', req.requestId);
        let result = await this.taskservice.delete(id, req);
        Telemetry.end('TaskController::Delete', req.requestId);
        return result;
    }
}
```

You can create your own custom controllers by simply informing the application of their existence during the creation process or by including them via modules. This allows for greater flexibility and customization in your application's architecture. When you define custom controllers, you can register them within the application module during the application setup phase, ensuring that they integrate smoothly with the core framework, such as HTTP routing or WebSocket handling, while adhering to your project's specific requirements.

## Services

Just like the controller, the service layer is also automatically generated by the system. In the absence of persistence modules such as the repository, a placeholder service is created, which temporarily stores records in memory while the application is online. This allows basic operations like adding, updating, and deleting data without a persistent store. Once a repository module is introduced, it overrides the default service, enabling direct interaction with a database, allowing data to be saved, queried, and managed efficiently from persistent storage.

Here is an example of a service created using the repository module. This service interacts directly with the database using the repository pattern:

```typescript
// Generated automatically by CMMV
    
import { Telemetry } from "@cmmv/core";
import { AbstractService, Service } from '@cmmv/http';
import { Repository } from '@cmmv/repository';
import { TaskEntity } from '../entities/task.entity';

@Service("task")
export class TaskService extends AbstractService {
    public override name = "task";

    async getAll(queries?: any, req?: any): Promise<TaskEntity[]> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::GetAll', req?.requestId);
        let result = await repository.find();
        Telemetry.end('TaskService::GetAll', req?.requestId);
        return result;
    }

    async getById(id: string, req?: any): Promise<TaskEntity> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::GetById', req?.requestId);
        const item = await repository.findOneBy({ id });
        Telemetry.end('TaskService::GetById', req?.requestId);

        if (!item) 
            throw new Error('Item not found');
        
        return item;
    }

    async add(item: Partial<TaskEntity>, req?: any): Promise<TaskEntity> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::Add', req?.requestId);
        const result = await repository.save(item);
        Telemetry.end('TaskService::Add', req?.requestId);
        return result;
    }

    async update(
        id: string, item: Partial<TaskEntity>, req?: any
    ): Promise<TaskEntity> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::Update', req?.requestId);
        await repository.update(id, item);
        let result = await repository.findOneBy({ id });
        Telemetry.end('TaskService::Update', req?.requestId);
        return result;
    }

    async delete(
        id: string, req?: any
    ): Promise<{ success: boolean, affected: number }> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::Delete', req?.requestId);
        const result = await repository.delete(id);
        Telemetry.end('TaskService::Delete', req?.requestId);
        return { success: result.affected > 0, affected: result.affected };
    }
}
```

## More

The installation of additional modules, such as RPC, caching, and authentication, will extend the functionality of your application. Each module introduces new capabilities, like enabling remote procedure calls, enhancing security through authentication, or improving performance with caching. For more details, specific documentation for each module is available in the sidebar, where you'll find comprehensive guides on how to integrate and implement these modules effectively within your application.