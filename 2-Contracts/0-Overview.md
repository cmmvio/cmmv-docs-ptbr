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
- **generateController**: Define se os transpiladores devem gerar automaticamente controladores para este contrato.
- **auth**: Define acesso a dados que requerem autenticação.

## @ContractField
Define um campo de contrato com opções como:
- **protoType**: Tipo do campo (`string`, `number`, etc.).
- **protoRepeated**: Define se o campo é uma lista.
- **defaultValue**: Define um valor padrão ao criar o registro.
- **unique**: Garante unicidade.
- **index**: Define que o campo em questão é um índice.

### Tipos de Campo Suportados
O CMMV suporta uma variedade de tipos de campo:
- **Tipos Básicos**: `string`, `boolean`, `int`, `float`, `double`, `bytes`, `uuid`
- **Tipos Numéricos**: `int32`, `int64`, `uint32`, `uint64`, `sint32`, `sint64`, `fixed32`, `fixed64`, `sfixed32`, `sfixed64`
- **Tipos Avançados**: `json`, `jsonb`, `simpleArray`, `simpleJson`, `any`
- **Tipos de Data e Hora**: `date`, `time`, `timestamp`

Quando a aplicação é iniciada, o sistema gera automaticamente controladores, serviços e entidades genéricas com base nas definições do contrato. Esse processo simplifica o desenvolvimento ao garantir que componentes comuns, como operações CRUD, sejam pré-construídos e prontos para uso. Cada entidade e serviço é criado dinamicamente para corresponder às especificações do contrato, permitindo que o desenvolvedor foque na lógica personalizada sem precisar definir manualmente as estruturas básicas. Essa geração automática melhora a eficiência e a consistência em toda a aplicação.

## @ContractMessage

O decorador `@ContractMessage`, introduzido na versão 0.9, permite a definição explícita de mensagens estruturadas dentro dos contratos. Essas mensagens atuam como DTOs (Objetos de Transferência de Dados) que especificam as estruturas de dados necessárias para requisições RPC e RESTful, possibilitando uma comunicação mais eficiente e segura ao transmitir apenas os dados necessários, em vez de todo o esquema do contrato.

### Opções

* **name:** Define o nome da mensagem (usado nos DTOs gerados).
* **properties:** Especifica os campos da mensagem, cada um com:
    * **type:** O tipo de dado (por exemplo, string, boolean, number).
    * **required:** Indica se o campo é obrigatório.
    * **default: (Opcional)** Especifica um valor padrão para o campo.

Esses DTOs são integrados aos controladores e gateways gerados, garantindo consistência em toda a aplicação enquanto simplificam a validação e o gerenciamento de dados.

## Módulo HTTP

Para iniciar uma aplicação REST básica usando o módulo `@cmmv/http`, siga estes passos:

```
$ pnpm add @cmmv/http
```

Configure a aplicação:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [DefaultHTTPModule]
});
```

Para implementar corretamente uma aplicação CMMV básica usando os adaptadores Express ou Fastify, você pode usar a seguinte estrutura no seu arquivo `src/index.ts`. O sistema suporta tanto Express quanto Fastify para lidar com requisições HTTP, e o adaptador correto é selecionado na configuração.

Por padrão, ao iniciar a aplicação, ela será hospedada em `http://localhost:3000`. Isso pode ser facilmente modificado configurando a porta e o endereço de vinculação no arquivo de configuração da aplicação. As configurações permitem alterar o host, a porta e outras propriedades relacionadas ao servidor HTTP. Essas opções serão abordadas em mais detalhes posteriormente, quando mostrarmos a configuração do arquivo, proporcionando flexibilidade para ajustar o ambiente de execução do seu servidor.

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

Essa interface Task define a estrutura do modelo, incluindo campos opcionais e obrigatórios. O id é opcional, e campos como label, checked e removed são obrigatórios com tipos definidos. Você pode personalizar ou estender ainda mais este modelo com base nas necessidades da sua aplicação.

Essa interface base, como a interface Task, será utilizada por outros módulos, incluindo o módulo de repositório, para definir entidades no banco de dados. Ao compartilhar a mesma estrutura de interface entre os módulos, garante-se consistência e segurança de tipo em toda a aplicação. Por exemplo, o módulo de repositório usará essa interface Task para mapear e gerenciar operações de banco de dados, facilitando a manutenção e escalabilidade da sua aplicação enquanto garante que o modelo de dados esteja alinhado em diferentes camadas do sistema.

## Controlador

O contrato de exemplo fornecido acima gerará automaticamente um controlador no caminho `/src/controllers/task.controller.ts`. Este controlador será estruturado da seguinte forma:

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

Você pode criar seus próprios controladores personalizados simplesmente informando a aplicação sobre sua existência durante o processo de criação ou incluindo-os por meio de módulos. Isso permite maior flexibilidade e personalização na arquitetura da sua aplicação. Ao definir controladores personalizados, você pode registrá-los no módulo da aplicação durante a fase de configuração, garantindo que eles se integrem suavemente ao núcleo do framework, como roteamento HTTP ou manipulação de WebSocket, enquanto atendem aos requisitos específicos do seu projeto.

## Serviços

Assim como o controlador, a camada de serviço também é gerada automaticamente pelo sistema. Na ausência de módulos de persistência, como o repositório, um serviço placeholder é criado, que armazena temporariamente registros na memória enquanto a aplicação está online. Isso permite operações básicas como adicionar, atualizar e excluir dados sem um armazenamento persistente. Uma vez que um módulo de repositório é introduzido, ele substitui o serviço padrão, permitindo interação direta com um banco de dados, possibilitando que os dados sejam salvos, consultados e gerenciados eficientemente a partir de um armazenamento persistente.

Aqui está um exemplo de um serviço criado usando o módulo de repositório. Este serviço interage diretamente com o banco de dados usando o padrão de repositório:

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

    async getById(id: string, req?: any): Promise<TaskEntity> {
        const instance = Repository.getInstance();
        const repository = instance.dataSource.getRepository(TaskEntity);
        Telemetry.start('TaskService::GetById', req?.requestId);
        const item = await repository.findOneBy({ id });
        Telemetry.end('TaskService::GetById', req?.requestId);

        if (!item)
            throw new Error('Item não encontrado');

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

## Mais

A instalação de módulos adicionais, como RPC, cache e autenticação, estenderá a funcionalidade da sua aplicação. Cada módulo introduz novas capacidades, como habilitar chamadas de procedimento remoto, aumentar a segurança por meio de autenticação ou melhorar o desempenho com cache. Para mais detalhes, a documentação específica de cada módulo está disponível na barra lateral, onde você encontrará guias abrangentes sobre como integrar e implementar esses módulos de forma eficaz na sua aplicação.
