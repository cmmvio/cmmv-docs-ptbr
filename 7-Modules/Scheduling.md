# Scheduling (Agora parte do `@cmmv/core` desde a versão v0.13)

> **Aviso:** A partir da versão `0.13`, o módulo `@cmmv/scheduling` foi totalmente integrado ao `@cmmv/core`. Você não precisa mais instalá-lo ou registrá-lo separadamente — todos os recursos de agendamento agora estão disponíveis nativamente.

## Migração

Se sua aplicação utilizava anteriormente o `@cmmv/scheduling`, você pode removê-lo das dependências e atualizar seus imports:

### Antes:
```ts
import { SchedulingModule, SchedulingService, Cron } from '@cmmv/scheduling';
```

### Depois:
```ts
import { Cron } from '@cmmv/core';
```

Você não precisa mais adicionar `SchedulingModule` nem registrar o `SchedulingService` — o agendamento agora está disponível globalmente por padrão.

## Funcionalidades

- **Agendamento com Cron:** Agende tarefas usando expressões cron.
- **Decorador @Cron:** Atribua facilmente tarefas agendadas a métodos de classe.
- **Integração Nativa:** Não é necessário nenhuma configuração manual com aplicações baseadas no core.
- **Suporte a Logger:** Compatível com o `Logger` do `@cmmv/core`.

## Uso

Use o decorador `@Cron` para definir tarefas que devem rodar em um cronograma:

```ts
import { Cron, Logger, Service } from '@cmmv/core';

@Service('task')
export class TaskService {
    private logger: Logger = new Logger('TaskService');

    @Cron('*/5 * * * * *')  // Executa a cada 5 segundos
    handleTask() {
        this.logger.log('Tarefa executada a cada 5 segundos');
    }
}
```

## Formato da Expressão Cron

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    └ Dia da semana (0 - 7) (Domingo = 0 ou 7)
│    │    │    │    └───── Mês (1 - 12)
│    │    │    └────────── Dia do mês (1 - 31)
│    │    └─────────────── Hora (0 - 23)
│    └──────────────────── Minuto (0 - 59)
└───────────────────────── Segundo (0 - 59, opcional)
```

## Exemplos

- `* * * * * *` — Executa a cada segundo
- `*/5 * * * * *` — Executa a cada 5 segundos
- `0 0 * * * *` — Executa a cada hora
- `0 0 12 * * *` — Executa todos os dias ao meio-dia
- `0 0 1 1 *` — Executa uma vez por ano em 1º de janeiro

## Observação Importante sobre Múltiplas Instâncias

Ao rodar múltiplas instâncias da mesma aplicação, **cada instância executará a mesma tarefa cron**. Isso pode causar **execuções redundantes**.

Até que um sistema nativo de limitação por instância seja introduzido, recomenda-se:

- Usar validações baseadas em ENV (`if (process.env.RUN_CRONS === 'true')`)
- Confiar em orquestração externa (ex: designar apenas um pod/container executor no k8s)
- Evitar operações compartilhadas que não sejam idempotentes

## Vantagens

- **Zero Configuração:** O agendamento roda sem necessidade de registro adicional
- **Sintaxe Limpa:** Baseado em decoradores
- **Tipagem Segura:** Totalmente compatível com TypeScript
- **Pronto para o Futuro:** Suporte planejado para orquestração distribuída de tarefas agendadas
