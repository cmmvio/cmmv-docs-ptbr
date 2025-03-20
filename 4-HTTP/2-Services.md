# Serviços

No ``@cmmv/core``, os serviços desempenham um papel crucial no gerenciamento da lógica de negócios, funcionando como uma ponte entre os repositórios de dados, o cache e as camadas de comunicação externa, como HTTP ou RPC. Os serviços são responsáveis por lidar com transformações de dados e garantir que a comunicação externa seja agnóstica ao modelo, o que significa que o serviço trabalha com modelos e os converte automaticamente conforme necessário.

Para criar um serviço, utilize o decorador ``@Service``. Esse decorador permite que o serviço seja reconhecido pelo framework e utilizado em toda a aplicação.

```typescript
@Service('task')
export class TaskService extends AbstractService {
    // Métodos e lógica do serviço
}
```

## Como os Serviços Funcionam

* **Modelos de Entrada e Saída:** Os serviços usam modelos (por exemplo, Task, ITask) para definir suas entradas e saídas. Esses modelos são automaticamente transformados ao interagir com repositórios ou camadas de comunicação externa.
* **Manuseio de Dados:** Os serviços interagem com o Repositório para lidar com a persistência e recuperação de dados. Eles também são responsáveis por validação, caching e tratamento de erros.
* **Telemetria:** O serviço inclui telemetria para rastreamento de desempenho e registro, ajudando a monitorar tempos de execução e identificadores de requisições.

## Usando Serviços

No ``@cmmv``, como o sistema não implementa injeção de dependências tradicional, você pode facilmente registrar serviços como provedores em qualquer módulo e acessá-los via construtor de uma classe. Isso simplifica o uso de serviços em controladores, gateways ou outros componentes.

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

## Agnóstico

No framework ``@cmmv``, os serviços são agnósticos ao tipo de controlador que os invoca, garantindo que possam ser usados em controladores HTTP, gateways RPC ou outros componentes. Os serviços são gerados automaticamente com base nas configurações de contratos e podem ser estendidos com o módulo ``@cmmv/repository`` para suportar entidades de banco de dados ou com o ``@cmmv/cache`` para gerenciar e recuperar dados em cache. Enquanto os serviços lidam com a lógica de negócios, diretivas de autenticação (``@cmmv/auth``) são aplicadas no nível do controlador ou gateway para garantir acesso seguro. Essa abordagem modular permite um gerenciamento de serviços flexível e escalável.

## Singleton

Os serviços de acesso global no ``@cmmv`` devem ser implementados como singletons para garantir comportamento consistente e uso eficiente de recursos em toda a aplicação. Singletons evitam a criação de múltiplas instâncias de um serviço, centralizando operações como caching e gerenciamento de banco de dados. Para saber mais sobre a implementação e os benefícios dos singletons no ``@cmmv``, consulte a documentação disponível em [Documentação de Singleton do CMMV](https://cmmv.io/docs/overview/singleton). Esta documentação fornece orientações detalhadas sobre como implementar e gerenciar serviços singleton de forma eficaz.
