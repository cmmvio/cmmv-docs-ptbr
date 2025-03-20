# Fila

Repositório: [https://github.com/cmmvio/cmmv-queue](https://github.com/cmmvio/cmmv-queue)

O módulo ``@cmmv/queue`` oferece uma interface poderosa e unificada para gerenciar filas de mensagens em aplicações Node.js construídas com o framework ``@cmmv/core``. Ele suporta RabbitMQ, Kafka e Redis como backends de fila, permitindo que os desenvolvedores definam produtores e consumidores para o processamento de mensagens de forma estruturada e modular. O módulo simplifica a integração de uma arquitetura orientada a filas, facilitando a construção de sistemas escaláveis e orientados a eventos.

## Instalação

Instale o pacote ``@cmmv/queue`` via npm:

```bash
$ pnpm add @cmmv/queue
```

### Backends de Fila Suportados

RabbitMQ ([amqp-connection-manager](https://www.npmjs.com/package/amqp-connection-manager)):

```bash
$ pnpm add amqp-connection-manager
```

Kafka ([kafkajs](https://www.npmjs.com/package/kafkajs)):

```bash
$ pnpm add kafkajs
```

Redis ([ioredis](https://www.npmjs.com/package/ioredis)):

```bash
$ pnpm add ioredis
```

## Configuração

O módulo ``@cmmv/queue`` requer um arquivo de configuração (``.cmmv.config.cjs``) para definir o tipo de backend da fila e os detalhes de conexão.

```javascript
module.exports = {
    queue: {
        type: process.env.QUEUE_TYPE || "rabbitmq", // "rabbitmq" | "kafka" | "redis"
        url: process.env.QUEUE_URL || "amqp://guest:guest@localhost:5672/cmmv",
    },
};
```

## Recursos

* **Suporte a Múltiplos Backends de Fila:** Funciona perfeitamente com RabbitMQ, Kafka e Redis.
* **Pub/Sub:** Suporta padrões de publicação/assinatura para arquitetura orientada a eventos.
* **Gerenciamento de Consumidores:** Define e registra consumidores usando decoradores.
* **Suporte a Produtores:** Publica e envia mensagens para filas com facilidade.
* **Integração com o Framework CMMV:** Integra-se facilmente a aplicações CMMV.

## Primeiros Passos

Use os decoradores ``@Channel`` e ``@Consume`` para definir consumidores de mensagens. Abaixo está um exemplo de consumidor para RabbitMQ:

```typescript
import {
    Channel, Consume,
    QueueMessage, QueueConn,
    QueueChannel
} from "@cmmv/queue";

import { QueueService } from "../services";

@Channel("hello-world")
export class HelloWorldConsumer {
    constructor(private readonly queueService: QueueService) {}

    @Consume("hello-world")
    public async onReceiveMessage(
        @QueueMessage() message,
        @QueueChannel() channel,
        @QueueConn() conn
    ){
        console.log("Mensagem recebida:", message);
        this.queueService.send("hello-world", "niceday", "Tenha um bom dia!");
    }

    @Consume("niceday")
    public async onReceiveNiceDayMessage(@QueueMessage() message){
        console.log("Tenha um bom dia!");
    }
}
```

## Exemplo de Pub/Sub

Para habilitar o Pub/Sub, use a opção ``pubSub`` no decorador ``@Channel``.

```typescript
import {
    Channel, Consume,
    QueueMessage
} from "@cmmv/queue";

import { QueueService } from "../services";

@Channel("broadcast", {
    exchangeName: "broadcast",
    pubSub: true
})
export class BroadcastConsumer {
    constructor(private readonly queueService: QueueService) {}

    @Consume("broadcast")
    public async onBroadcastMessage(@QueueMessage() message) {
        console.log("Mensagem de broadcast recebida:", message);
    }
}
```

## Registro de Consumidores

Os consumidores são registrados em um módulo dedicado:

```typescript
import { Module } from '@cmmv/core';

import { HelloWorldConsumer } from './consumers/hello-world.consumer';
import { BroadcastConsumer } from './consumers/broadcast.consumer';

export let ConsumersModule = new Module("consumers", {
    providers: [HelloWorldConsumer, BroadcastConsumer],
});
```

## Iniciando a Aplicação

Integre o módulo ``@cmmv/queue`` e seus módulos de consumidores na configuração da aplicação.

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { QueueModule, QueueService } from "@cmmv/queue";

import { ConsumersModule } from "./consumers.module";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        QueueModule,
        ConsumersModule
    ],
    services: [QueueService],
});
```

## Enviando Mensagens

As mensagens podem ser enviadas para uma fila específica usando o ``QueueService``:

```typescript
import { QueueService } from "@cmmv/queue";

// Enviando uma mensagem direta
QueueService.send("hello-world", "niceday", { message: "Aproveite seu dia!" });

// Publicando uma mensagem (Pub/Sub)
QueueService.publish("broadcast", "exchangeName", { event: "system_update" });
```

## Decoradores

### ``@Channel(queueName: string, options?: ChannelOptions)``
Define uma fila/canal para uma classe consumidora.

Opções:

| Opção          | Tipo      | Descrição                                            | Padrão          |
|----------------|-----------|------------------------------------------------------|-----------------|
| `pubSub`       | boolean   | Habilita mensagens Pub/Sub.                          | `false`         |
| `exchangeName` | string    | Define o nome do exchange para roteamento de mensagens. | `"exchange"`    |
| `exclusive`    | boolean   | Cria uma fila exclusiva.                             | `false`         |
| `autoDelete`   | boolean   | Exclui a fila quando não há consumidores.            | `false`         |
| `durable`      | boolean   | Torna a fila durável (sobrevive a reinícios do broker). | `true`          |

### ``@Consume(message: string)``
Registra um método para lidar com mensagens da fila especificada.

**Decoradores de Parâmetro**
* **@QueueMessage():** Injeta o payload da mensagem recebida.
* **@QueueChannel():** Injeta o canal para a fila.
* **@QueueConn():** Injeta a instância de conexão.

O módulo ``@cmmv/queue`` permite configurar opções avançadas como padrões Pub/Sub, filas duráveis e consumidores exclusivos. Use as opções no decorador ``@Channel`` para personalizar sua configuração de fila conforme os requisitos da sua aplicação.

Com suporte a múltiplos backends e integração perfeita com o framework ``@cmmv/core``, o módulo ``@cmmv/queue`` simplifica a construção de sistemas escaláveis e modulares orientados a eventos.
