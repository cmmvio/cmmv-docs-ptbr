# CLI

O CMMV CLI simplifica a inicialização de projetos, fornecendo uma maneira interativa de criar um novo projeto com configurações personalizáveis. Abaixo está a documentação atualizada para usar o CLI na geração de um projeto CMMV.

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

## Introdução

### Instalar o CLI globalmente

Para usar o CLI globalmente em seu sistema, instale-o com `pnpm`:

```bash
$ pnpm add -g @cmmv/cli
```

### Criar um novo projeto

Execute o comando `cmmv create` para criar um novo projeto:

```bash
$ cmmv create nome-do-projeto
```

Isso iniciará um prompt interativo perguntando sobre as preferências do seu projeto, como:

* Ativar Middleware Vite
* Usar RPC (WebSocket + Protobuf)
* Ativar o módulo de Cache
* Escolher o tipo de repositório (SQLite, MongoDB, PostgreSQL, MySQL, MSSQL, Oracle)
* Escolher o tipo de cache (Redis, Memcached, MongoDB, Sistema de Arquivos)
* Escolher o tipo de fila (Redis, RabbitMQ, Kafka)
* Ativar ESLint, Prettier e Vitest

## Utilização

Se não quiser instalar o CLI globalmente, use `pnpm dlx` para executá-lo diretamente:

```bash
$ pnpm dlx @cmmv/cli@latest create nome-do-projeto
```

Isso garante que você sempre use a versão mais recente do CLI sem necessidade de instalação global.

## Mudanças na Versão 5.9

O CMMV CLI foi refatorado, introduzindo novos comandos e otimizando fluxos de trabalho. Recomenda-se a atualização a partir de versões anteriores.

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso</p>
    <p>
        A partir da versão <strong>5.9</strong>, o <strong>@cmmv/cli</strong> foi totalmente refatorado, introduzindo novos comandos e otimizando fluxos de trabalho.
        Recomenda-se fortemente a atualização para essa versão para melhor desempenho e usabilidade.
    </p>
    <p>
        O CLI agora inclui suporte nativo para <strong>ESLint</strong>, <strong>release de módulos</strong> e um <strong>sistema de build aprimorado</strong> com suporte a <strong>ESM</strong> e <strong>CJS</strong>.
        Além disso, <strong>hot reload</strong> para desenvolvimento e um novo <strong>comando run</strong> para execução de scripts foram adicionados.
    </p>
</div>

### Novos Recursos

* **ESLint integrado**: Sem necessidade de dependências separadas—execute `$ cmmv lint`.
* **Automação de release de módulos**: `$ cmmv release` agora gerencia todas as dependências automaticamente.
* **Sistema de Build Aprimorado**: Suporte a builds ESM e CJS: `$ cmmv build`.
* **Modo de Desenvolvimento Aprimorado**:
    * Watch e debug integrados, sem necessidade de `nodemon`.
    * Suporte a hot reload com `$ cmmv dev`.
    * Exemplo de configuração:

```json
"dev": {
    "watch": ["src", "docs"],
    "ignore": ["**/*.spec.ts", "src/app.module.ts", "docs/**/*.html"]
}
```

* **Comando de início para produção**: `$ cmmv start`.
* **Execução de scripts**: `$ cmmv run ./src/<script>.ts`.
* **Melhoria de desempenho**:
    * `@swc-node/register` substitui `ts-node` para `dev` e `run`.
    * `tsc` continua sendo o padrão para builds de produção.

* **Configuração do ESLint**:
    * Usa ESLint 9.20 com Prettier.
    * Configurável via `eslint.config.cjs` (ou `eslint.config.ts` para projetos ESM).

## Estrutura do Projeto Gerado

O CLI gera uma estrutura de pastas organizada com arquivos e diretórios necessários, conforme suas preferências. Abaixo está um exemplo:

```
.
├── .generated/
├── public/
│   ├── assets/
│   │   └── protobuf.min.js (se RPC estiver ativado)
│   ├── core/
│   ├── templates/
│   └── views/
├── src/
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── tests/
│   ├── app.controller.spec.ts
│   ├── app.module.spec.ts
│   ├── app.service.spec.ts
├── tsconfig.json
├── eslint.config.cjs
├── .cmmv.config.cjs
├── package.json
├── .gitignore
└── ...
```

## Comandos Disponíveis

Modo de Desenvolvimento:

```bash
$ pnpm dev
```

Build para Produção:

```bash
$ pnpm build
```

Iniciar Servidor de Produção:

```bash
$ pnpm start
```

Executar Testes (se Vitest estiver ativado):

```bash
$ pnpm test
```

Executar ESLint:

```bash
$ pnpm lint
```

## Criando Módulos

O CMMV CLI agora inclui o comando `module` para facilitar a criação de novos módulos dentro de um projeto CMMV existente.

```bash
$ cmmv module <nome-do-módulo>
```

## Criando Contratos

O comando `cmmv contract` permite criar contratos no framework CMMV, definindo estrutura, validação e metadados para suas entidades e controladores.

### Uso

```bash
$ cmmv contract <nome-do-contrato>
```

Isso iniciará um prompt interativo para configurar seu contrato, permitindo definir nome, metadados, campos e regras de validação.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
    cache: { key: 'task:', ttl: 300, compress: true },
})
export class TasksContract extends AbstractContract {
    @ContractField({ protoType: 'string', unique: true })
    label: string;

    @ContractField({ protoType: 'bool', defaultValue: false })
    checked: boolean;

    @ContractField({ protoType: 'bool', defaultValue: false })
    removed: boolean;

    @ContractField({ protoType: 'date' })
    createdAt?: Date;
}
```

Este contrato pode ser registrado no `Application.create`:

```typescript
Application.create({
    contracts: [TasksContract]
});
```
