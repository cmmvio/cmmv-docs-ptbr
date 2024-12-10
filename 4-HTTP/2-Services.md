# Services

In ``@cmmv/core``, services play a crucial role in managing business logic, serving as a bridge between the data repositories, cache, and external communication layers like HTTP or RPC. Services are responsible for handling data transformations and ensuring that external communication is model-agnostic, meaning the service works with models and automatically converts them as necessary.

To create a service, use the ``@Service`` decorator. This decorator allows the service to be recognized by the framework and utilized throughout the application.

```typescript
@Service('task')
export class TaskService extends AbstractService {
    public override name = 'task';
    // Service methods and logic
}
```

## How Services Work

* **Input and Output Models:** Services use models (e.g., Task, ITask) to define their input and output. These models are automatically transformed when interacting with repositories or external communication layers.
* **Data Handling:** Services interface with the Repository to handle data persistence and retrieval. They are also responsible for validation, caching, and error handling.
* **Telemetry:** The service includes telemetry for performance tracking and logging, helping monitor execution times and request identifiers.

## Using Services

In ``@cmmv``, since the system doesn't implement traditional dependency injection, you can easily register services as providers in any module and access them via the constructor of a class. This simplifies the service usage across controllers, gateways, or other components.

```typescript
import { TaskService } from '../services/task.service';
import { Task } from '../models/task.model';
import { Cache, CacheService } from "@cmmv/cache";

@Controller('task')
export class TaskController {
    constructor(private readonly taskservice: TaskService) {}

    @Get()
    @Cache("task:getAll", { ttl: 300, compress: true })
    async getAll(@Queries() queries: any, @Request() req): Promise<Task[]> {
        let result = await this.taskservice.getAll(queries, req);
        return result;
    }
}
```

## Agnostic 

In the ``@cmmv`` framework, services are agnostic to the type of controller that invokes them, ensuring they can be used across HTTP controllers, RPC gateways, or other components. Services are automatically generated based on contract configurations and can be extended with the ``@cmmv/repository`` module to support database entities or ``@cmmv/cache`` for managing and retrieving cached data. While services handle business logic, authentication directives (``@cmmv/auth``) are applied at the controller or gateway level to ensure secure access. This modular approach allows for flexible and scalable service management.

## Singleton

Global access services in ``@cmmv`` should be implemented as singletons to ensure consistent behavior and efficient resource usage across the application. Singletons prevent the creation of multiple instances of a service, centralizing operations such as caching and database management. To learn more about the implementation and benefits of singletons in ``@cmmv``, please refer to the documentation available at [CMMV Singleton Documentation](https://cmmv.io/docs/overview/singleton). This documentation provides detailed guidance on how to implement and manage singleton services effectively.