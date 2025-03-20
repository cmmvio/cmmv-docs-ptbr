# Inspetor

Repositório: [https://github.com/cmmvio/cmmv-inspector](https://github.com/cmmvio/cmmv-inspector)

O módulo ``@cmmv/inspector`` fornece ferramentas para perfilamento de desempenho em tempo de execução e depuração de aplicações Node.js. Ele se integra perfeitamente a projetos baseados em CMMV e utiliza o módulo embutido `node:inspector` para capturar perfis de CPU e instantâneos de heap. O módulo também oferece métodos utilitários para gerenciar e persistir dados de perfilamento, tornando-o uma ferramenta essencial para otimizar e depurar suas aplicações.

## Instalação

Para instalar o módulo ``@cmmv/inspector``, use o seguinte comando:

```bash
$ pnpm add @cmmv/inspector
```

## Recursos

* **Perfilamento de CPU:** Inicia e para o perfilamento de CPU para capturar dados de desempenho.
* **Instantâneos de Heap:** Realiza e salva instantâneos de heap para analisar o uso de memória e detectar vazamentos.
* **Tratamento de Sinais do Processo:** Gerencia automaticamente a limpeza durante a terminação do processo.
* **Ganchos de Finalização Personalizados:** Registra tarefas de limpeza com `Inspector.once`.
* **Persistência de Dados:** Salva dados de perfilamento para análise com ferramentas compatíveis, como o Chrome DevTools.

## Exemplos

Você pode iniciar e parar o perfilamento de CPU para capturar métricas de desempenho durante operações específicas.

```typescript
import { Inspector } from '@cmmv/inspector';

async function runProfiler() {
    await Inspector.start();

    // Realiza algumas operações
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('Perfil de CPU salvo!');
}

runProfiler();
```

## Vincular Processo

Garante a limpeza adequada durante a terminação do processo ao vincular sinais de término.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.bindKillProcess();

// Realiza operações
```

## Finalização Personalizada

Registra tarefas de limpeza para serem executadas antes que o processo termine.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.once(async () => {
    console.log('Realizando limpeza: Salvando instantâneo de heap...');
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Instantâneo de heap salvo!');
});

// Realiza operações
```

## Instantâneo de Heap

Realiza e salva um instantâneo de heap para analisar o uso de memória.

```typescript
import { Inspector } from '@cmmv/inspector';

async function captureHeapSnapshot() {
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Instantâneo de heap salvo!');
}

captureHeapSnapshot();
```

## Referência da API

### ``Inspector.start(): Promise<void>``

Inicia o perfilador de CPU.

```typescript
await Inspector.start();
```

### ``Inspector.stop(): Promise<void>``

Para o perfilador de CPU e desconecta a sessão.

```typescript
await Inspector.stop();
```

### ``Inspector.saveProfile(dirPath: string, restart: boolean = true): Promise<void>``

Salva o perfil de CPU no diretório especificado. Opcionalmente, reinicia o perfilador após salvar.

```typescript
await Inspector.saveProfile('./profiles');
```

### ``Inspector.takeHeapSnapshot(dirPath: string): Promise<void>``

Realiza um instantâneo de heap e o salva no diretório especificado para análise de memória.

```typescript
await Inspector.takeHeapSnapshot('./snapshots');
```

### ``Inspector.bindKillProcess(): void``

Vincula sinais de terminação do processo para garantir a finalização adequada das tarefas de perfilamento.

```typescript
Inspector.bindKillProcess();
```

### ``Inspector.once(callback: () => Promise<void>): void``

Registra um gancho de finalização único para ser executado antes da terminação do processo.

```typescript
Inspector.once(async () => {
    console.log('Realizando limpeza...');
});
```

## Fluxo de Trabalho

O fluxo de trabalho a seguir demonstra como usar o módulo ``@cmmv/inspector`` para iniciar o perfilamento, capturar um instantâneo de heap e parar o perfilamento durante a terminação do processo.

```typescript
import { Inspector } from '@cmmv/inspector';

async function main() {
    // Registra tarefa de limpeza
    Inspector.once(async () => {
        console.log('Realizando limpeza: Salvando instantâneo de heap...');
        await Inspector.takeHeapSnapshot('./snapshots');
        console.log('Instantâneo de heap salvo!');
    });

    // Vincula sinais do processo
    Inspector.bindKillProcess();

    // Inicia o perfilamento
    await Inspector.start();

    // Realiza operações
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    // Para o perfilamento e salva o perfil de CPU
    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('Perfil salvo!');
}

main();
```

O módulo ``@cmmv/inspector`` é uma ferramenta indispensável para depuração e ajuste de desempenho em projetos baseados em CMMV, fornecendo uma interface simplificada e poderosa para capturar e gerenciar dados de perfilamento.
