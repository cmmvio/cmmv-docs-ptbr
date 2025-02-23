# Agendamento

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/scheduling](https://github.com/cmmvio/cmmv/tree/main/packages/scheduling)

O pacote ``@cmmv/scheduling`` oferece uma maneira simples de agendar tarefas em sua aplicação CMMV usando padrões cron. Este módulo é construído com a biblioteca ``cron``, e o decorador ``@Cron`` permite que você agende métodos para serem executados em intervalos específicos.

Para instalar o módulo de agendamento, execute o seguinte comando:

```bash
$ pnpm add @cmmv/scheduling cron
```

## Uso

O decorador ``@Cron`` é usado para definir métodos que devem ser agendados de acordo com um padrão cron. O agendamento é gerenciado pela biblioteca cron, que fornece recursos de agendamento flexíveis e poderosos.

Depois de instalar o pacote, você pode usar o decorador ``@Cron`` e o ``SchedulingService`` em seu projeto:

```typescript
import { Cron } from '@cmmv/scheduling';
import { Logger } from '@cmmv/core';

export class TaskService {
    private logger: Logger = new Logger('TaskService');

    @Cron('*/5 * * * * *')  // Executa a cada 5 segundos
    handleTask() {
        this.logger.log('Tarefa executada a cada 5 segundos');
    }
}
```

## Configuração

Certifique-se de que o ``SchedulingService`` seja inicializado durante a inicialização da sua aplicação. Isso registrará e iniciará todas as tarefas agendadas definidas com o decorador ``@Cron``.

```typescript
require('dotenv').config();

import { Application } from '@cmmv/core';
import { ExpressAdapter, ExpressModule } from '@cmmv/http';
import { WSModule, WSAdapter } from '@cmmv/ws';
...
import { SchedulingModule, SchedulingService } from '@cmmv/scheduling';

Application.create({
    httpAdapter: ExpressAdapter,
    wsAdapter: WSAdapter,
    modules: [
        ...
        SchedulingModule
    ],
    services: [..., SchedulingService]
});
```

## Decorador

O decorador ``@Cron`` é usado para agendar um método para ser executado com base em um padrão cron. Ele aceita uma expressão cron como argumento, que define o cronograma.

```typescript
@Cron('*/5 * * * * *')  // Executa a cada 5 segundos
handleTask() {
    this.logger.log('Tarefa executada a cada 5 segundos');
}
```

Os padrões cron seguem o formato padrão usado pela biblioteca cron:

```scss
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    └ Dia da semana (0 - 7) (Domingo=0 ou 7)
│    │    │    │    └───── Mês (1 - 12)
│    │    │    └────────── Dia do mês (1 - 31)
│    │    └─────────────── Hora (0 - 23)
│    └──────────────────── Minuto (0 - 59)
└───────────────────────── Segundo (0 - 59, opcional)
```

**Alguns Exemplos de Padrões Cron**

Aqui estão alguns exemplos de padrões cron que você pode usar com o decorador ``@Cron``:

* ``* * * * * *`` – Executa a cada segundo
* ``*/5 * * * * *`` – Executa a cada 5 segundos
* ``0 0 * * * *`` – Executa a cada hora
* ``0 0 12 * * *`` – Executa todos os dias ao meio-dia
* ``0 0 1 1 *`` – Executa à meia-noite do dia 1º de janeiro

O módulo ``@cmmv/scheduling`` fornece uma maneira poderosa e flexível de agendar tarefas em uma aplicação CMMV usando padrões cron. O decorador ``@Cron`` facilita a definição de quando certos métodos devem ser executados, e o ``SchedulingService`` garante que essas tarefas sejam gerenciadas e executadas corretamente em tempo de execução.
