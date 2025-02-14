# CLI

O CLI do CMMV simplifica a inicialização de projetos, oferecendo uma maneira interativa de criar um novo projeto com configurações personalizáveis. Abaixo está a documentação atualizada para usar o CLI e gerar um projeto CMMV.

## Primeiros Passos

Instale o CLI globalmente: Para usar o CLI globalmente em seu sistema, instale-o usando `pnpm`:

```bash 
$ pnpm add -g @cmmv/cli
```

Crie um Novo Projeto: Execute o comando `cmmv create` para criar um novo projeto:

```bash
$ cmmv create <nome-do-projeto>
```

Isso iniciará um prompt interativo perguntando sobre suas preferências de projeto, como:

* Ativar o Middleware Vite
* Usar RPC (WebSocket)
* Ativar o módulo Cache
* Selecionar o tipo de repositório (SQLite, MongoDB, PostgreSQL, MySQL)
* Escolher a configuração de visualização (Reatividade, Vue3 ou Vue3 + TailwindCSS)
* Ativar ESLint, Prettier e Vitest

## Usando `pnpm dlx`

Se você não quiser instalar o CLI globalmente, use o `pnpm dlx` para executá-lo diretamente:

```bash
$ pnpm dlx @cmmv/cli@latest create <nome-do-projeto>
```

Isso garante que você sempre use a versão mais recente do CLI sem exigir uma instalação global.

## Estrutura do Projeto Gerado

O CLI gera uma pasta de projeto estruturada com os arquivos e diretórios necessários com base em suas preferências. Abaixo está um exemplo de estrutura:

```
.
├── public/
│   ├── assets/
│   │   └── protobuf.min.js (se o RPC estiver ativado)
│   ├── core/
│   ├── templates/
│   └── views/
├── src/
│   ├── contracts/
│   ├── controllers/
│   ├── entities/
│   ├── models/
│   ├── modules/
│   ├── services/
│   ├── locale/
│   ├── app.module.ts
│   ├── server.ts
│   └── client.ts (se o Vite estiver ativado)
├── node_modules/
├── index.html (se o Vite estiver ativado)
├── tailwind.config.js (se o TailwindCSS estiver ativado)
├── tsconfig.json
├── tsconfig.client.json (se o Vite estiver ativado)
├── tsconfig.vue.json (se Vue3 ou Vue3 + TailwindCSS estiver ativado)
├── vite.config.js (se o Vite estiver ativado)
├── vitest.config.ts (se o Vitest estiver ativado)
├── .cmmv.config.cjs
├── package.json
├── .gitignore
└── ...
```

**Arquivos de Configuração Gerados**
* `.cmmv.config.cjs`: Configuração central para a aplicação CMMV.
* `package.json`: Inclui as dependências e scripts necessários com base nas opções selecionadas.
* `tsconfig.json`: Referências para configurações do TypeScript.
* `.gitignore`, `.npmignore`, `.prettierignore`, `.prettierrc`, `.swcrc`: Arquivos pré-configurados para padrões de desenvolvimento.
* `vite.config.js`: Configuração do Vite (se ativado).
* `tailwind.config.js` e `src/tailwind.css`: Configuração do TailwindCSS (se ativado).

## Scripts Disponíveis

Modo de Desenvolvimento:

```bash
$ pnpm dev
```

Compilar para Produção:

```bash
$ pnpm build
```

Iniciar Servidor em Produção:

```bash
$ pnpm start
```

Executar Testes (se o Vitest estiver ativado):

```bash
$ pnpm test
```

## Módulo 

O CLI do CMMV agora inclui o comando `module` para simplificar a criação de novos módulos dentro de um projeto CMMV existente. Os módulos ajudam a organizar sua aplicação em unidades reutilizáveis e específicas por funcionalidade. Abaixo está a documentação para o comando `module`.

```bash
$ cmmv module <nome-do-módulo>
```

## Estrutura do Módulo Gerado

```bash
module/
├── src/
│   ├── index.ts                # Ponto de entrada principal do módulo
├── scripts/
│   └── release.js (se o release estiver ativado)
├── tests/
│   └── index.test.ts (se o vitest estiver ativado)
├── .gitignore
├── .npmignore
├── .swcrc
├── tsconfig.json
├── tsconfig.cjs.json
├── tsconfig.esm.json
├── package.json
└── ...
```

O arquivo `package.json` gerado inclui metadados essenciais e scripts para o módulo:

```json
{
    "name": "module",
    "version": "0.0.1",
    "description": "",
    "keywords": [],
    "author": "",
    "publishConfig": {
        "access": "public"
    },
    "engines": {
        "node": ">=18.18.0 || >=20.0.0"
    },
    "scripts": {
        "build:cjs": "tsc --project tsconfig.cjs.json",
        "build:esm": "tsc --project tsconfig.esm.json",
        "build": "npm run build:cjs && npm run build:esm",
        "test": "vitest",
        "prepare": "husky install",
        "lint": "pnpm run lint:spec",
        "lint:fix": "pnpm run lint:spec -- --fix",
        "release": "node scripts/release.js",
        "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
    },
    "devDependencies": {
        ...
    },
    "dependencies": {
        "@cmmv/core": "^1.0.0"
    }
}
```

## Contrato

O comando `cmmv contract` permite criar contratos no framework CMMV. Contratos definem a estrutura, validação e metadados para as entidades e controladores da sua aplicação. Abaixo está a documentação para usar o comando `contract`.

### Uso

Para criar um contrato, use o seguinte comando:

```bash
$ cmmv contract <nome-do-contrato>
```

Isso iniciará um prompt interativo para configurar seu contrato. Você pode definir o nome, metadados, campos e regras de validação do contrato.

### Prompts Interativos

O CLI pedirá os seguintes detalhes:

* **Metadados do Contrato:** Configurações como nome do controlador, caminho do proto, pacote proto, cache e imports.
* **Campos:** Adicione campos ao contrato com propriedades como tipo proto, valor padrão, validações, entre outros.

### Estrutura do Contrato

Depois que o contrato for criado, ele será adicionado ao diretório `src/contracts` com a seguinte estrutura:

```bash
src/contracts/
├── <nome-do-contrato>.contract.ts
```

### Exemplo

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    imports: ['crypto'],
    cache: {
        key: 'task:',
        ttl: 300,
        compress: true,
    },
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [
            {
                type: 'IsString',
                message: 'Invalid label',
            },
            {
                type: 'IsNotEmpty',
                message: 'Invalid label',
            },
        ],
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [
            {
                type: 'IsBoolean',
                message: 'Invalid checked type',
            },
        ],
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [
            {
                type: 'IsBoolean',
                message: 'Invalid removed type',
            },
        ],
    })
    removed: boolean;

    @ContractField({ protoType: 'date' })
    createdAt?: Date;
}
```

### Integração

Para integrar um contrato à sua aplicação, você precisa registrá-lo na configuração do `Application` dentro de seu arquivo `index.ts` ou `server.ts` ou arquivo de entrada. Isso garante que o contrato esteja disponível para uso em sua aplicação.

Inclua o contrato na propriedade `contracts` da configuração `Application.create`. Veja um exemplo:

```typescript
// Imports

import { TasksContract } from './contracts/tasks.contract'; // Registre sua importação aqui

// Crie a aplicação
Application.create({
    httpAdapter: DefaultAdapter,
    wsAdapter: WSAdapter,
    modules: [
        ...
    ],
    services: [...],
    transpilers: [...],
    contracts: [TasksContract], // Registre seu contrato aqui
});
```
