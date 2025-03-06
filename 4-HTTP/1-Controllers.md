# Controladores

O módulo ``@cmmv/http`` gera automaticamente controladores REST com operações CRUD padrão com base nos contratos definidos. No entanto, você pode criar controladores personalizados usando uma sintaxe semelhante ao NestJS, com decoradores para definir rotas e métodos. Diferentemente do NestJS, o CMMV não utiliza injeção de dependências complexa. Em vez disso, serviços como bancos de dados, filas e cache devem ser implementados como singletons.

Aqui está um exemplo de um controlador personalizado:

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

Define o controlador e o prefixo de rota para todos os seus métodos. O parâmetro `prefix` é opcional, mas define uma URL base para as rotas do controlador.

```typescript
@Controller('tasks')
export class TaskController { ... }
```

## @Request() / @Req()

Vincula o objeto de solicitação inteiro ao parâmetro do método.

```typescript
@Get()
async getRequestData(@Request() req: any): Promise<any> {
    return req;
}
```

## @Response() / @Res()

Vincula o objeto de resposta inteiro ao parâmetro do método, útil para manipular respostas personalizadas.

```typescript
@Get()
async customResponse(@Response() res: any): Promise<void> {
    res.send('Resposta Personalizada');
}
```

## @Next()

Vincula a função `next` do middleware ao parâmetro do método, útil para encadeamento de middlewares.

```typescript
@Get()
async handleRequest(@Next() next: Function): Promise<void> {
    next();
}
```

## @Get(path?: string)

Mapeia uma solicitação HTTP GET para um método específico. O argumento opcional `path` pode definir a rota ou será padrão para a rota base do controlador.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Post(path?: string)

Lida com solicitações HTTP POST. O decorador `@Body` pode ser usado para acessar o corpo da solicitação e passá-lo para o método.

```typescript
@Post()
addTask(@Body() task: any) {
    return this.taskService.add(task);
}
```

## @Put(path?: string)

Mapeia uma solicitação HTTP PUT para atualizar recursos existentes. Assim como `@Post`, pode aceitar dados através do corpo da solicitação.

```typescript
@Put(':id')
updateTask(@Param('id') id: string, @Body() task: any) {
    return this.taskService.update(id, task);
}
```

## @Delete(path?: string)

Lida com solicitações DELETE para remover recursos por ID.

```typescript
@Delete(':id')
deleteTask(@Param('id') id: string) {
    return this.taskService.delete(id);
}
```

## @Param(param: string)

Usado para extrair parâmetros da rota. No exemplo, `@Param('id')` extrai o ID da rota da solicitação.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Body()

Este decorador extrai o corpo da solicitação e o torna disponível no método. É comumente usado com `@Post` e `@Put` para criar e atualizar dados.

## @Query()

Extrai parâmetros de consulta da solicitação.

```typescript
@Get()
getTasks(@Query('status') status: string) {
    return this.taskService.getByStatus(status);
}
```

## @Queries()

Vincula todos os parâmetros de consulta ao parâmetro do método.

```typescript
@Get()
async getAll(@Queries() queries: any): Promise<Task[]> {
    return this.taskService.getAll(queries);
}
```

## @Header(headerName: string)

Vincula um valor de cabeçalho específico ao parâmetro do método.

```typescript
@Get()
async checkHeader(@Header('Authorization') auth: string): Promise<boolean> {
    return this.authService.verifyToken(auth);
}
```

## @Headers()

Vincula todos os cabeçalhos ao parâmetro do método.

```typescript
@Get()
async getHeaders(@Headers() headers: any): Promise<any> {
    return headers;
}
```

## @Session()

Extrai os dados da sessão e os vincula ao parâmetro do método.

```typescript
@Get()
async getSessionData(@Session() session: any): Promise<any> {
    return session;
}
```

## @Ip()

Vincula o endereço IP do cliente ao parâmetro do método.

```typescript
@Get()
async getClientIp(@Ip() ip: string): Promise<string> {
    return ip;
}
```

## @HostParam()

Extrai as informações do host da solicitação.

```typescript
@Get()
async getHost(@HostParam() host: string): Promise<string> {
    return host;
}
```

## Inicializando

Para que o controlador seja funcional, ele precisa ser adicionado a um módulo e chamado na aplicação. O processo envolve criar um módulo, registrar o controlador e garantir que o módulo seja usado durante a inicialização da aplicação.

Aqui está um exemplo de registro de um controlador dentro de um módulo:

```typescript
import { Module } from '@cmmv/core';
import { TaskController } from './controllers/task.controller';

export let TaskModule = new Module({
    controllers: [TaskController],
    ...
});
```

Depois que o módulo é criado, você pode importá-lo e carregá-lo na configuração principal da aplicação, garantindo que o controlador seja inicializado corretamente e acessível para lidar com as solicitações.

O módulo gerenciará automaticamente o roteamento e a lógica do controlador quando vinculado à configuração da aplicação. Esse design modular permite uma separação clara de responsabilidades e simplifica a adição e gerenciamento de controladores no framework CMMV.
