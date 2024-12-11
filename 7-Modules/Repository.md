# Repositório

O módulo ``@cmmv/repository`` foi projetado para funcionar em conjunto com o framework ``@cmmv/core`` e é construído utilizando [TypeORM](https://typeorm.io/). Este módulo gera automaticamente entidades com base nos contratos definidos e estende a camada de serviço para lidar com operações CRUD no banco de dados configurado. Abaixo, fornecemos informações detalhadas sobre instalação, uso e recursos deste módulo.

**Instalação**

Para usar o módulo ``@cmmv/repository`` em seu projeto, você pode instalá-lo via npm:

```bash
$ pnpm add @cmmv/repository typeorm
```

Além de instalar o módulo ``@cmmv/repository``, dependendo do tipo de banco de dados que você está usando, será necessário instalar o driver correspondente.

**Bancos de Dados Suportados e Drivers**

O TypeORM suporta múltiplos sistemas de banco de dados, e cada sistema requer o pacote npm apropriado para funcionar corretamente. Aqui está uma lista de bancos de dados suportados com o respectivo driver para instalação:

**SQLite:** [NPM](https://www.npmjs.com/package/sqlite3)

```bash
$ pnpm add sqlite3
```

**MySQL / MariaDB:** [NPM](https://www.npmjs.com/package/mysql2)

```bash
$ pnpm add mysql2
```

**PostgreSQL:** [NPM](https://www.npmjs.com/package/pg)

```bash
$ pnpm add pg
```

**Microsoft SQL Server:** [NPM](https://www.npmjs.com/package/mssql)

```bash
$ pnpm add mssql
```

**Oracle:** [NPM](https://www.npmjs.com/package/oracledb)

```bash
$ pnpm add oracledb
```

**MongoDB:** [NPM](https://www.npmjs.com/package/mongodb)

```bash
$ pnpm add mongodb
```

Depois de instalar o módulo e o driver correspondente, você pode configurar a conexão com o banco de dados no arquivo ``.cmmv.config.js``. O TypeORM usará automaticamente o driver apropriado com base na configuração.

## Propósito

O módulo ``@cmmv/repository`` simplifica o processo de integração de entidades e serviços de banco de dados para operações CRUD, aproveitando contratos definidos com ``@cmmv/core``. Ele transpila automaticamente os contratos em entidades do TypeORM e fornece uma maneira padronizada de gerenciar interações com o banco de dados.

* **Geração Automática de Entidades:** Com base nos contratos definidos em seu projeto, o módulo transpila as definições do contrato em entidades do TypeORM.
* **Serviço CRUD:** O módulo gera operações CRUD para as entidades sem a necessidade de implementar serviços manualmente, permitindo um gerenciamento fácil de dados.
* **Integração com TypeORM:** Integração completa com o TypeORM, facilitando o trabalho com vários bancos de dados, como PostgreSQL, MySQL, SQLite, etc.
* **Baseado em Contratos:** Usando contratos definidos com ``@cmmv/core``, o módulo gera automaticamente o esquema do banco de dados e a camada de serviço CRUD.

```typescript
// /src/contract/task.contract.ts
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
    })
    removed: boolean;
}
```

Este contrato define os campos e a estrutura para a entidade Task, que será transpilada em uma entidade de banco de dados pelo módulo ``@cmmv/repository``.

Uma vez definido o contrato, o módulo ``@cmmv/repository`` transpila o contrato em uma entidade do TypeORM. Abaixo está um exemplo de uma entidade gerada automaticamente (``task.entity.ts``):

```typescript
// Gerado automaticamente pelo CMMV

import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';
import { Task } from '../models/task.model';

@Entity('task')
@Index("idx_task_label", ["label"], { unique: true })
export class TaskEntity implements Task {
    @PrimaryGeneratedColumn('uuid')
    id: string;

    @Column({ type: 'varchar' })
    label: string;

    @Column({ type: 'boolean', default: false })
    checked: boolean;

    @Column({ type: 'boolean', default: false })
    removed: boolean;
}
```

O módulo ``@cmmv/repository`` também gera um serviço CRUD que lida automaticamente com interações com o banco de dados. Este serviço usa a entidade gerada a partir do contrato para fornecer métodos CRUD padrão (create, read, update, delete).

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

O módulo ``@cmmv/repository`` simplifica significativamente a interação com o banco de dados, gerando automaticamente as entidades e serviços necessários com base em contratos pré-definidos. Com integração nativa ao TypeORM, este módulo agiliza operações CRUD, permitindo um desenvolvimento mais rápido e fácil gerenciamento do banco de dados.

## Configurações

Para garantir que o módulo ``@cmmv/repository`` funcione corretamente com seu projeto, é necessário definir as configurações do banco de dados em um arquivo ``.cmmv.config.js``. Este arquivo serve como a configuração central para seu projeto, incluindo as configurações de repositório para interação com o banco de dados.

```javascript
module.exports = {
    // Outras configurações do projeto...

    repository: {
        type: "sqlite", 
        database: "./database.sqlite",
        synchronize: true,
        logging: false,
    },
};
```

Neste exemplo, o módulo está configurado para usar SQLite como banco de dados com sincronização automática ativada. Isso significa que quaisquer alterações nas suas entidades serão refletidas automaticamente no esquema do banco de dados sem a necessidade de migrações.

Para uma lista mais detalhada de todas as configurações disponíveis, visite a [documentação de opções de Data Source do TypeORM](https://typeorm.io/data-source). Lá, você encontrará opções adicionais como:

* **entities:** Um array de caminhos ou classes para especificar quais entidades o TypeORM deve gerenciar.
* **migrations:** Um caminho para seus arquivos de migração.
* **cache:** Habilitar cache de consultas.
* **username e password:** Credenciais para conexão com o banco de dados (para tipos de banco de dados que exigem autenticação).
* **host e port:** Para especificar o host e a porta para bancos de dados como PostgreSQL, MySQL, etc.

Para configurar adequadamente o módulo ``@cmmv/repository``, você deve especificar suas configurações de conexão com o banco de dados no arquivo ``.cmmv.config.js``. O exemplo fornecido mostra uma configuração típica usando SQLite, mas você pode ajustar as configurações para outros bancos de dados como PostgreSQL ou MySQL. Para mais detalhes sobre configurações possíveis, consulte a [documentação do TypeORM](https://typeorm.io).
