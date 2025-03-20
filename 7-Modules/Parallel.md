# Paralelo

Repositório: [https://github.com/cmmvio/cmmv-parallel](https://github.com/cmmvio/cmmv-parallel)

O módulo `@cmmv/parallel` introduz paralelismo no framework CMMV, permitindo o processamento eficiente de dados usando threads baseadas no `fast-thread`. Essa implementação utiliza `SharedArrayBuffer`, `Atomics` e, opcionalmente, `fast-json-stringify` para transferência de dados sem cópia entre threads, tornando-o significativamente mais rápido que a abordagem tradicional `parentPort.postMessage`.

## Instalação

Para instalar o módulo ``@cmmv/parallel``, execute o seguinte comando:

```typescript
$ pnpm add @cmmv/parallel
```

## Como Funciona

Diferente do multi-threading tradicional em JavaScript, o `@cmmv/parallel` cria um contexto de execução isolado dentro de cada thread de trabalho. Isso permite que cálculos complexos sejam executados sem bloquear a thread principal, tornando-o ideal para processamento de dados em grande escala.

| **Recurso**           | **Tradicional `worker_threads`** | **`@cmmv/parallel`** |
|-----------------------|----------------------------------|----------------------|
| **Transferência de Dados** | Serialização JSON (lenta)   | SharedArrayBuffer (sem cópia) |
| **Carregamento de Contexto** | Requer importações manuais | Injeção automática de contexto |
| **Gerenciamento de Threads** | Criação manual de workers | Pool de threads com escalonamento dinâmico |
| **Comunicação**       | `parentPort.postMessage`    | Acesso direto à memória via Atomics |

## Benchmarks

* [https://github.com/andrehrferreira/fast-thread/blob/main/benchmarks/index.js](https://github.com/andrehrferreira/fast-thread/blob/main/benchmarks/index.js)
* Máquina: Linux x64 | 32 vCPUs | 128.0GB de Memória
* Node: v20.17.0

| Nome                     | Mensagens | Mensagens por Segundo | MB por Segundo |
|--------------------------|-----------|-----------------------|----------------|
| **fast-thread**          | 617.483   | 61.748,30             | 65,34          |
| **JSON**                 | 524.235   | 52.423,50             | 55,48          |
| **fast-json-stringify**  | 500.024   | 50.002,40             | 52,10          |
| **BSON**                 | 420.946   | 42.094,60             | 44,19          |
| **Protobuf.js**          | 296.340   | 29.634,00             | 29,75          |
| **msgpack-lite**         | 288.180   | 28.818,00             | 29,86          |
| **CBOR**                 | 223.945   | 22.394,50             | 23,20          |

## Decorador

O módulo `@cmmv/parallel` introduz um conjunto de decoradores que simplificam a execução paralela ao automatizar o gerenciamento de threads, a transferência de dados e a inicialização do contexto. Esses decoradores fornecem uma maneira intuitiva de definir tarefas paralelas sem lidar manualmente com threads de trabalho, serialização e passagem de mensagens.

## @Parallel

Marca uma função para ser executada em paralelo usando um pool de threads de trabalho.

* Gerencia automaticamente as threads de trabalho e sincroniza os resultados.
* Usa um namespace para agrupar tarefas paralelas relacionadas.
* Número de threads configurável, permitindo escalonamento dinâmico.

```typescript
@Parallel({
    namespace: "parserLine",
    threads: 3
})
async parserLine(@Tread() thread: any, @ThreadData() payload: any) {
    return {
        data: await thread.jsonParser.parser(payload.value),
        index: payload.index
    }
}
```

## @ThreadContext

Define um contexto de execução compartilhado para uma função paralela.

* Carrega dependências e recursos dentro da thread de trabalho.
* Garante que todos os trabalhadores em um pool compartilhem o mesmo contexto.
* Retorna um objeto acessível via `@Tread()`.

```typescript
@TreadContext("parserLine")
async threadContext() {
    const { JSONParser, AbstractParserSchema, ToLowerCase, ToDate } = await import("@cmmv/normalizer");

    class CustomerSchema extends AbstractParserSchema {
        public field = {
            name: { to: 'name' },
            email: {
                to: 'email',
                transform: [ToLowerCase],
            },
            registrationDate: {
                to: 'createdAt',
                transform: [ToDate],
            },
        };
    }

    const jsonParser = new JSONParser({ schema: CustomerSchema });
    return { jsonParser };
}
```

## @ThreadData

Extrai o payload de dados enviado para a thread de trabalho.

* Torna a assinatura da função limpa e legível.
* Injeta apenas os dados relevantes para o processamento.
* Funciona junto com `@Tread()` para acessar tanto os dados de entrada quanto o contexto compartilhado.

```typescript
async parserLine(@Tread() thread: any, @ThreadData() payload: any) {
    return {
        data: await thread.jsonParser.parser(payload.value),
        index: payload.index
    }
}
```

## @Tread

Fornece acesso ao contexto compartilhado da thread, conforme definido por `@TreadContext()`.

* Concede acesso a recursos pré-carregados dentro do worker.
* Garante processamento eficiente de dados sem inicialização redundante.
* Trabalha em conjunto com `@ThreadData()` para uma execução fluida da função.

```typescript
async parserLine(@Tread() thread: any, @ThreadData() payload: any) {
    return {
        data: await thread.jsonParser.parser(payload.value),
        index: payload.index
    }
}
```

Ao usar esses decoradores, os desenvolvedores podem eliminar código boilerplate, alcançar compartilhamento de memória sem cópia e processar eficientemente grandes volumes de dados em paralelo. 🚀

## Integração

Este exemplo demonstra como o `@cmmv/parallel` pode processar eficientemente arquivos JSON grandes usando múltiplas threads.

```typescript
import * as fs from 'node:fs';
import * as path from 'node:path';
import { parser } from 'stream-json';
import { streamArray } from 'stream-json/streamers/StreamArray';

import { Application, Hook, HooksType } from "@cmmv/core";

import {
    AbstractParallel, Parallel, Thread,
    ThreadData, ThreadPool, TreadContext
} from "@cmmv/parallel";

export class ReadBigFileWithParallel extends AbstractParallel {
    @Hook(HooksType.onInitialize)
    async start() {
        const finalData = new Array<any>();
        const poolNamespace = "parserLine";
        const pool = ThreadPool.getThreadPool(poolNamespace);
        const filename = path.resolve('./sample/large-customers.json');

        if (pool) {
            console.log('Processando com Multi-Thread...');
            let start;
            const readStream = fs.createReadStream(filename);
            await pool.awaitStart();
            const jsonStream = readStream.pipe(parser()).pipe(streamArray());

            pool.on('message', async (response) => {
                finalData[response.index] = response.data;
            });

            pool.on('end', () => {
                const end = Date.now();
                console.log(`Parser paralelo: \${finalData.length} | \${(end - start).toFixed(2)}s`);
            });

            jsonStream.on('data', async ({ value, key }) => {
                if (!start) start = Date.now();
                pool.send({ value, index: key });
            });

            jsonStream.on('end', () => pool.endData());
            jsonStream.on('error', error => console.error(error));

            await pool.awaitEnd();
        } else {
            throw new Error(`Pool de threads '\${poolNamespace}' não encontrado`);
        }
    }

    @Parallel({
        namespace: "parserLine",
        threads: 3
    })
    async parserLine(@Thread() thread: any, @ThreadData() payload: any) {
        return {
            data: await thread.jsonParser.parser(payload.value),
            index: payload.index
        }
    }

    @TreadContext("parserLine")
    async threadContext() {
        const { JSONParser, AbstractParserSchema, ToLowerCase, ToDate } = await import("@cmmv/normalizer");

        class CustomerSchema extends AbstractParserSchema {
            public field = {
                id: { to: 'id' },
                name: { to: 'name' },
                email: {
                    to: 'email',
                    validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
                    transform: [ToLowerCase],
                },
                registrationDate: {
                    to: 'createdAt',
                    transform: [ToDate],
                },
            };
        }

        const jsonParser = new JSONParser({ schema: CustomerSchema });
        return { jsonParser };
    }
}

Application.exec({
    modules: [ParallelModule],
    services: [ReadBigFileWithParallel]
});
```
<br/>

* Processamento multi-thread – Distribui tarefas eficientemente entre vários núcleos de CPU.
* Comunicação sem cópia – Usa SharedArrayBuffer para evitar duplicação de memória.
* Threads conscientes do contexto – Carrega recursos específicos dentro de cada thread.
* Serialização rápida – Suporta fast-json-stringify para otimização de desempenho.
* API simplificada – Não há necessidade de criar arquivos separados para threads de trabalho.
