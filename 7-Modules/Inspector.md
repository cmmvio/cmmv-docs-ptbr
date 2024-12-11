# Inspector

O módulo ``@cmmv/inspector`` fornece ferramentas para análise de desempenho em tempo de execução e depuração de aplicações Node.js. Ele se integra perfeitamente com projetos baseados em CMMV e utiliza o módulo integrado ``node:inspector`` para capturar perfis de CPU e snapshots de heap. O módulo também oferece métodos utilitários para gerenciar e persistir dados de perfil, tornando-se uma ferramenta essencial para otimizar e depurar suas aplicações.

## Instalação

Para instalar o módulo ``@cmmv/inspector``, use o seguinte comando:

```bash
$ pnpm add @cmmv/inspector
```

## Recursos

* **Perfil de CPU:** Inicie e pare perfis de CPU para capturar dados de desempenho.
* **Snapshots de Heap:** Tire e salve snapshots de heap para analisar o uso de memória e detectar vazamentos.
* **Manipulação de Sinais de Processo:** Lida automaticamente com a limpeza durante a finalização do processo.
* **Hooks de Finalização Personalizados:** Registre tarefas de limpeza com ``Inspector.once``.
* **Persistência de Dados:** Salve dados de perfil para análise com ferramentas compatíveis, como o Chrome DevTools.

## Exemplos

Você pode iniciar e parar o perfil de CPU para capturar métricas de desempenho durante operações específicas.

```typescript
import { Inspector } from '@cmmv/inspector';

async function runProfiler() {
    await Inspector.start();

    // Realize algumas operações
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('Perfil de CPU salvo!');
}

runProfiler();
```

## Vinculando ao Processo 

Garanta a limpeza adequada durante a finalização do processo vinculando sinais de kill.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.bindKillProcess();

// Realize operações
```

## Finalização Personalizada

Registre tarefas de limpeza para executar antes da finalização do processo.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.once(async () => {
    console.log('Executando limpeza: Salvando snapshot de heap...');
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Snapshot de heap salvo!');
});

// Realize operações
```

## Snapshot de Heap

Tire e salve um snapshot de heap para analisar o uso de memória.

```typescript
import { Inspector } from '@cmmv/inspector';

async function captureHeapSnapshot() {
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Snapshot de heap salvo!');
}

captureHeapSnapshot();
```

## Referência da API

### ``Inspector.start(): Promise<void>``

Inicia o profiler de CPU.

```typescript
await Inspector.start();
```

### ``Inspector.stop(): Promise<void>``

Para o profiler de CPU e desconecta a sessão.

```typescript
await Inspector.stop();
```

### ``Inspector.saveProfile(dirPath: string, restart: boolean = true): Promise<void>``

Salva o perfil de CPU no diretório especificado. Opcionalmente reinicia o profiler após salvar.

```typescript
await Inspector.saveProfile('./profiles');
```

### ``Inspector.takeHeapSnapshot(dirPath: string): Promise<void>``

Tira um snapshot de heap e salva no diretório especificado para análise de memória.

```typescript
await Inspector.takeHeapSnapshot('./snapshots');
```

### ``Inspector.bindKillProcess(): void``

Vincula sinais de término de processo para garantir a finalização adequada das tarefas de perfil.

```typescript
Inspector.bindKillProcess();
```

### ``Inspector.once(callback: () => Promise<void>): void``

Registra um hook de finalização único para executar antes da finalização do processo.

```typescript
Inspector.once(async () => {
    console.log('Executando limpeza...');
});
```

## Fluxo de Trabalho 

O fluxo de trabalho a seguir demonstra como usar o módulo ``@cmmv/inspector`` para iniciar o perfil, capturar um snapshot de heap e parar o perfil durante a finalização do processo.

```typescript
import { Inspector } from '@cmmv/inspector';

async function main() {
    // Registre a tarefa de limpeza
    Inspector.once(async () => {
        console.log('Executando limpeza: Salvando snapshot de heap...');
        await Inspector.takeHeapSnapshot('./snapshots');
        console.log('Snapshot de heap salvo!');
    });

    // Vincule sinais de processo
    Inspector.bindKillProcess();

    // Inicie o profiler
    await Inspector.start();

    // Realize operações
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    // Pare o profiler e salve o perfil de CPU
    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('Perfil salvo!');
}

main();
```

O módulo ``@cmmv/inspector`` é uma ferramenta indispensável para depuração e ajuste de desempenho em projetos baseados em CMMV, fornecendo uma interface poderosa e simplificada para capturar e gerenciar dados de perfil.
