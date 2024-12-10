# Controllers

The ``@cmmv/http`` module automatically generates REST controllers with standard CRUD operations based on defined contracts. However, you can create custom controllers using a syntax similar to NestJS, with decorators for defining routes and methods. Unlike NestJS, CMMV does not use complex dependency injection. Instead, services like databases, queues, and cache must be implemented as singletons.

Here’s an example of a custom controller:

```typescript
import * as fs from 'fs';
import * as path from "path";

import { 
    Controller, Get, Param, 
    Response, ServiceRegistry 
} from '@cmmv/http';

import { DocsService } from './docs.service';

@Controller("docs")
export class DocsController {
    constructor(private docsService: DocsService){}

	@Get()
	async index(@Response() res) {		
		res.render("views/docs/index", {
			docs: await this.docsService.getDocsStrutucture(),
			services: ServiceRegistry.getServicesArr()
		});
	}

	@Get(":item")
	async getDoc(@Param("item") item: string, @Response() res) {
		const file = path.resolve("./docs/" + item + ".html");
		const data = await this.docsService.getDocsStrutucture(file);

		res.render("views/docs/index", {
			docs: data,
			services: ServiceRegistry.getServicesArr()
		});
	}
}
```

## @Controller(prefix: string)

Defines the controller and the route prefix for all its methods. The prefix parameter is optional, but it defines a base URL for the controller's routes.

```typescript
@Controller('tasks')
export class TaskController { ... }
```

## @Request() / @Req()

Binds the entire request object to the method parameter.

```typescript
@Get()
async getRequestData(@Request() req: any): Promise<any> {
    return req;
}
```

## @Response() / @Res()

Binds the entire response object to the method parameter, useful for handling custom responses.

```typescript
@Get()
async customResponse(@Response() res: any): Promise<void> {
    res.send('Custom Response');
}
```

## @Next()

Binds the next function in middleware to the method parameter, useful for middleware chaining.

```typescript
@Get()
async handleRequest(@Next() next: Function): Promise<void> {
    next();
}
```

## @Get(path?: string)

Maps an HTTP GET request to a specific method. The optional path argument can define the route, or it will default to the controller's base route.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Post(path?: string)

Handles HTTP POST requests. The @Body decorator can be used to access the request body and pass it to the method.

```typescript
@Post()
addTask(@Body() task: any) {
    return this.taskService.add(task);
}
```

## @Put(path?: string)

Maps an HTTP PUT request to update existing resources. Like @Post, it can accept data through the request body.

```typescript
@Put(':id')
updateTask(@Param('id') id: string, @Body() task: any) {
    return this.taskService.update(id, task);
}
```

## @Delete(path?: string)

Handles DELETE requests for removing resources by ID.

```typescript
@Delete(':id')
deleteTask(@Param('id') id: string) {
    return this.taskService.delete(id);
}
```

## @Param(param: string)

Used to extract parameters from the route. In the example, @Param('id') extracts the id from the request's route.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Body()

This decorator extracts the request body and makes it available in the method. It's commonly used with @Post and @Put for creating and updating data.

## @Query()

Extracts query parameters from the request.

```typescript
@Get()
getTasks(@Query('status') status: string) {
    return this.taskService.getByStatus(status);
}
```

## @Queries()
Binds all query parameters to the method parameter.

```typescript
@Get()
async getAll(@Queries() queries: any): Promise<Task[]> {
    return this.taskService.getAll(queries);
}
```

## @Header(headerName: string)

Binds a specific header value to the method parameter.

```typescript
@Get()
async checkHeader(@Header('Authorization') auth: string): Promise<boolean> {
    return this.authService.verifyToken(auth);
}
```

## @Headers()

Binds all headers to the method parameter.

```typescript
@Get()
async getHeaders(@Headers() headers: any): Promise<any> {
    return headers;
}
```

## @Session()

Extracts session data and binds it to the method parameter.

```typescript
@Get()
async getSessionData(@Session() session: any): Promise<any> {
    return session;
}
```

## @Ip()

Binds the client's IP address to the method parameter.

```typescript
@Get()
async getClientIp(@Ip() ip: string): Promise<string> {
    return ip;
}
```

## @HostParam()

Extracts the host information from the request.

```typescript
@Get()
async getHost(@HostParam() host: string): Promise<string> {
    return host;
}
```

# Getting up and running

To make the controller functional, it needs to be added to a module and called in the application. The process involves creating a module, registering the controller, and ensuring the module is used during application initialization.

Here is an example of registering a controller inside a module:

```typescript   
import { Module } from '@cmmv/core';
import { TaskController } from './controllers/task.controller';
    
export let TaskModule = new Module({
    controllers: [TaskController],
    ...
});
```

Once the module is created, you can import and load it into the main application configuration, ensuring the controller is properly initialized and accessible for handling requests.

The module will automatically handle routing and the controller logic when linked to the application setup. This modular design enables the clean separation of concerns and simplifies adding and managing controllers in the CMMV framework.