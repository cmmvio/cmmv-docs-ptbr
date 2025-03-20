# Logger

A classe `Logger` no módulo `@cmmv/core` foi projetada para fornecer um sistema de logging padronizado para aplicações. Ela permite registrar mensagens em diferentes níveis de severidade (log, error, warning, debug, verbose) e formatar a saída com timestamps, contextos coloridos e níveis de mensagens para melhorar a legibilidade.

| Método   | Descrição                                                                |
|----------|--------------------------------------------------------------------------|
| `log`    | Registra uma mensagem geral com nível `LOG` e um `context` opcional.     |
| `error`  | Registra uma mensagem de erro com nível `ERROR` e um `context` opcional.  |
| `warning`| Registra um aviso com nível `WARNING` e um `context` opcional.           |
| `debug`  | Registra uma mensagem de depuração com nível `DEBUG` e um `context` opcional. |
| `verbose`| Registra informações detalhadas com nível `VERBOSE` e um `context` opcional. |

Cada um desses métodos formata a mensagem, adiciona um timestamp e colore a saída com base no nível de severidade para garantir clareza no registro de logs.

**Contexto:**

* Um contexto padrão (`Server`) é usado, a menos que um contexto específico seja fornecido durante a instanciação.
* O contexto é exibido entre colchetes amarelos para diferenciar diferentes partes da aplicação, se necessário.

**Timestamps:**

* Cada mensagem de log é registrada com um timestamp no formato `MM/DD/YYYY, HH:MM:SS AM/PM` para rastrear quando os eventos ocorrem.

**Níveis de Severidade Coloridos:**

* A severidade das mensagens é destacada usando diferentes cores:
    * ERROR: Vermelho
    * WARNING: Laranja
    * DEBUG: Azul
    * VERBOSE: Ciano
    * LOG: Verde
* O contexto é sempre destacado em amarelo, e o corpo da mensagem é colorido com base no nível de severidade.

**Formatação da Mensagem:**

* As mensagens são construídas usando um método chamado `formatMessage`, que combina timestamp, nível de severidade, contexto e corpo da mensagem em um log estruturado e legível.

**Personalização:**

O logger pode ser inicializado com um nome de contexto personalizado para uma categorização mais específica dos logs.

## Exemplo

```typescript
const logger = new Logger('MyApp');

logger.log('Aplicação iniciada');
logger.error('Falha ao conectar ao banco de dados', 'DatabaseService');
logger.warning('Uso de memória alto');
logger.debug('Objeto do usuário:', 'UserService');
logger.verbose('Informações detalhadas sobre o processamento da requisição');
```

**Saída**

<pre>
<code class="hljs language-shell" lang="shell"><span style="color:green;">[Server]</span> - 12/04/2024, 10:14:32 AM <span style="color:green;">LOG</span> <span style="color:yellow;">[MyApp]</span> <span style="color:green;">Aplicação iniciada</span> <br>
<span style="color:red;">[Server]</span> - 12/04/2024, 10:15:01 AM <span style="color:red;">ERROR</span> <span style="color:yellow;">[DatabaseService]</span> <span style="color:red;">Falha ao conectar ao banco de dados</span> <br>
<span style="color:orange;">[Server]</span> - 12/04/2024, 10:15:45 AM <span style="color:orange;">WARNING</span> <span style="color:yellow;">[Server]</span> <span style="color:orange;">Uso de memória alto</span> <br>
<span style="color:blue;">[Server]</span> - 12/04/2024, 10:16:12 AM <span style="color:blue;">DEBUG</span> <span style="color:yellow;">[UserService]</span> <span style="color:blue;">Objeto do usuário:</span> <br>
<span style="color:cyan;">[Server]</span> - 12/04/2024, 10:16:58 AM <span style="color:cyan;">VERBOSE</span> <span style="color:yellow;">[Server]</span> <span style="color:cyan;">Informações detalhadas sobre o processamento da requisição</span>
</code>
</pre>

## Logger em Classes

A classe `Logger` no módulo `@cmmv/core` fornece uma ferramenta de logging simples que pode ser usada em diversas classes, como transpiladores e serviços, para registrar mensagens, erros, avisos e informações de depuração. Abaixo está um exemplo de como utilizar a classe `Logger` no seu código e o que cada método de logging faz.

No exemplo da classe `RepositoryTranspile`, o `Logger` é usado para registrar informações importantes durante o processo de transpile. Veja como integrar o `Logger` na sua classe:

```typescript
import { ITranspile, Logger, Scope } from '@cmmv/core';

export class RepositoryTranspile implements ITranspile {
    // Inicializa a instância do Logger com um contexto personalizado
    private logger: Logger = new Logger('RepositoryTranspile');

    // Método principal que utiliza o logger para registrar o início do processo de transpile
    run(): void {
        this.logger.log('Iniciando o processo de transpile para contratos');

        const contracts = Scope.getArray<any>('__contracts');

        // Registra quando nenhum contrato é encontrado
        if (!contracts || contracts.length === 0) {
            this.logger.warning('Nenhum contrato encontrado para transpile.');
            return;
        }

        contracts.forEach((contract: any) => {
            if (contract.generateController) {
                this.logger.log(`Gerando entidade e serviço para o contrato: \${contract.controllerName}`);
                this.generateEntity(contract);
                this.generateService(contract);
            }
        });

        this.logger.log('Processo de transpile concluído com sucesso.');
    }

    private generateEntity(contract: any): void {
        try {
            // Código para gerar a entidade
            this.logger.debug(`Gerando entidade para \${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Falha ao gerar entidade para \${contract.controllerName}`, error);
        }
    }

    private generateService(contract: any): void {
        try {
            // Código para gerar o serviço
            this.logger.debug(`Gerando serviço para \${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Falha ao gerar serviço para \${contract.controllerName}`, error);
        }
    }
}
```

Ao utilizar esses métodos de logging, você garante que sua aplicação fornece logs informativos, estruturados e específicos para cada contexto, essenciais para depuração e monitoramento em ambientes de produção.
