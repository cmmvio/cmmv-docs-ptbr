# Repositório

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/repository](https://github.com/cmmvio/cmmv/tree/main/packages/repository)

O módulo ``@cmmv/repository`` foi projetado para funcionar em conjunto com o framework ``@cmmv/core`` e é construído utilizando [TypeORM](https://typeorm.io/). Este módulo gera automaticamente entidades com base nos contratos definidos e estende a camada de serviço para lidar com operações CRUD com o banco de dados configurado. A seguir, fornecemos informações detalhadas sobre instalação, uso e recursos deste módulo.

**Instalação**

Para usar o módulo ``@cmmv/repository`` em seu projeto, você pode instalá-lo via npm:

```bash
$ pnpm add @cmmv/repository
```

Além de instalar o módulo ``@cmmv/repository``, dependendo do tipo de banco de dados que você está utilizando, será necessário instalar o respectivo driver de banco de dados.

**Bancos de Dados Suportados e Drivers**

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #2563EB;
    color: #1E3A8A;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso</p>
    <p>
        Atualizações recentes do <strong>PNPM</strong> podem exigir aprovação manual para módulos com scripts de construção, afetando pacotes como
        <strong>better-sqlite3-multiple-ciphers</strong>, <strong>esbuild</strong>, <strong>protobufjs</strong>, <strong>sqlite3</strong>, entre outros.
    </p>
    <p>
        Uma correção para esse problema foi implementada a partir da <strong>versão 0.8.20</strong>. No entanto, se seu projeto estiver usando uma versão mais antiga,
        você pode resolver isso manualmente adicionando a seguinte configuração ao seu arquivo <code>.npmrc</code>:
    </p>
    <pre style="
        background-color: #F3F4F6;
        padding: 0.75rem;
        border-radius: 0.375rem;
        overflow-x: auto;
    ">
auto-install-peers=true
approve-builds=always</pre>
    <p>
        Isso garantirá que todas as construções necessárias sejam aprovadas automaticamente durante a instalação, evitando problemas relacionados a prompts de aprovação manual.
    </p>
</div>

O TypeORM suporta vários sistemas de banco de dados, e cada sistema requer o pacote npm apropriado para funcionar corretamente. Aqui está uma lista de bancos de dados suportados junto com o driver correspondente a instalar:

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

Após instalar o módulo e o driver de banco de dados correspondente, você pode configurar sua conexão com o banco de dados no arquivo ``.cmmv.config.cjs``. O TypeORM usará automaticamente o driver apropriado com base na configuração.

## Propósito

O módulo ``@cmmv/repository`` simplifica o processo de integração de entidades de banco de dados e serviços para operações CRUD, aproveitando os contratos definidos usando o ``@cmmv/core``. Ele transpila automaticamente as definições de contratos em entidades TypeORM e fornece uma maneira padronizada de gerenciar interações com o banco de dados.

* **Geração Automática de Entidades:** Com base nos contratos definidos em seu projeto, o módulo transpila as definições de contrato em entidades TypeORM.
* **Serviço CRUD:** O módulo gera operações CRUD para as entidades sem a necessidade de implementar serviços manualmente, permitindo um gerenciamento fácil dos dados.
* **Integração com TypeORM:** Integração completa com TypeORM, facilitando o trabalho com uma variedade de bancos de dados, como PostgreSQL, MySQL, SQLite, etc.
* **Baseado em Contratos:** Usando contratos definidos com ``@cmmv/core``, o módulo gera automaticamente tanto o esquema do banco de dados quanto a camada de serviço CRUD.

``/src/contract/task.contract.ts``
```typescript
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

Uma vez que o contrato é definido, o módulo ``@cmmv/repository`` transpila o contrato em uma entidade TypeORM. Abaixo está um exemplo de uma entidade gerada automaticamente (``task.entity.ts``):

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

O módulo ``@cmmv/repository`` também gera um serviço CRUD que lida automaticamente com interações com o banco de dados. Este serviço usará a entidade gerada a partir do contrato para fornecer métodos CRUD padrão (criar, ler, atualizar, excluir).

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

O módulo ``@cmmv/repository`` simplifica significativamente a interação com o banco de dados ao gerar automaticamente as entidades e serviços necessários com base nos contratos predefinidos. Com a integração embutida ao TypeORM, este módulo agiliza as operações CRUD, possibilitando um desenvolvimento mais rápido e um gerenciamento mais fácil do banco de dados.

## Configurações

Para garantir que o módulo ``@cmmv/repository`` funcione corretamente em seu projeto, você precisa definir as configurações do banco de dados em um arquivo ``.cmmv.config.cjs``. Este arquivo serve como a configuração central do seu projeto, incluindo as configurações do repositório para interação com o banco de dados.

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

Neste exemplo, o módulo está configurado para usar SQLite como banco de dados com sincronização automática habilitada. Isso significa que quaisquer mudanças em suas entidades serão automaticamente refletidas no esquema do banco de dados sem a necessidade de migrações.

Para uma lista mais detalhada de todas as configurações disponíveis, visite a [documentação de opções de fonte de dados do TypeORM](https://typeorm.io/data-source). Lá, você encontrará opções adicionais como:

* **entities:** Um array de caminhos ou classes para especificar quais entidades o TypeORM deve gerenciar.
* **migrations:** Um caminho para seus arquivos de migração.
* **cache:** Habilita o cache de consultas.
* **username e password:** Credenciais para conexão com o banco de dados (para tipos de banco que exigem autenticação).
* **host e port:** Para especificar o host e a porta para bancos como PostgreSQL, MySQL, etc.

Para configurar corretamente o módulo ``@cmmv/repository``, você precisa especificar as configurações de conexão do banco de dados no arquivo ``.cmmv.config.cjs``. O exemplo fornecido mostra uma configuração típica usando SQLite, mas você pode ajustar a configuração para outros bancos como PostgreSQL ou MySQL. Para mais detalhes sobre configurações possíveis, consulte a [documentação do TypeORM](https://typeorm.io).

## Índice Personalizado

A partir da versão 0.9, o módulo ``@cmmv/repository`` introduz suporte para declarações de índices personalizados dentro dos contratos. Esse recurso permite que os desenvolvedores definam índices mais complexos diretamente no contrato usando o parâmetro `index` no decorador ``@Contract``. Ele suporta índices de múltiplos campos e inclui todas as opções de indexação disponíveis fornecidas pelo TypeORM.

```typescript
@Contract({
    controllerName: 'User',
    protoPath: 'src/protos/auth.proto',
    protoPackage: 'auth',
    generateController: true,
    auth: true,
    imports: ['crypto'],
    index: [
        {
            name: 'idx_user_login',
            fields: ['username', 'password'],
            options: {
                where: "password IS NOT NULL",
                expireAfterSeconds: 3600,
            },
        },
    ],
})
export class UserContract extends AbstractContract {
    @ContractField({ protoType: 'string', unique: true })
    username: string;

    @ContractField({ protoType: 'string' })
    password: string;

    @ContractField({ protoType: 'string', nullable: true })
    googleId: string;

    @ContractField({ protoType: 'string', defaultValue: '"[]"' })
    groups: string;

    @ContractField({ protoType: 'string', defaultValue: '"[]"' })
    roles: string;

    @ContractField({ protoType: 'bool', defaultValue: false })
    root: boolean;
}
```

### Opções de Índice

O parâmetro ``index`` aceita um array de definições de índices, cada uma contendo as seguintes opções:

```typescript
export interface ContractIndexOptions {
    unique?: boolean;
    spatial?: boolean;
    fulltext?: boolean;
    nullFiltered?: boolean;
    parser?: string;
    where?: string;
    sparse?: boolean;
    background?: boolean;
    concurrent?: boolean;
    expireAfterSeconds?: number;
}
```

### Entidade Gerada

O módulo ``@cmmv/repository`` gerará a entidade TypeORM correspondente com os índices definidos:

```typescript
@Entity('user')
@Index("idx_user_username", ["username"], { unique: true })
@Index("idx_user_googleId", ["googleId"])
@Index("idx_user_login", ["username", "password"], {
    where: "password IS NOT NULL",
    expireAfterSeconds: 3600
})
export class UserEntity implements IUser {
    @ObjectIdColumn()
    _id: ObjectId;

    @Column({ type: 'varchar' })
    username: string;

    @Column({ type: 'varchar' })
    password: string;

    @Column({ type: 'varchar', nullable: true })
    googleId: string;

    @Column({ type: 'varchar', default: "[]" })
    groups: string;

    @Column({ type: 'varchar', default: "[]" })
    roles: string;

    @Column({ type: 'boolean', default: false })
    root: boolean;
}
```

## Suporte a Migrações

Com a integração ao TypeORM, o módulo ``@cmmv/repository`` agora oferece suporte completo a migrações para gerenciar alterações no esquema do banco de dados. Esse recurso permite que os desenvolvedores mantenham controle sobre a evolução do banco de dados, sincronizando automaticamente mudanças a partir dos contratos ou gerando migrações personalizadas.

### Configuração de Migrações

As configurações relacionadas a migrações foram adicionadas ao arquivo ``.cmmv.config.cjs`` dentro do objeto `repository`. Os novos parâmetros são:

```javascript
module.exports = {
    repository: {
        type: "sqlite",
        database: "./database.sqlite",
        synchronize: true, // Cuidado: use com cautela em produção
        logging: false,
        migrations: true,
        migrationsDir: "/src/migrations" // Diretório padrão para geração de migrações
    },
};
```
<br/>

Esses parâmetros permitem que o TypeORM localize e execute as migrações, bem como defina onde novas migrações serão criadas.

### Criação de Migrações

Existem duas formas principais de criar migrações com o módulo ``@cmmv/repository``:

#### 1. Geração Automática via ``@cmmv/sandbox``
Contratos criados usando o módulo ``@cmmv/sandbox`` geram automaticamente arquivos de migração correspondentes às entidades definidas. Quando um contrato é criado ou modificado no sandbox, o sistema detecta as alterações e cria os arquivos de migração necessários no diretório especificado por `migrationsDir`. Isso inclui a criação de tabelas, índices e quaisquer ajustes nos campos.

#### 2. Geração Manual com `RepositoryMigration`
Para cenários mais personalizados ou quando os contratos são definidos fora do sandbox, você pode usar a função ``RepositoryMigration.generateMigration`` para gerar migrações com base nas diferenças entre dois contratos:

```typescript
import { RepositoryMigration } from '@cmmv/repository';

// Exemplo: Criar uma migração para um novo contrato
const currentContract = null; // Nenhum contrato anterior
const updatedContract = {
    controllerName: 'User',
    fields: [
        { name: 'username', protoType: 'string', unique: true },
        { name: 'email', protoType: 'string' }
    ]
};

await RepositoryMigration.generateMigration(currentContract, updatedContract);

// Exemplo: Atualizar um contrato existente
const currentContract = {
    controllerName: 'User',
    fields: [
        { name: 'username', protoType: 'string', unique: true }
    ]
};
const updatedContract = {
    controllerName: 'User',
    fields: [
        { name: 'username', protoType: 'string', unique: true },
        { name: 'email', protoType: 'string' }
    ]
};

await RepositoryMigration.generateMigration(currentContract, updatedContract);

// Exemplo: Excluir uma tabela
const currentContract = {
    controllerName: 'User',
    fields: [
        { name: 'username', protoType: 'string', unique: true }
    ]
};
const updatedContract = null;

await RepositoryMigration.generateMigration(currentContract, updatedContract);
```
<br/>

- **Casos de Uso:**
  - **`currentContract` nulo, `updatedContract` preenchido**: Gera uma migração para criar uma tabela (`CREATE TABLE`) e índices com base no contrato fornecido.
  - **Ambos os parâmetros preenchidos**: Compara os dois contratos e gera uma migração (`ALTER TABLE`) com alterações nos campos e índices.
  - **`currentContract` preenchido, `updatedContract` nulo**: Gera uma migração para excluir a tabela (`DROP TABLE`).

Os arquivos de migração gerados são salvos no diretório especificado por `migrationsDir` e seguem o padrão do TypeORM, permitindo edição manual, se necessário.

### Integração com Comandos de Migração do TypeORM

Para simplificar o uso das migrações, adicione os seguintes scripts ao seu `package.json`:

```json
{
    "scripts": {
        "typeorm": "node -r @swc-node/register ./node_modules/typeorm/cli.js",
        "migration:run": "pnpm typeorm migration:run -d src/migration.ts",
        "schema:sync": "pnpm typeorm schema:sync -d src/migration.ts",
        "migration:show": "pnpm typeorm migration:show -d src/migration.ts",
        "migration:generate": "pnpm typeorm migration:generate -d src/migration.ts",
        "migration:create": "pnpm typeorm migration:create -d src/migration.ts"
    }
}
```

Além disso, crie um arquivo chamado `migration.ts` dentro do diretório `src` para configurar a fonte de dados do TypeORM:

```typescript
import { Repository } from '@cmmv/repository';
const datasource = Repository.getDataSource();
export default datasource;
```

Esses scripts fornecem uma integração fluida com a CLI do TypeORM, permitindo que você execute migrações, sincronize esquemas e gere novos arquivos de migração diretamente da linha de comando.
