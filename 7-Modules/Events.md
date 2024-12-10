# Events

The `@cmmv/events` module provides a robust and flexible event-driven system for managing asynchronous communication within your application. Leveraging `eventemitter2`, this module enables developers to create, emit, and listen to events seamlessly, enhancing the modularity and scalability of the application.

## Features

- **Event Binding:** Easily bind methods to events using the `@OnEvent` decorator.
- **Asynchronous Communication:** Facilitate inter-service communication with minimal coupling.
- **Payload Flexibility:** Supports any payload type, with recommended use of TypeScript interfaces for structured data.
- **Integration with CMMV Framework:** Built-in compatibility with `@cmmv/core` modules and services.
- **Error Handling:** Ensures robust error logging and debugging for event-driven workflows.

## Installation

To install the `@cmmv/events` module:

```bash
$ pnpm add @cmmv/events eventemitter2
```

## Configuration

No additional configuration is required. The `@cmmv/events` module works out of the box with your CMMV application.

## Setting Up the Application

In your `index.ts`, include the `EventsModule` along with any modules or services that use event-based communication:

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { EventsModule, EventsService } from '@cmmv/events';

import { ListernersModule } from './listeners.module';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        EventsModule,
        ListernersModule,
    ],
    services: [EventsService],
});
```

## Usage

### Creating Event Listeners

Use the `@OnEvent` decorator to bind a method to an event. Event listeners can be added to any service, controller, or gateway in your application.

```typescript
import { Service } from '@cmmv/core';
import { OnEvent } from '@cmmv/events';
import { EventsService } from '@cmmv/events';

@Service('listener')
export class Listener {
    constructor(private readonly eventsService: EventsService) {}

    @OnEvent('hello-world')
    public async OnReceiveMessage(payload: any) {
        console.log('hello-world event received:', payload);
    }
}
```

### Emitting Events

To emit an event, inject `EventsService` into your class and call the `emit` method:

```typescript
this.eventsService.emit('event-name', { key: 'value' });
```

<br/>

- **First Parameter:** Name of the event.
- **Second Parameter:** Payload for the event (any type). Recommended to use a TypeScript interface for the payload structure.

### Example Event Emission

```typescript
this.eventsService.emit('user.created', { id: 1, name: 'John Doe' });
```

## Decorators

### `@OnEvent(eventName: string)`
Binds a method to listen for a specific event.

## Best Practices

- **Use Interfaces for Payloads:** Define TypeScript interfaces for event payloads to ensure consistency and type safety.
- **Group Related Events:** Use event namespaces or prefixes to group related events (e.g., `user.created`, `user.updated`).

## Example Application

Here is an example of an application utilizing the `@cmmv/events` module:

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { EventsModule } from '@cmmv/events';

import { UserModule } from './user.module';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        EventsModule,
        UserModule,
    ],
});
```

### User Module

```typescript
import { Module } from '@cmmv/core';
import { UserListener } from './listeners/user.listener';

export const UserModule = new Module('user', {
    providers: [UserListener],
});
```

### User Listener

```typescript
import { Service } from '@cmmv/core';
import { OnEvent } from '@cmmv/events';

@Service('user-listener')
export class UserListener {
    @OnEvent('user.created')
    public handleUserCreated(payload: { id: number; name: string }) {
        console.log(`User created:`, payload);
    }
}
```

### Emitting Events in a Controller

```typescript
import { Controller } from '@cmmv/core';
import { EventsService } from '@cmmv/events';

@Controller('user')
export class UserController {
    constructor(private readonly eventsService: EventsService) {}

    public async createUser() {
        // Perform user creation logic...

        // Emit user.created event
        this.eventsService.emit('user.created', { id: 1, name: 'John Doe' });
    }
}
```

## Advantages

- **Decoupled Logic:** Enables better separation of concerns by decoupling event emitters and listeners.
- **Scalable Communication:** Ideal for applications with complex workflows requiring event-based communication.
- **Seamless Integration:** Works out of the box with the CMMV framework, simplifying setup and usage.