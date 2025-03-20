# Compile

A partir da **versão 8.5**, o CMMV introduz a função `compile`, que permite processar todos os módulos e gerar arquivos derivados a partir dos transpiladores que, anteriormente, só eram criados ao iniciar a aplicação.

Essa funcionalidade é especialmente útil em cenários onde você deseja **pré-compilar** todos os arquivos necessários sem realmente executar uma aplicação HTTP ou RPC.

## Quando Usar

Você deve usar `Application.compile` quando:

- Precisar **gerar arquivos transpilados** antecipadamente, sem iniciar a aplicação.
- Estiver preparando um pipeline de deploy e quiser pré-processar contratos e arquivos relacionados.
- Quiser garantir que todas as configurações e módulos sejam corretamente processados antes da execução da aplicação.

## Como Usar

Em vez de usar `Application.create` para iniciar a aplicação, basta substituí-lo por `Application.compile`. Abaixo está um exemplo de uso:

```typescript
import { Application } from '@cmmv/core';
import { DefaultHTTPModule } from '@cmmv/http';
import { ProtobufModule } from '@cmmv/protobuf';
import { WSModule } from '@cmmv/ws';
import { RepositoryModule, Repository } from '@cmmv/repository';
import { AuthModule } from '@cmmv/auth';

// Contratos
import { TasksContract } from './contracts/tasks.contract';

Application.compile({
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule,
        RepositoryModule,
        AuthModule,
    ],
    services: [Repository],
    contracts: [TasksContract],
});
```

### Explicação

- **modules:** Uma lista de módulos da aplicação, como HTTP, WebSockets, Protobuf, repositório, cache e autenticação.
- **services:** Os serviços principais usados dentro da aplicação, como `Repository`.
- **contracts:** Contratos que definem a lógica de negócio da aplicação, como `TasksContract`.

## Script Adicionado

A partir da versão **8.5**, um script foi adicionado para simplificar o processo de compilação, executando etapas de limpeza necessárias e inicializando a compilação.

Para compilar sem executar um servidor HTTP ou RPC real, execute o seguinte comando:

```bash
./tools/cleanupPackages.sh &&
    pnpm clean &&
    NODE_ENV=dev node -r @swc-node/register ./src/compile.ts
```

<br/>

1. **`./tools/cleanupPackages.sh`** – Limpa dependências de pacotes não utilizadas ou desatualizadas.
2. **`pnpm clean`** – Remove arquivos temporários e diretórios de build.
3. **`NODE_ENV=dev`** – Define o ambiente para modo desenvolvimento.
4. **`node -r @swc-node/register ./src/compile.ts`** – Compila a aplicação com SWC para execução mais rápida.

## Execução

Para iniciar o processo de compilação, você pode executar o seguinte comando:

```bash
$ pnpm run compile
```

Este comando garante que todos os arquivos necessários estejam preparados e prontos para produção ou para uma verificação manual posterior.
