# CLI

O CMMV CLI simplifica a inicialização do projeto ao fornecer uma maneira interativa de criar um novo projeto com configurações personalizáveis. Abaixo está a documentação atualizada para usar o CLI para gerar um projeto CMMV.

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
        Isso garantirá que todas as compilações necessárias sejam aprovadas automaticamente durante a instalação, evitando problemas relacionados a prompts de aprovação manual.
    </p>
</div>

## Primeiros passos

Instale a CLI globalmente: para usar a CLI globalmente em seu sistema, instale-a usando ``pnpm``:

```bash
$ pnpm add -g @cmmv/cli@latest
```

Crie um novo projeto: execute o comando ``cmmv create`` para criar um novo projeto:

```bash
$ cmmv create project-name
```

Isso iniciará um prompt interativo solicitando suas preferências de projeto, como:

* Se deve habilitar o Vite Middleware
* Use RPC (WebSocket + Protobuf)
* Habilite o módulo Cache
* Selecione o tipo de repositório (SQLite, MongoDB, PostgreSQL, MySQL, MSSQL, Oracle)
* Selecione o tipo de cache (Redis, Memcached, MongoDB, File System)
* Selecione o tipo de fila (Redis, RabbitMQ, Kafka)
* Habilite ESLint, Prettier e Vitest

## Usando

Se você não quiser instalar a CLI globalmente, use ``pnpm dlx`` para executá-la diretamente:

```bash
$ pnpm dlx @cmmv/cli@latest create project-name
```

Isso garante que você sempre use a versão mais recente da CLI sem exigir uma instalação global.

## Alterações na versão 5.9

A CLI do CMMV foi refatorada, introduzindo novos comandos e simplificando os fluxos de trabalho do projeto. A atualização de versões anteriores é altamente recomendada.

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso</p>
    <p>
        A partir da versão <strong>v.0.5.9</strong>, o <strong>@cmmv/cli</strong> foi totalmente refatorado, introduzindo novos comandos e simplificando os fluxos de trabalho do projeto.
        É altamente recomendável atualizar para esta versão para melhor desempenho e usabilidade.
    </p>
    <p>
        A CLI agora inclui suporte integrado para <strong>ESLint</strong>, <strong>lançamentos de módulos</strong> e um <strong>sistema de construção</strong> aprimorado com suporte para <strong>ESM</strong> e <strong>CJS</strong>.
        Além disso, <strong>hot reload</strong> para desenvolvimento e um novo <strong>comando run</strong> para executar scripts agora estão disponíveis.
    </p>
</div>

### Novos recursos

* ESLint integrado: não há necessidade de dependências separadas — execute `$ cmmv lint`.
* Automação de lançamento de módulo: `$ cmmv release` agora lida com todas as dependências automaticamente.
* Sistema de compilação aprimorado: Suporta compilações ESM e CJS: `$ cmmv build`
* Modo de desenvolvimento aprimorado:
* Watch e debug integrados sem `nodemon`
* Suporte para hot reload usando `$ cmmv dev`
* Exemplo de configuração:

```json
"dev": {
    "watch": ["src", "docs"],
    "ignore": ["**/*.spec.ts", "src/app.module.ts", "docs/**/*.html"]
}
```
* Comando de início de produção: `$ cmmv start`
* Execução de script: `$ cmmv run ./src/<script>.ts`
* Aumento de desempenho:
* `@swc-node/register` substitui `ts-node` para dev e run.
* `tsc` continua sendo o padrão para compilações de produção.

* Configuração ESLint:
* Usa ESLint 9.20 com Prettier.
* Configurável via `eslint.config.cjs` (ou `eslint.config.ts` para projetos ESM).

## Estrutura de Projeto Gerada

A CLI gera uma pasta de projeto estruturada com os arquivos e diretórios necessários com base em suas preferências. Abaixo está um exemplo de estrutura:

```
.
├── .generated/
├── src/
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── tests/
│   ├── app.controller.spec.ts
│   ├── app.module.spec.ts
│   └── app.service.spec.ts
├── tsconfig.json
├── eslint.config.cjs
├── .cmmv.config.cjs
├── package.json
├── .gitignore
└── ...
```

**Arquivos de configuração gerados**
* ``.cmmv.config.cjs``: Configuração central para o aplicativo CMMV.
* ``package.json``: Inclui dependências e scripts necessários com base nas opções selecionadas.
* ``tsconfig.json``: Referências para configurações TypeScript.
* ``.gitignore``, ``.npmignore``, ``.prettierignore``, ``.prettierrc``, ``.swcrc``: Arquivos pré-configurados para padrões de desenvolvimento.

## Scripts disponíveis

Modo de desenvolvimento:

```bash
$ pnpm dev
```

Construir para produção:

```bash
$ pnpm build
```

Iniciar servidor de produção:

```bash
$ pnpm start
```

Executar testes (se o Vitest estiver habilitado):

```bash
$ pnpm test
```

Executar ESLint:

```bash
$ pnpm lint
```

## Módulo

A CLI do CMMV agora inclui um comando ``module`` para simplificar a criação de novos módulos em um projeto CMMV existente. Os módulos ajudam a organizar seu aplicativo em unidades reutilizáveis ​​e específicas de recursos. Abaixo está a documentação para o comando ``module``.

```bash
$ cmmv module <nome-do-módulo>
```

## Estrutura do módulo gerado

```bash
module/
├── src/
│   ├── main.ts
├── tests/
│   └── main.spec.ts
├── .gitignore
├── .npmignore
├── eslint.config.cjs
├── tsconfig.json
├── tsconfig.cjs.json
├── tsconfig.esm.json
├── package.json
└── ...
```

O pacote gerado.json inclui metadados e scripts essenciais para o módulo:

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
        "node": ">=20.0.0"
    },
    "scripts": {
        "build": "cmmv build",
        "lint": "cmmv lint",
        "release": "cmmv release",
        "test": "vitest",
        "prepare": "husky install",
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

O comando `cmmv contract` permite que você crie contratos na estrutura CMMV. Os contratos definem a estrutura, validação e metadados para as entidades e controladores do seu aplicativo. Abaixo está a documentação para usar o comando `contract`.

### Uso

Para criar um contrato, use o seguinte comando:

```bash
$ cmmv contract contract-name
```

Isso iniciará um prompt interativo para configurar seu contrato. Você pode definir o nome do contrato, metadados, campos e regras de validação.

### Prompts interativos

A CLI solicitará os seguintes detalhes:

* **Metadados do contrato:** Configurações como nome do controlador, caminho proto, pacote proto, cache e importações.
* **Campos:** Adicione campos ao contrato com propriedades como tipo proto, valor padrão, validações e muito mais.

### Estrutura do contrato

Depois que o contrato for criado, ele será adicionado ao diretório src/contracts com a seguinte estrutura:

```bash
src/contracts/
├── <contract-name>.contract.ts
```

### Exemplo

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    importações: ['crypto'],
    cache: {
        chave: 'task:',
        ttl: 300,
        compress: true,
    },
})
export class TasksContract estende AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validações: [
            {
                type: 'IsString',
                mensagem: 'Rótulo inválido',
            },
            {
                type: 'IsNotEmpty',
                mensagem: 'Rótulo inválido',
            },
        ],
    })
    rótulo: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validates: [
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
        validates: [
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

Para integrar um contrato ao seu aplicativo, você precisa registrá-lo na configuração `Application` dentro do seu `index.ts` ou `server.ts` ou arquivo de ponto de entrada. Isso garante que o contrato esteja disponível para uso no seu aplicativo.

Inclua o contrato na propriedade contracts da configuração Application.create. Aqui está um exemplo:

```typescript
// Imports

import { TasksContract } from './contracts/tasks.contract'; // Registre sua importação aqui

// Crie o aplicativo
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
