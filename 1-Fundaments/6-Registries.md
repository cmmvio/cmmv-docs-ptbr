# Registros

Os registros no CMMV funcionam como um armazenamento centralizado para registros importantes que serão necessários posteriormente na aplicação. Eles são usados para gerenciar serviços, controladores, gateways, filas e mais.

Os registros são particularmente úteis em combinação com decoradores, que armazenam e processam metadados. Por exemplo, o **Registro HTTP** recebe definições de classe por meio de decoradores. Dentro dessas classes, funções decoradas com decoradores específicos de método atuam como manipuladores de requisições que precisam ser processados pelo adaptador HTTP. Essas funções manipuladoras podem ter decoradores de entrada que aparecem em uma ordem não predeterminada, exigindo que o registro as pré-processe para que possam ser chamadas corretamente.

Os registros desempenham um papel crucial em:
- **Armazenar e formatar metadados** que serão usados por adaptadores (por exemplo, HTTP, WebSocket).
- **Garantir a execução correta de funções** ao pré-processar manipuladores e seus parâmetros.
- **Manter o comportamento singleton**, pois os registros sempre funcionam de maneira estática.

## Por que usar Registros?

Essa abordagem elimina a necessidade de declarar manualmente dependências em cada módulo. Como os registros usam um **padrão de classe estática**, os serviços registrados podem ser acessados de qualquer lugar na aplicação enquanto preservam suas propriedades.

Por exemplo:
- Serviços e dependências podem ser instanciados **uma vez** e armazenados no registro.
- Essas instâncias podem ser recuperadas de qualquer parte da aplicação, como controladores, gateways e provedores.
- Registros mais complexos, como **Registros de Controladores**, podem pré-processar manipuladores e parâmetros, facilitando o processamento dinâmico dessas chamadas pelo framework.

## Exemplo de Registro de Serviço

Abaixo está uma implementação simples de um **Registro de Serviço**, que é responsável por registrar e gerenciar serviços.

```typescript
export class ServiceRegistry {
    private static services = new Map<any, { name: string }>();

    public static registerService(target: any, name: string) {
        this.services.set(target, { name });
    }

    public static getServices() {
        return Array.from(this.services.entries());
    }

    public static getServicesArr() {
        return Array.from(this.services.entries()).reduce(
            (acc, [cls, instance]) => {
                acc[instance.name] = new cls();
                return acc;
            },
            {},
        );
    }

    public static getService(name: string) {
        for (const [target, metadata] of this.services.entries()) {
            if (metadata.name === name) return target;
        }
        return undefined;
    }

    public static clear() {
        this.services.clear();
    }
}
```

Fonte: [https://github.com/cmmvio/cmmv/blob/main/packages/core/registries/service.registry.ts](https://github.com/cmmvio/cmmv/blob/main/packages/core/registries/service.registry.ts)

<br/>

* Registra serviços usando registerService(target, name).
* Recupera uma instância de serviço usando getService(name).
* Recupera todos os serviços registrados usando getServices() ou getServicesArr().
* Limpa todos os serviços registrados quando necessário.

## Registros Genéricos

Para simplificar ainda mais a criação de registros e decoradores personalizados, o CMMV inclui uma abstração genérica chamada GenericRegistry. Isso permite:

* A criação de decoradores de classe, método e parâmetro de maneira estruturada.
* Armazenar e recuperar metadados de forma eficiente sem exigir Symbols ou APIs de Reflexão.

```typescript
const META_OPTIONS = Symbol('controller_options');

export class GenericRegistry {
    public static controllers = new Map<
        any,
        {
            options?: any;
            handlers: any[];
        }
    >();

    public static registerController(target: any, options: any) {
        if (!this.controllers.has(target)) {
            this.controllers.set(target, {
                handlers: [],
                options,
            });
        } else {
            this.controllers.set(target, {
                ...this.controllers.get(target),
                options,
            });
        }
    }

    public static registerHandler(target: any, handlerName: string) {
        let controller = this.controllers.get(target.constructor);

        if (!controller) {
            const options =
                Reflect.getMetadata(META_OPTIONS, target.constructor) || {};
            this.registerController(target.constructor, options);
            controller = this.controllers.get(target.constructor);
        }

        if (controller) {
            const handler = controller.handlers.find(
                msg => msg.handlerName === handlerName,
            );

            if (!handler) controller.handlers.push({ handlerName, params: [] });
        }
    }

    public static registerParam(
        target: any,
        handlerName: string,
        paramType: string,
        index: number,
    ) {
        let controller = this.controllers.get(target.constructor);

        if (!controller) {
            const options =
                Reflect.getMetadata(META_OPTIONS, target.constructor) || {};
            this.registerController(target.constructor, options);
            controller = this.controllers.get(target.constructor);
        }

        if (controller) {
            let handler = controller.handlers.find(
                msg => msg.handlerName === handlerName,
            );

            if (!handler) {
                handler = { handlerName, params: [] };
                controller.handlers.push(handler);
            }

            handler.params = handler.params || [];
            handler.params.push({ paramType, index });
        }
    }

    public static getControllers() {
        return Array.from(this.controllers.entries());
    }

    public static getHandlers(target: any): any[] {
        const controller = this.controllers.get(target.constructor);
        return controller ? controller.handlers : [];
    }

    public static getParams(target: any, handlerName: string): any[] {
        const queues = this.controllers.get(target.constructor);

        if (!queues) return [];

        const handler = queues.handlers.find(
            handler => handler.handlerName === handlerName,
        );

        return handler ? handler.params : [];
    }

    public static clear() {
        this.controllers = new Map<
            any,
            {
                options?: any;
                handlers: any[];
            }
        >();
    }
}
```

Fonte: [https://github.com/cmmvio/cmmv/blob/main/packages/core/registries/generic.registry.ts](https://github.com/cmmvio/cmmv/blob/main/packages/core/registries/generic.registry.ts)

## Usando GenericRegistry

### Decorador de Controlador
<br/>

```typescript
function Controller(options?: any) {
    return function (target: any) {
        GenericRegistry.registerController(target, options);
    };
}
```

### Decorador de Rota
<br/>

```typescript
function Route(route: string, method: string = 'GET') {
    return function (target: any, handlerName: string) {
        GenericRegistry.registerHandler(target, handlerName);
    };
}
```

### Decorador de Parâmetro
<br/>

```typescript
function Param(type: string) {
    return function (target: any, handlerName: string, index: number) {
        GenericRegistry.registerParam(target, handlerName, type, index);
    };
}
```

### Exemplo de Uso
<br/>

```typescript
@Controller({ prefix: '/users' })
class UserController {
    @Route('/get/:id', 'GET')
    getUser(@Param('id') id: number) {
        return { id, name: "John Doe" };
    }
}
```

### Recuperando Registrados
<br/>

```typescript
// Obter todos os controladores
const controllers = GenericRegistry.getControllers();
console.log(controllers);

// Obter manipuladores de um controlador específico
const handlers = GenericRegistry.getHandlers(UserController);
console.log(handlers);
```

### Como a aplicação lê os controladores?
A aplicação, por meio de adaptadores HTTP, lê os controladores registrados no Registro, instancia as classes com base nos provedores definidos no construtor e processa os manipuladores com seus respectivos parâmetros e decoradores. Isso permite a vinculação dinâmica de rotas e a execução correta de requisições.

Abaixo está a implementação do adaptador HTTP padrão:

```typescript
const controllers = ControllerRegistry.getControllers();

controllers.forEach(([controllerClass, metadata]) => {
    const paramTypes =
        Reflect.getMetadata('design:paramtypes', controllerClass) || [];
    const instances = paramTypes.map((paramType: any) =>
        this.application.providersMap.get(paramType.name),
    );

    const instance = new controllerClass(...instances);
    const prefix = metadata.prefix;
    const routes = metadata.routes;

    routes.forEach(route => {
        let fullPath = `\${prefix}\${route.path ? '/' + route.path : ''}`;
        fullPath = fullPath.replace(/\/\//g, '/');
        const method = route.method.toLowerCase();

        if (this.instance[method] && fullPath !== '/*') {
            const handler = async (req: any, res: any, next: any) => {
                ...
            };
        }
    });
});
```

Fonte: [https://github.com/cmmvio/cmmv/blob/main/packages/http/default.adapter.ts#L236](https://github.com/cmmvio/cmmv/blob/main/packages/http/default.adapter.ts#L236)

### Parâmetros do manipulador

No caso de parâmetros de manipuladores, sua recuperação varia dependendo da implementação. No entanto, para controladores, esses parâmetros são extraídos diretamente do objeto req fornecido pelo servidor HTTP.

Abaixo está a implementação:

```typescript
private buildRouteArgs(req: any, res: any, next: any, params: any[]) {
    const args: any[] = [];

    params?.forEach(param => {
        const [paramType, paramName] = param.paramType.split(':');
        switch (paramType) {
            case 'body':
                args[param.index] = req.body;
                break;
            case 'param':
                args[param.index] = req.params[paramName];
                break;
            case 'query':
                args[param.index] = req.query[paramName];
                break;
            case 'queries':
                args[param.index] = req.query;
                break;
            case 'header':
                args[param.index] = req.headers[paramName.toLowerCase()];
                break;
            case 'headers':
                args[param.index] = req.headers;
                break;
            case 'request':
                args[param.index] = req;
                break;
            case 'response':
                args[param.index] = res;
                break;
            case 'next':
                args[param.index] = next;
                break;
            case 'session':
                args[param.index] = req.session;
                break;
            case 'user':
                args[param.index] = req.user;
                break;
            case 'ip':
                args[param.index] = req.ip;
                break;
            case 'hosts':
                args[param.index] = req.hosts;
                break;
            default:
                args[param.index] = undefined;
                break;
        }
    });

    return args;
}
```

Fonte: [https://github.com/cmmvio/cmmv/blob/main/packages/http/default.adapter.ts#L478](https://github.com/cmmvio/cmmv/blob/main/packages/http/default.adapter.ts#L478)

## Registros vs Reflexão

Usar registros em vez de reflexão e símbolos oferece vantagens significativas em termos de desempenho, eficiência e manutenibilidade. Diferentemente da reflexão, que recupera metadados dinamicamente toda vez que é acessada, os registros armazenam e processam metadados apenas uma vez durante a inicialização da aplicação. Isso elimina buscas redundantes e garante uma execução mais rápida ao evitar a resolução de metadados em tempo de execução. Os registros pré-processam e estruturam os dados para uso eficiente, tornando os manipuladores de controladores e parâmetros prontamente disponíveis para execução imediata sem processamento adicional. Isso elimina cálculos desnecessários em tempo de execução, permite a recuperação eficiente dos metadados dos decoradores e garante dados estruturados e ordenados para a execução de funções.

Abordagens baseadas em reflexão exigem buscas repetidas de metadados em tempo de execução, não fornecem armazenamento estruturado e dependem do comportamento de reflexão do TypeScript, que pode ter limitações em alguns ambientes. Os registros, por outro lado, armazenam dados estruturados estaticamente, permitem acesso direto baseado em chaves em vez de buscas repetidas e permanecem independentes de framework ao não depender do sistema de reflexão do TypeScript. Ao gerenciar metadados centralmente, os registros fornecem melhor consistência entre diferentes módulos, eliminando a necessidade de chamadas dispersas de recuperação de metadados e garantindo uma estrutura previsível para os decoradores.

Além disso, os registros integram-se perfeitamente com a injeção de dependência, facilitando a recuperação e injeção de dependências em toda a aplicação. Essa abordagem simplifica o armazenamento e a recuperação de metadados, oferecendo melhor desempenho, acesso estruturado e manutenibilidade em comparação com métodos baseados em reflexão. Ao carregar os metadados uma vez e estruturá-los eficientemente, a aplicação pode acessar e utilizar dinamicamente os metadados dos decoradores sem penalidades de desempenho, tornando os registros uma solução mais eficiente e escalável.
