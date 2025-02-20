# Registradores (Registries)

Os registradores no CMMV servem como armazenamento centralizado para registros importantes que serão utilizados posteriormente na aplicação. Eles são usados para gerenciar serviços, controladores, gateways, filas e mais.

Os registradores são particularmente úteis quando combinados com decoradores, que armazenam e processam metadados. Por exemplo, o **Registrador HTTP** recebe definições de classe via decoradores. Dentro dessas classes, funções decoradas com métodos específicos atuam como manipuladores de requisição que precisam ser processados pelo adaptador HTTP. Essas funções podem ter decoradores de entrada em uma ordem não predefinida, exigindo que o registrador as pré-processe para que possam ser chamadas corretamente.

Os registradores desempenham um papel crucial em:
- **Armazenar e formatar metadados** que serão usados pelos adaptadores (ex: HTTP, WebSocket).
- **Garantir a execução correta das funções**, pré-processando manipuladores e seus parâmetros.
- **Manter o comportamento singleton**, pois os registradores sempre operam de forma estática.

## Por que usar registradores?

Essa abordagem elimina a necessidade de declarar manualmente dependências em cada módulo. Como os registradores utilizam um **padrão de classe estática**, os serviços registrados podem ser acessados de qualquer parte da aplicação sem perder suas propriedades.

Por exemplo:
- Serviços e dependências podem ser **instanciados uma única vez** e armazenados no registrador.
- Essas instâncias podem ser recuperadas em qualquer parte da aplicação, como controladores, gateways e provedores.
- Registradores mais complexos, como os **Registradores de Controladores**, podem pré-processar manipuladores e parâmetros, facilitando a execução dinâmica dessas chamadas pelo framework.

## Exemplo de Registrador de Serviços

Abaixo está uma implementação simples de um **Registrador de Serviços**, responsável por registrar e gerenciar serviços.

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

## Registradores Genéricos

Para facilitar ainda mais a criação de registradores e decoradores personalizados, o CMMV inclui uma abstração genérica chamada **GenericRegistry**. Isso permite:

- Criar decoradores de classe, método e parâmetro de forma estruturada.
- Armazenar e recuperar metadados de maneira eficiente, sem precisar de Symbols ou APIs de reflexão.

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
}
```

## Como a aplicação lê os controladores?

A aplicação, por meio dos adaptadores HTTP, lê os **controladores** registrados no **Registry**, instancia as classes conforme os **providers** definidos no construtor e processa os **handlers** com seus respectivos **parâmetros e decoradores**. Isso permite a vinculação dinâmica das rotas e a execução correta das requisições.

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

## Registradores vs Reflexão

Usar registradores em vez de reflexão e símbolos oferece vantagens significativas em termos de **desempenho, eficiência e manutenção**. Diferente da reflexão, que recupera dinamicamente os metadados a cada acesso, os registradores armazenam e processam os metadados apenas uma vez durante a inicialização da aplicação. Isso elimina buscas redundantes e garante uma execução mais rápida ao evitar a resolução de metadados em tempo de execução.  

Registradores pré-processam e estruturam dados para uso eficiente, tornando os manipuladores de controladores e seus parâmetros prontos para execução imediata sem processamento adicional. Isso elimina cálculos desnecessários em tempo de execução, permite a recuperação eficiente de metadados de decoradores e garante dados ordenados para execução correta das funções.

A abordagem baseada em reflexão requer buscas repetidas de metadados, não oferece armazenamento estruturado e depende do comportamento da reflexão do TypeScript, que pode ter limitações em alguns ambientes. Registradores, por outro lado, armazenam dados de forma estática, permitem acesso direto baseado em chave em vez de buscas repetidas e são independentes do sistema de reflexão do TypeScript.  

Além disso, registradores se integram perfeitamente com injeção de dependência, tornando mais fácil recuperar e injetar dependências em toda a aplicação. Essa abordagem simplifica o armazenamento e recuperação de metadados, oferecendo melhor desempenho, acesso estruturado e manutenção em comparação com métodos baseados em reflexão. Ao carregar metadados uma única vez e estruturá-los de forma eficiente, a aplicação pode acessá-los dinamicamente sem penalidades de desempenho, tornando os registradores uma solução mais eficiente e escalável.
