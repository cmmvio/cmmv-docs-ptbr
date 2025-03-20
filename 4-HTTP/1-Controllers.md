# Controladores

O módulo ``@cmmv/http`` gera automaticamente controladores REST com operações CRUD padrão baseadas em contratos definidos. No entanto, você pode criar controladores personalizados usando uma sintaxe semelhante ao NestJS, com decoradores para definir rotas e métodos. Diferentemente do NestJS, o CMMV não utiliza injeção de dependências complexa. Em vez disso, serviços como bancos de dados, filas e cache devem ser implementados como singletons.

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

Define o controlador e o prefixo de rota para todos os seus métodos. O parâmetro ``prefix`` é opcional, mas define uma URL base para as rotas do controlador.

```typescript
@Controller('tasks')
export class TaskController { ... }
```

## @Request() / @Req()

Vincula o objeto completo da requisição ao parâmetro do método.

```typescript
@Get()
async getRequestData(@Request() req: any): Promise<any> {
    return req;
}
```

## @Response() / @Res()

Vincula o objeto completo da resposta ao parâmetro do método, útil para lidar com respostas personalizadas.

```typescript
@Get()
async customResponse(@Response() res: any): Promise<void> {
    res.send('Resposta Personalizada');
}
```

## @Next()

Vincula a função ``next`` no middleware ao parâmetro do método, útil para encadeamento de middlewares.

```typescript
@Get()
async handleRequest(@Next() next: Function): Promise<void> {
    next();
}
```

## @Get(path?: string)

Mapeia uma requisição HTTP GET para um método específico. O argumento opcional ``path`` pode definir a rota, ou ela será padrão para a rota base do controlador.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Post(path?: string)

Lida com requisições HTTP POST. O decorador ``@Body`` pode ser usado para acessar o corpo da requisição e passá-lo ao método.

```typescript
@Post()
addTask(@Body() task: any) {
    return this.taskService.add(task);
}
```

## @Put(path?: string)

Mapeia uma requisição HTTP PUT para atualizar recursos existentes. Assim como o ``@Post``, pode aceitar dados pelo corpo da requisição.

```typescript
@Put(':id')
updateTask(@Param('id') id: string, @Body() task: any) {
    return this.taskService.update(id, task);
}
```

## @Delete(path?: string)

Lida com requisições DELETE para remover recursos por ID.

```typescript
@Delete(':id')
deleteTask(@Param('id') id: string) {
    return this.taskService.delete(id);
}
```

## @Param(param: string)

Usado para extrair parâmetros da rota. No exemplo, ``@Param('id')`` extrai o ``id`` da rota da requisição.

```typescript
@Get(':id')
getTaskById(@Param('id') id: string) {
    return this.taskService.getById(id);
}
```

## @Body()

Este decorador extrai o corpo da requisição e o disponibiliza no método. É comumente usado com ``@Post`` e ``@Put`` para criar e atualizar dados.

## @Query()

Extrai parâmetros de consulta da requisição.

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

Vincula o valor de um cabeçalho específico ao parâmetro do método.

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

Extrai dados da sessão e os vincula ao parâmetro do método.

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

Extrai as informações do host da requisição.

```typescript
@Get()
async getHost(@HostParam() host: string): Promise<string> {
    return host;
}
```

## Configuração e Inicialização

Para tornar o controlador funcional, ele precisa ser adicionado a um módulo e chamado na aplicação. O processo envolve criar um módulo, registrar o controlador e garantir que o módulo seja usado durante a inicialização da aplicação.

Aqui está um exemplo de registro de um controlador dentro de um módulo:

```typescript
import { Module } from '@cmmv/core';
import { TaskController } from './controllers/task.controller';

export let TaskModule = new Module({
    controllers: [TaskController],
    ...
});
```

Uma vez que o módulo é criado, você pode importá-lo e carregá-lo na configuração principal da aplicação, garantindo que o controlador seja devidamente inicializado e acessível para lidar com requisições.

O módulo gerenciará automaticamente o roteamento e a lógica do controlador quando vinculado à configuração da aplicação. Esse design modular permite uma separação clara de responsabilidades e simplifica a adição e gerenciamento de controladores no framework CMMV.
