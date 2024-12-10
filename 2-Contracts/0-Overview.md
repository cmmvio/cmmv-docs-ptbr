# Contratos

O sistema de contratos do CMMV permite definir modelos estruturados que são usados para gerar automaticamente APIs, RPCs e rotas WebSocket. Os contratos são definidos usando decoradores aplicados a classes e seus campos.

Os contratos são definidos usando os decoradores `@Contract` e `@ContractField`. Esses decoradores especificam o nome do contrato, o caminho do arquivo para buffers de protocolo e o tipo dos campos do contrato.

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
Define uma classe de contrato com as seguintes opções:
- **controllerName**: Especifica o nome do contrato.
- **controllerCustomPath**: Define um caminho personalizado para o controlador REST.
- **protoPath**: Especifica o caminho para o arquivo de buffer de protocolo.
- **protoPackage**: Define o namespace do contrato gerado no buffer de protocolo.
- **directMessage**: Define se as chamadas RPC são diretas ou se uma estrutura CRUD deve ser criada.
- **generateController**: Define se os controladores devem ser gerados automaticamente.
- **auth**: Define o acesso aos dados que exigem autenticação.

## @ContractField
Define um campo de contrato com opções como:
- **protoType**: Tipo do campo (`string`, `number`, etc.).
- **protoRepeated**: Define se o campo é uma lista.
- **defaultValue**: Define um valor padrão ao criar o registro.
- **unique**: Garante unicidade.
- **index**: Define que o campo em questão é um índice.

## Tipos de Campos Suportados
O CMMV suporta uma variedade de tipos de campos:
- **Tipos Básicos**: `string`, `boolean`, `int`, `float`, `double`, `bytes`, `uuid`
- **Tipos Numéricos**: `int32`, `int64`, `uint32`, `uint64`, `sint32`, `sint64`, `fixed32`, `fixed64`, `sfixed32`, `sfixed64`
- **Tipos Avançados**: `json`, `jsonb`, `simpleArray`, `simpleJson`, `any`
- **Tipos de Data e Hora**: `date`, `time`, `timestamp`

Ao iniciar a aplicação, o sistema gera automaticamente controladores, serviços e entidades genéricas com base nas definições de contrato. Esse processo simplifica o desenvolvimento, garantindo que componentes comuns, como operações CRUD, estejam pré-construídos e prontos para uso. Cada entidade e serviço é criado dinamicamente para corresponder às especificações do contrato, permitindo que o desenvolvedor foque na lógica personalizada sem precisar definir manualmente as estruturas básicas. Essa geração automática melhora a eficiência e consistência em toda a aplicação.

## Módulo HTTP

Para iniciar uma aplicação REST básica usando os módulos `@cmmv/http` e `@cmmv/view`, siga os passos abaixo:

```
$ pnpm add @cmmv/http @cmmv/view
```

Configure a aplicação:

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

A estrutura padrão da aplicação estará disponível em `http://localhost:3000`. Essa configuração pode ser facilmente modificada ajustando a porta e o endereço no arquivo de configuração da aplicação.

## Modelo

O sistema gerará automaticamente o modelo para sua entidade no seguinte formato:

```typescript
// Gerado automaticamente pelo CMMV

export interface Task {
    id?: any;
    label: string;
    checked: boolean;
    removed: boolean;
}
```

Essa interface define a estrutura do modelo, incluindo campos opcionais e obrigatórios. O `id` é opcional, enquanto `label`, `checked` e `removed` são obrigatórios com tipos definidos.

## Controlador

O exemplo de contrato acima gerará automaticamente um controlador no caminho `/src/controllers/task.controller.ts`. Este controlador será estruturado como mostrado abaixo:

```typescript
// Gerado automaticamente pelo CMMV

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

    // Outros métodos foram omitidos para brevidade.
}
```

## Serviços

Assim como o controlador, a camada de serviços também é gerada automaticamente. Aqui está um exemplo:

```typescript
// Gerado automaticamente pelo CMMV

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

    // Outros métodos foram omitidos para brevidade.
}
```

## Mais

A instalação de módulos adicionais, como RPC, caching e autenticação, pode estender significativamente a funcionalidade da sua aplicação. Cada módulo introduz novas capacidades, como a possibilidade de realizar chamadas de procedimento remoto (RPC), melhorar a segurança através da autenticação ou aumentar o desempenho com caching. Para mais detalhes, a documentação específica de cada módulo está disponível na barra lateral, onde você encontrará guias completos sobre como integrar e implementar esses módulos de forma eficiente na sua aplicação.

Por exemplo, ao incluir o módulo de autenticação (`@cmmv/auth`), você pode facilmente proteger suas rotas e serviços com autenticação baseada em JWT ou outros métodos de autenticação suportados. Da mesma forma, o módulo de cache (`@cmmv/cache`) pode ser integrado para otimizar o desempenho armazenando dados em memória ou em um store Redis para acesso rápido. Para maior flexibilidade, você pode configurar cada módulo com base nas necessidades específicas do seu projeto.

O sistema de contratos do CMMV, juntamente com seus módulos complementares, oferece uma solução robusta e escalável para desenvolver aplicativos modernos e altamente integrados, eliminando grande parte da codificação repetitiva e permitindo que os desenvolvedores se concentrem em resolver problemas específicos do domínio. Isso resulta em um desenvolvimento mais ágil e eficiente, com maior consistência e qualidade em toda a aplicação.
