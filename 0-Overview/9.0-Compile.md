# Compilação

A partir da **versão 8.5**, o CMMV introduz a função `compile`, que permite processar todos os módulos e gerar arquivos derivados dos transpiladores que anteriormente eram criados apenas ao iniciar a aplicação.

Esse recurso é particularmente útil em cenários onde você deseja pré-compilar todos os arquivos necessários sem executar efetivamente uma aplicação HTTP ou RPC.

## Quando usar Application.compile

Você deve usar `Application.compile` quando:

- Precisar **gerar arquivos transpilados** antecipadamente, sem iniciar a aplicação.
- Estiver preparando um pipeline de implantação e quiser pré-processar contratos e arquivos relacionados.
- Desejar garantir que todas as configurações e módulos sejam processados corretamente antes de executar a aplicação.

## Como usar

Em vez de usar `Application.create` para iniciar a aplicação, basta substituí-lo por `Application.compile`. Abaixo está um exemplo de uso:

```typescript
import { Application } from '@cmmv/core';
import { DefaultHTTPModule } from '@cmmv/http';
import { ProtobufModule } from '@cmmv/protobuf';
import { WSModule } from '@cmmv/ws';
import { ViewModule } from '@cmmv/view';
import { RepositoryModule, Repository } from '@cmmv/repository';
import { CacheModule, CacheService } from '@cmmv/cache';
import { SchedulingModule, SchedulingService } from '@cmmv/scheduling';
import { AuthModule } from '@cmmv/auth';

// Contratos
import { TasksContract } from './contracts/tasks.contract';

Application.compile({
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule,
        ViewModule,
        RepositoryModule,
        CacheModule,
        SchedulingModule,
        AuthModule,
    ],
    services: [Repository, CacheService, SchedulingService],
    contracts: [TasksContract],
});

### Explicação

- **modules:** Uma lista de módulos da aplicação, como HTTP, WebSockets, Protobuf, repositório, cache e autenticação.
- **services:** Os serviços principais usados na aplicação, como `Repository`, `CacheService` e `SchedulingService`.
- **contracts:** Contratos que definem a lógica de negócios da aplicação, como `TasksContract`.

## Script Adicionado

A partir da versão **8.5**, um script foi adicionado para simplificar o processo de compilação, executando as etapas necessárias de limpeza e inicializando o processo de compilação.

Para compilar sem executar efetivamente um servidor HTTP ou RPC, execute o seguinte comando:

```bash
./tools/cleanupPackages.sh && 
    pnpm run clean && 
    set "VITE_CJS_TRACE=true" && 
    NODE_ENV=dev node -r @swc-node/register ./src/compile.ts
```

<br/>

1. **./tools/cleanupPackages.sh** – Limpa pacotes desatualizados ou não utilizados.
2. **pnpm run clean** – Remove arquivos temporários e diretórios de build.
3. **set "VITE_CJS_TRACE=true"** – Habilita rastreamento para fins de depuração.
4. **NODE_ENV=dev** – Define o ambiente para o modo de desenvolvimento.
5. **node -r @swc-node/register ./src/compile.ts** – Compila a aplicação com SWC para uma execução mais rápida.

## Executando

Para iniciar o processo de compilação, você pode executar o seguinte comando:

```bash
$ pnpm run compile
```

Este comando garante que todos os arquivos necessários estejam preparados e prontos para produção ou verificação manual posterior.