# Eventos

Repositório: [https://github.com/cmmvio/cmmv-events](https://github.com/cmmvio/cmmv-events)

O módulo `@cmmv/events` oferece um sistema robusto e flexível orientado a eventos para gerenciar comunicação assíncrona dentro da sua aplicação. Utilizando o `eventemitter2`, este módulo permite que os desenvolvedores criem, emitam e escutem eventos de forma transparente, aprimorando a modularidade e a escalabilidade da aplicação.

## Recursos

- **Vinculação de Eventos:** Vincule métodos a eventos facilmente usando o decorador `@OnEvent`.
- **Comunicação Assíncrona:** Facilite a comunicação entre serviços com acoplamento mínimo.
- **Flexibilidade de Payload:** Suporta qualquer tipo de payload, com recomendação de uso de interfaces TypeScript para dados estruturados.
- **Integração com o Framework CMMV:** Compatibilidade integrada com módulos e serviços do `@cmmv/core`.
- **Tratamento de Erros:** Garante registro robusto de erros e depuração para fluxos orientados a eventos.

## Instalação

Para instalar o módulo `@cmmv/events`:

```bash
$ pnpm add @cmmv/events
```

## Configuração

Nenhuma configuração adicional é necessária. O módulo `@cmmv/events` funciona diretamente com sua aplicação CMMV sem necessidade de ajustes.

## Configurando a Aplicação

No seu arquivo `index.ts`, inclua o `EventsModule` junto com quaisquer módulos ou serviços que utilizem comunicação baseada em eventos:

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

### Criando Ouvintes de Eventos

Use o decorador `@OnEvent` para vincular um método a um evento. Ouvintes de eventos podem ser adicionados a qualquer serviço, controlador ou gateway na sua aplicação.

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

Para emitir um evento, injete o `EventsService` na sua classe e chame o método `emit`:

```typescript
this.eventsService.emit('nome-do-evento', { chave: 'valor' });
```
<br/>

- **Primeiro Parâmetro:** Nome do evento.
- **Segundo Parâmetro:** Payload do evento (qualquer tipo). Recomenda-se usar uma interface TypeScript para a estrutura do payload.

### Exemplo de Emissão de Evento

```typescript
this.eventsService.emit('user.created', { id: 1, name: 'João Silva' });
```

## Decoradores

### `@OnEvent(eventName: string)`
Vincula um método para escutar um evento específico.

## Melhores Práticas

- **Use Interfaces para Payloads:** Defina interfaces TypeScript para os payloads dos eventos para garantir consistência e segurança de tipo.
- **Agrupe Eventos Relacionados:** Use namespaces ou prefixos de eventos para agrupar eventos relacionados (por exemplo, `user.created`, `user.updated`).

## Exemplo de Aplicação

Aqui está um exemplo de uma aplicação utilizando o módulo `@cmmv/events`:

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

### Ouvinte de Usuário

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
        // Lógica de criação de usuário...

        // Emite o evento user.created
        this.eventsService.emit('user.created', { id: 1, name: 'João Silva' });
    }
}
```

## Vantagens

- **Lógica Desacoplada:** Permite uma melhor separação de responsabilidades ao desacoplar emissores e ouvintes de eventos.
- **Comunicação Escalável:** Ideal para aplicações com fluxos de trabalho complexos que requerem comunicação baseada em eventos.
- **Integração Transparente:** Funciona diretamente com o framework CMMV, simplificando a configuração e o uso.
