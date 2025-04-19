# Events (Agora parte do `@cmmv/core` desde a versão v0.13)

> **Aviso:** A partir da versão `0.13`, o módulo `@cmmv/events` foi totalmente incorporado ao `@cmmv/core`. Você não precisa mais instalar ou importar `@cmmv/events` separadamente — toda a funcionalidade está disponível por padrão via `@cmmv/core`.

## Migração

Se sua aplicação utilizava o `@cmmv/events` antes da v0.13, você pode removê-lo com segurança das dependências e atualizar seus imports da seguinte forma:

### Antes:
```ts
import { EventsModule, EventsService, OnEvent } from '@cmmv/events';
```

### Depois:
```ts
import { EventsService, OnEvent } from '@cmmv/core';
```

Não é mais necessário incluir o `EventsModule` na lista de módulos — o suporte a eventos agora é global.

## Funcionalidades

- **Vinculação de Eventos:** Utilize o decorador `@OnEvent` para associar métodos a eventos.
- **Comunicação Assíncrona:** Facilita fluxos de trabalho desacoplados entre serviços.
- **Payloads Flexíveis:** Suporta qualquer tipo de dado, com tipagem forte usando TypeScript.
- **Integração Transparente:** Já integrado ao núcleo do framework.
- **Tratamento de Erros:** Execução confiável com logs detalhados para depuração.

## Instalação

> ❌ Nenhuma instalação necessária. O sistema de eventos já está embutido no `@cmmv/core`.

## Uso

### Escutando Eventos

Utilize o decorador `@OnEvent` para responder a eventos:

```ts
import { Service, OnEvent } from '@cmmv/core';

@Service('listener')
export class Listener {
    @OnEvent('task.completed')
    public onTaskCompleted(payload: any) {
        console.log('Tarefa concluída:', payload);
    }
}
```

### Emitindo Eventos

Use o `EventsService` para emitir eventos de qualquer serviço ou controller:

```ts
import { EventsService, Service } from '@cmmv/core';

@Service('task-service')
export class TaskService {
    constructor(private readonly events: EventsService) {}

    public completeTask() {
        // ... lógica da tarefa ...
        this.events.emit('task.completed', { id: 123, title: 'Escrever documentação' });
    }
}
```

## Boas Práticas

- **Use eventos com namespace:** ex: `user.created`, `task.failed`.
- **Defina interfaces para os payloads:** isso garante clareza e segurança de tipos.
- **Evite execução redundante de eventos/crons** em ambientes com múltiplas instâncias, a menos que isso seja controlado por configuração ou infraestrutura.

## Compatibilidade

- Totalmente compatível com aplicações usando `@cmmv/core >= 0.13`.
- Aplicações legadas usando `@cmmv/events` continuarão funcionando, mas o uso está **deprecated** (obsoleto).

## Vantagens

- **Sem necessidade de configuração extra**
- **Menos boilerplate**
- **Maior coesão com o framework**
- **Ideal para design modular e baseado em serviços**

