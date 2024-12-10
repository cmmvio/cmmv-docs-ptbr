# Logger

The ``Logger`` class in the ``@cmmv/core`` module is designed to provide standardized logging for applications. It allows you to log messages at different severity levels (log, error, warning, debug, verbose) and format the output with timestamps, colored contexts, and message levels to improve readability.

| Method   | Description                                                              |
|----------|--------------------------------------------------------------------------|
| `log`    | Logs a general message with `LOG` level and the optional `context`.      |
| `error`  | Logs an error message with `ERROR` level and the optional `context`.     |
| `warning`| Logs a warning message with `WARNING` level and the optional `context`.  |
| `debug`  | Logs a debugging message with `DEBUG` level and the optional `context`.  |
| `verbose`| Logs detailed verbose information with `VERBOSE` level and the optional `context`. |

Each of these methods formats the message, adds a timestamp, and colors the output based on the severity level to ensure clarity during logging.

**Context:**

* A default context (``Server``) is used unless a specific context is provided during instantiation.
* The context is shown in yellow brackets to differentiate between different parts of the application if necessary.

**Timestamps:**

* Each log message is timestamped in the format ``MM/DD/YYYY, HH:MM:SS AM/PM`` to track when events occur.

**Color-Coded Severity Levels:**

* The message severity is highlighted using different colors:
    * ERROR: Red
    * WARNING: Orange
    * DEBUG: Blue
    * VERBOSE: Cyan
    * LOG: Green
* Context is always highlighted in yellow, and the message body is colored based on the severity level.

**Message Formatting:**

* Messages are constructed using a method called ``formatMessage`` that combines the timestamp, severity level, context, and message body into a cohesive, readable log entry.

**Customization:**

The logger can be initialized with a custom context name for more specific categorization of logs.

## Example 

```typescript
const logger = new Logger('MyApp');

logger.log('Application has started');
logger.error('Failed to connect to the database', 'DatabaseService');
logger.warning('Memory usage is high');
logger.debug('User object: ', 'UserService');
logger.verbose('Detailed information about request processing');
```

**Output**

<pre>
<code class="hljs language-shell" lang="shell"><span style="color:green;">[Server]</span> - 12/04/2024, 10:14:32 AM <span style="color:green;">LOG</span> <span style="color:yellow;">[MyApp]</span> <span style="color:green;">Application has started</span> <br>
<span style="color:red;">[Server]</span> - 12/04/2024, 10:15:01 AM <span style="color:red;">ERROR</span> <span style="color:yellow;">[DatabaseService]</span> <span style="color:red;">Failed to connect to the database</span> <br>
<span style="color:orange;">[Server]</span> - 12/04/2024, 10:15:45 AM <span style="color:orange;">WARNING</span> <span style="color:yellow;">[Server]</span> <span style="color:orange;">Memory usage is high</span> <br>
<span style="color:blue;">[Server]</span> - 12/04/2024, 10:16:12 AM <span style="color:blue;">DEBUG</span> <span style="color:yellow;">[UserService]</span> <span style="color:blue;">User object:</span> <br>
<span style="color:cyan;">[Server]</span> - 12/04/2024, 10:16:58 AM <span style="color:cyan;">VERBOSE</span> <span style="color:yellow;">[Server]</span> <span style="color:cyan;">Detailed information about request processing</span> 
</code>
</pre>

## Logger in Classes

The ``Logger`` class in the ``@cmmv/core`` module provides a simple logging utility that can be used in various classes, such as transpilers and services, to log messages, errors, warnings, and debugging information. Below is an example of how to use the ``Logger`` class in your code and what each logging method does.

In the example of the ``RepositoryTranspile`` class, the ``Logger`` is used to log important information during the transpile process. Here is how you can integrate the ``Logger`` into your class:

```typescript
import { ITranspile, Logger, Scope } from '@cmmv/core';

export class RepositoryTranspile implements ITranspile {
    // Initialize the Logger instance with a custom context
    private logger: Logger = new Logger('RepositoryTranspile');

    // Main run method that uses the logger to log the start of the transpile process
    run(): void {
        this.logger.log('Starting transpile process for contracts');

        const contracts = Scope.getArray<any>('__contracts');
        
        // Log when no contracts are found
        if (!contracts || contracts.length === 0) {
            this.logger.warning('No contracts found for transpiling.');
            return;
        }

        contracts.forEach((contract: any) => {
            if (contract.generateController) {
                this.logger.log(`Generating entity and service for contract: ${contract.controllerName}`);
                this.generateEntity(contract);
                this.generateService(contract);
            }
        });

        this.logger.log('Transpile process completed successfully.');
    }

    private generateEntity(contract: any): void {
        try {
            // Code for generating the entity
            this.logger.debug(`Generating entity for ${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Failed to generate entity for ${contract.controllerName}`, error);
        }
    }

    private generateService(contract: any): void {
        try {
            // Code for generating the service
            this.logger.debug(`Generating service for ${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Failed to generate service for ${contract.controllerName}`, error);
        }
    }
}
```

By using these logging methods, you can ensure that your application provides informative, structured, and context-specific logs, which are essential for debugging and monitoring production environments.