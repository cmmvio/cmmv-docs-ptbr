# Logger

A classe `Logger` no módulo `@cmmv/core` foi projetada para fornecer um sistema de logs padronizado para aplicações. Ela permite registrar mensagens em diferentes níveis de severidade (`log`, `error`, `warning`, `debug`, `verbose`) e formatar a saída com timestamps, contextos coloridos e níveis de mensagem para melhorar a legibilidade.

| Método    | Descrição                                                               |
|-----------|-------------------------------------------------------------------------|
| `log`    | Registra uma mensagem geral com o nível `LOG` e o `context` opcional.      |
| `error`  | Registra uma mensagem de erro com o nível `ERROR` e o `context` opcional.   |
| `warning`| Registra uma mensagem de aviso com o nível `WARNING` e o `context` opcional.|
| `debug`  | Registra uma mensagem de depuração com o nível `DEBUG` e o `context` opcional. |
| `verbose`| Registra informações detalhadas com o nível `VERBOSE` e o `context` opcional.|

Cada um desses métodos formata a mensagem, adiciona um timestamp e colore a saída com base no nível de severidade para garantir clareza durante o registro.

**Contexto:**

* Um contexto padrão (`Server`) é usado, a menos que um contexto específico seja fornecido durante a instância.
* O contexto é exibido entre colchetes amarelos para diferenciar as partes da aplicação, caso necessário.

**Timestamps:**

* Cada mensagem de log possui um timestamp no formato `MM/DD/YYYY, HH:MM:SS AM/PM`, permitindo rastrear quando os eventos ocorrem.

**Níveis de Severidade com Cores:**

* A severidade da mensagem é destacada com cores diferentes:
    * **ERROR:** Vermelho
    * **WARNING:** Laranja
    * **DEBUG:** Azul
    * **VERBOSE:** Ciano
    * **LOG:** Verde
* O contexto é sempre destacado em amarelo, e o corpo da mensagem é colorido com base no nível de severidade.

**Formatação da Mensagem:**

* As mensagens são construídas usando um método chamado `formatMessage`, que combina o timestamp, o nível de severidade, o contexto e o corpo da mensagem em uma entrada de log coesa e legível.

**Personalização:**

O logger pode ser inicializado com um nome de contexto personalizado para uma categorização mais específica dos logs.

## Exemplo 

```typescript
const logger = new Logger('MyApp');

logger.log('Aplicação iniciada');
logger.error('Falha ao conectar ao banco de dados', 'DatabaseService');
logger.warning('Uso de memória está alto');
logger.debug('Objeto do usuário: ', 'UserService');
logger.verbose('Informações detalhadas sobre o processamento da requisição');
```

**Saída**

<pre>
<code class="hljs language-shell" lang="shell">
<span style="color:green;">[Server]</span> - 12/04/2024, 10:14:32 AM <span style="color:green;">LOG</span> <span style="color:yellow;">[MyApp]</span> <span style="color:green;">Aplicação iniciada</span> <br>
<span style="color:red;">[Server]</span> - 12/04/2024, 10:15:01 AM <span style="color:red;">ERROR</span> <span style="color:yellow;">[DatabaseService]</span> <span style="color:red;">Falha ao conectar ao banco de dados</span> <br>
<span style="color:orange;">[Server]</span> - 12/04/2024, 10:15:45 AM <span style="color:orange;">WARNING</span> <span style="color:yellow;">[Server]</span> <span style="color:orange;">Uso de memória está alto</span> <br>
<span style="color:blue;">[Server]</span> - 12/04/2024, 10:16:12 AM <span style="color:blue;">DEBUG</span> <span style="color:yellow;">[UserService]</span> <span style="color:blue;">Objeto do usuário:</span> <br>
<span style="color:cyan;">[Server]</span> - 12/04/2024, 10:16:58 AM <span style="color:cyan;">VERBOSE</span> <span style="color:yellow;">[Server]</span> <span style="color:cyan;">Informações detalhadas sobre o processamento da requisição</span> 
</code>
</pre>

## Logger em Classes

A classe `Logger` no módulo `@cmmv/core` fornece uma utilidade de registro simples que pode ser usada em várias classes, como transpiladores e serviços, para registrar mensagens, erros, avisos e informações de depuração. Abaixo está um exemplo de como usar a classe `Logger` no seu código e o que cada método de log faz.

No exemplo da classe `RepositoryTranspile`, o `Logger` é usado para registrar informações importantes durante o processo de transpilação. Veja como integrá-lo à sua classe:

```typescript
import { ITranspile, Logger, Scope } from '@cmmv/core';

export class RepositoryTranspile implements ITranspile {
    // Inicializa a instância do Logger com um contexto personalizado
    private logger: Logger = new Logger('RepositoryTranspile');

    // Método principal que usa o logger para registrar o início do processo de transpilação
    run(): void {
        this.logger.log('Iniciando processo de transpilação para contratos');

        const contracts = Scope.getArray<any>('__contracts');
        
        // Registra quando nenhum contrato é encontrado
        if (!contracts || contracts.length === 0) {
            this.logger.warning('Nenhum contrato encontrado para transpilação.');
            return;
        }

        contracts.forEach((contract: any) => {
            if (contract.generateController) {
                this.logger.log(`Gerando entidade e serviço para o contrato: \${contract.controllerName}`);
                this.generateEntity(contract);
                this.generateService(contract);
            }
        });

        this.logger.log('Processo de transpilação concluído com sucesso.');
    }

    private generateEntity(contract: any): void {
        try {
            // Código para gerar a entidade
            this.logger.debug(`Gerando entidade para \${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Falha ao gerar a entidade para \${contract.controllerName}`, error);
        }
    }

    private generateService(contract: any): void {
        try {
            // Código para gerar o serviço
            this.logger.debug(`Gerando serviço para \${contract.controllerName}`);
        } catch (error) {
            this.logger.error(`Falha ao gerar o serviço para \${contract.controllerName}`, error);
        }
    }
}
```

Ao usar esses métodos de log, você pode garantir que sua aplicação forneça logs informativos, estruturados e específicos para o contexto, essenciais para depuração e monitoramento em ambientes de produção.
