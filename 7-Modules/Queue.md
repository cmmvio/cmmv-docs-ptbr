# Queue 

The ``@cmmv/queue`` module provides a powerful and unified interface for managing message queues in Node.js applications built with the ``@cmmv/core`` framework. It supports RabbitMQ, Kafka, and Redis as queue backends, enabling developers to define producers and consumers for message processing in a structured and modular manner. The module simplifies the integration of queue-driven architecture, making it easy to build scalable, event-driven systems.

## Installation

Install the ``@cmmv/queue`` package via npm:

```bash
$ pnpm add @cmmv/queue
```

### Supported Queue Backends

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

## Configuration

The ``@cmmv/queue`` module requires a configuration file (``.cmmv.config.js``) to define the queue backend type and connection details.

```javascript
module.exports = {
    queue: {
        type: process.env.QUEUE_TYPE || "rabbitmq", // "rabbitmq" | "kafka" | "redis"
        url: process.env.QUEUE_URL || "amqp://guest:guest@localhost:5672/cmmv",
    },
};
```

## Features

* **Multi-Queue Backend Support:** Works seamlessly with RabbitMQ, Kafka, and Redis.
* **Pub/Sub:** Supports publish/subscribe patterns for event-driven architecture.
* **Consumer Management:** Define and register consumers using decorators.
* **Producer Support:** Publish and send messages to queues with ease.
* **Integration with CMMV Framework:** Easily integrates into CMMV applications.

## Getting Started

Use the ``@Channel`` and ``@Consume`` decorators to define message consumers. Below is an example consumer for RabbitMQ:

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
        console.log("Received message:", message);
        this.queueService.send("hello-world", "niceday", "Have a nice day!");
    }

    @Consume("niceday")
    public async onReceiveNiceDayMessage(@QueueMessage() message){
        console.log("Have a nice day!");
    }
}
```

## Pub/Sub Example

To enable Pub/Sub, use the ``pubSub`` option in the ``@Channel`` decorator.

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
        console.log("Broadcast message received:", message);
    }
}
```

## Register Consumers

Consumers are registered in a dedicated module:

```typescript
import { Module } from '@cmmv/core';

import { HelloWorldConsumer } from './consumers/hello-world.consumer';
import { BroadcastConsumer } from './consumers/broadcast.consumer';

export let ConsumersModule = new Module("consumers", {
    providers: [HelloWorldConsumer, BroadcastConsumer],
});
```

## Start the Application

Integrate the ``@cmmv/queue`` module and your consumer modules in the application setup.

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

## Sending Messages

Messages can be sent to a specific queue using the ``QueueService``:

```typescript
import { QueueService } from "@cmmv/queue";

// Sending a direct message
QueueService.send("hello-world", "niceday", { message: "Enjoy your day!" });

// Publishing a message (Pub/Sub)
QueueService.publish("broadcast", "exchangeName", { event: "system_update" });
```

## Decorators

### ``@Channel(queueName: string, options?: ChannelOptions)``
Defines a queue/channel for a consumer class.

Options:

| Option         | Type      | Description                                          | Default         |
|----------------|-----------|------------------------------------------------------|-----------------|
| `pubSub`       | boolean   | Enables Pub/Sub messaging.                           | `false`         |
| `exchangeName` | string    | Defines the exchange name for routing messages.      | `"exchange"`    |
| `exclusive`    | boolean   | Creates an exclusive queue.                         | `false`         |
| `autoDelete`   | boolean   | Deletes the queue when no consumers exist.           | `false`         |
| `durable`      | boolean   | Makes the queue durable (survives broker restarts).  | `true`          |


### ``@Consume(message: string)``
Registers a method to handle messages from the specified queue.

**Parameter Decorators**
* **@QueueMessage():** Injects the received message payload.
* **@QueueChannel():** Injects the channel for the queue.
* **@QueueConn():** Injects the connection instance.

The ``@cmmv/queue`` module allows you to configure advanced options like Pub/Sub patterns, durable queues, and exclusive consumers. Use the options in the ``@Channel`` decorator to customize your queue setup as per your application requirements.

With its multi-backend support and seamless integration with the ``@cmmv/core`` framework, the ``@cmmv/queue`` module simplifies building scalable and modular event-driven systems.