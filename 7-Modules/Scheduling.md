# Scheduling 

The ``@cmmv/scheduling`` package provides a simple way to schedule tasks in your CMMV application using cron patterns. This module is built using the ``cron`` library, and the ``@Cron`` decorator allows you to easily schedule methods to run at specific intervals.

To install the scheduling module, run the following command:

```bash
$ pnpm add @cmmv/scheduling cron
```

## Usage

The ``@Cron`` decorator is used to define methods that should be scheduled according to a cron pattern. The scheduling is powered by the cron library, which provides flexible and powerful cron scheduling capabilities.

After installing the package, you can use the ``@Cron`` decorator and the SchedulingService in your project:

```typescript
import { Cron } from '@cmmv/scheduling';
import { Logger } from '@cmmv/core';

export class TaskService {
    private logger: Logger = new Logger('TaskService');

    @Cron('*/5 * * * * *')  // Runs every 5 seconds
    handleTask() {
        this.logger.log('Task executed every 5 seconds');
    }
}
```

## Setting

Ensure that the ``SchedulingService`` is initialized during the startup of your application. This will register and start all the scheduled tasks defined with the ``@Cron`` decorator.

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

## Decorator

The ``@Cron`` decorator is used to schedule a method to run based on a cron pattern. It accepts a cron expression as its argument, which defines the schedule.

```typescript
@Cron('*/5 * * * * *')  // Runs every 5 seconds
handleTask() {
    this.logger.log('Task executed every 5 seconds');
}
```

The cron patterns follow the standard format used by the cron library:

```scss
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    │
│    │    │    │    │    └ Day of the week (0 - 7) (Sunday=0 or 7)
│    │    │    │    └───── Month (1 - 12)
│    │    │    └────────── Day of the month (1 - 31)
│    │    └─────────────── Hour (0 - 23)
│    └──────────────────── Minute (0 - 59)
└───────────────────────── Second (0 - 59, optional)
```

**Some Sample Cron Patterns**

Here are a few examples of cron patterns you can use with the ``@Cron`` decorator:

* ``* * * * * *`` – Runs every second
* ``*/5 * * * * *`` – Runs every 5 seconds
* ``0 0 * * * *`` – Runs every hour
* ``0 0 12 * * *`` – Runs every day at noon
* ``0 0 1 1 *`` – Runs at midnight on January 1st

The ``@cmmv/scheduling`` module provides a powerful and flexible way to schedule tasks in a CMMV application using cron patterns. The ``@Cron`` decorator makes it easy to define when certain methods should run, and the ``SchedulingService`` ensures these tasks are properly managed and executed at runtime.