# Eventos

O módulo ``@cmmv/events`` fornece um sistema robusto e flexível para comunicação assíncrona em sua aplicação. Baseado no ``eventemitter2``, este módulo permite que os desenvolvedores criem, emitam e ouçam eventos de maneira simples, aprimorando a modularidade e escalabilidade da aplicação.

## Recursos

- **Vinculação de Eventos:** Vincule facilmente métodos a eventos usando o decorador ``@OnEvent``.
- **Comunicação Assíncrona:** Facilita a comunicação entre serviços com acoplamento mínimo.
- **Flexibilidade de Payload:** Suporta qualquer tipo de payload, com recomendação de uso de interfaces TypeScript para dados estruturados.
- **Integração com o Framework CMMV:** Compatibilidade integrada com módulos e serviços do ``@cmmv/core``.
- **Tratamento de Erros:** Garante registro robusto de erros e depuração para fluxos de trabalho orientados a eventos.

## Instalação

Para instalar o módulo ``@cmmv/events``:

```bash
$ pnpm add @cmmv/events
```

## Configuração

Não é necessária configuração adicional. O módulo ``@cmmv/events`` funciona imediatamente em sua aplicação CMMV.

## Configurando a Aplicação

No seu ``index.ts``, inclua o ``EventsModule`` juntamente com quaisquer módulos ou serviços que utilizem comunicação baseada em eventos:

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

## Uso

### Criando Listeners de Evento

Use o decorador ``@OnEvent`` para vincular um método a um evento. Os listeners podem ser adicionados a qualquer serviço, controlador ou gateway em sua aplicação.

```typescript
import { Service } from '@cmmv/core';
import { OnEvent } from '@cmmv/events';
import { EventsService } from '@cmmv/events';

@Service('listener')
export class Listener {
    constructor(private readonly eventsService: EventsService) {}

    @OnEvent('hello-world')
    public async OnReceiveMessage(payload: any) {
        console.log('Evento hello-world recebido:', payload);
    }
}
```

### Emitindo Eventos

Para emitir um evento, injete ``EventsService`` em sua classe e chame o método ``emit``:

```typescript
this.eventsService.emit('event-name', { key: 'value' });
```

<br/>

- **Primeiro Parâmetro:** Nome do evento.
- **Segundo Parâmetro:** Payload do evento (qualquer tipo). Recomenda-se usar uma interface TypeScript para a estrutura do payload.

### Exemplo de Emissão de Evento

```typescript
this.eventsService.emit('user.created', { id: 1, name: 'John Doe' });
```

## Decoradores

### ``@OnEvent(eventName: string)``
Vincula um método para ouvir um evento específico.

## Boas Práticas

- **Use Interfaces para Payloads:** Defina interfaces TypeScript para payloads de eventos para garantir consistência e segurança de tipo.
- **Agrupe Eventos Relacionados:** Use namespaces ou prefixos para agrupar eventos relacionados (ex.: ``user.created``, ``user.updated``).

## Exemplo de Aplicação

Aqui está um exemplo de aplicação utilizando o módulo ``@cmmv/events``:

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

### Módulo de Usuário

```typescript
import { Module } from '@cmmv/core';
import { UserListener } from './listeners/user.listener';

export const UserModule = new Module('user', {
    providers: [UserListener],
});
```

### Listener de Usuário

```typescript
import { Service } from '@cmmv/core';
import { OnEvent } from '@cmmv/events';

@Service('user-listener')
export class UserListener {
    @OnEvent('user.created')
    public handleUserCreated(payload: { id: number; name: string }) {
        console.log(`Usuário criado:`, payload);
    }
}
```

### Emitindo Eventos em um Controlador

```typescript
import { Controller } from '@cmmv/core';
import { EventsService } from '@cmmv/events';

@Controller('user')
export class UserController {
    constructor(private readonly eventsService: EventsService) {}

    public async createUser() {
        // Realize a lógica de criação de usuário...

        // Emita o evento user.created
        this.eventsService.emit('user.created', { id: 1, name: 'John Doe' });
    }
}
```

## Vantagens

- **Lógica Desacoplada:** Permite uma melhor separação de responsabilidades ao desacoplar emissores e ouvintes de eventos.
- **Comunicação Escalável:** Ideal para aplicações com fluxos de trabalho complexos que requerem comunicação baseada em eventos.
- **Integração Transparente:** Funciona imediatamente com o framework CMMV, simplificando a configuração e o uso.
