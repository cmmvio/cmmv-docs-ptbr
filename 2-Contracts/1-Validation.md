# Validação

O CMMV suporta validação automática de contratos usando o módulo `class-validator` ([NPM](https://www.npmjs.com/package/class-validator)). Durante as operações de inserção e atualização de dados, o CMMV pode aplicar regras de validação aos campos do contrato, garantindo que os dados atendam aos critérios definidos antes de serem processados. Você pode especificar regras de validação para cada campo adicionando o parâmetro `validations`.

Essas validações são executadas automaticamente durante as operações de `insert` e `update`, e o sistema retornará erros caso alguma validação falhe.

Abaixo está um exemplo de `TasksContract`, onde cada campo é validado usando diferentes regras:

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [{ 
            type: "IsString",
            message: "Label inválido"
        }, { 
            type: "IsNotEmpty",
            message: "Label não pode estar vazio"
        }]
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de checked inválido"
        }]
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de removed inválido"
        }]
    })
    removed: boolean;
}
```

## Estrutura 

Cada regra de validação é configurada usando o parâmetro `validations`, que contém um array de objetos de validação. Cada objeto pode incluir as seguintes propriedades:

* **type:** O tipo de validação a ser aplicado, como `IsString`, `IsNotEmpty`, `IsBoolean`, etc.
* **message (opcional):** Uma mensagem de erro personalizada que será exibida quando a validação falhar.
* **context (opcional):** Informações adicionais de contexto que podem ser incluídas com a validação.

## Tipos de Validação

O `class-validator` fornece uma ampla gama de tipos de validação que podem ser aplicados aos campos do contrato. Alguns tipos comuns são:

| **Decorador**               | **Descrição**                                                                                                                                                   |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IsDefined                   | Verifica se o valor está definido (!== undefined, !== null). Este é o único decorador que ignora a opção skipMissingProperties.                                  |
| IsOptional                  | Verifica se o valor fornecido está vazio (=== null, === undefined) e, se estiver, ignora todos os validadores na propriedade.                                    |
| Equals                      | Verifica se o valor é igual (`===`) à comparação.                                                                                                            |
| NotEquals                   | Verifica se o valor não é igual (`!==`) à comparação.                                                                                                        |
| IsEmpty                     | Verifica se o valor fornecido está vazio (=== '', === null, === undefined).                                                                                    |
| IsNotEmpty                  | Verifica se o valor fornecido não está vazio (!== '', !== null, !== undefined).                                                                                |
| IsIn                        | Verifica se o valor está em um array de valores permitidos.                                                                                                    |
| IsNotIn                     | Verifica se o valor não está em um array de valores não permitidos.                                                                                            |
| IsBoolean                   | Verifica se um valor é um booleano.                                                                                                                            |
| IsDate                      | Verifica se o valor é uma data.                                                                                                                                |
| IsString                    | Verifica se o valor é uma string.                                                                                                                              |
| IsNumber                    | Verifica se o valor é um número.                                                                                                                               |
| IsInt                       | Verifica se o valor é um número inteiro.                                                                                                                       |
| IsArray                     | Verifica se o valor é um array.                                                                                                                                |
| IsEnum                      | Verifica se o valor é um enum válido.                                                                                                                          |
| Min                         | Verifica se o número fornecido é maior ou igual ao número fornecido.                                                                                           |
| Max                         | Verifica se o número fornecido é menor ou igual ao número fornecido.                                                                                           |
| IsPositive                  | Verifica se o valor é um número positivo maior que zero.                                                                                                       |
| IsNegative                  | Verifica se o valor é um número negativo menor que zero.                                                                                                       |
| IsUUID                      | Verifica se a string é um UUID (versão 3, 4, 5 ou todos).                                                                                                      |
| IsEmail                     | Verifica se a string é um email válido.                                                                                                                        |
| IsNotEmptyObject            | Verifica se um objeto não está vazio.                                                                                                                           |
| ArrayNotEmpty               | Verifica se um array não está vazio.                                                                                                                            |

Quando um contrato é usado em operações de inserção ou atualização, o CMMV converte automaticamente os dados recebidos em instâncias de classe, aplica quaisquer transformações e realiza a validação de acordo com as regras definidas no contrato. Se algum campo não atender aos critérios de validação, o processo é interrompido e uma lista de erros de validação é retornada.

Esse processo garante que os dados estejam sempre em conformidade com as regras definidas no contrato, protegendo a integridade do sistema.

## Validação em Modelos

Quando as validações são definidas nos campos do contrato, o framework CMMV adiciona automaticamente os decoradores correspondentes do `class-validator` ao modelo gerado. Esses decoradores realizam validações em tempo de execução para garantir que os dados aderem às regras definidas. 

A geração de modelos insere os decoradores de validação corretos acima de cada campo, junto com quaisquer opções de validação especificadas, como mensagens de erro personalizadas.

A validação garante consistência e integridade dos dados em toda a aplicação.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Task',
    protoPath: 'src/protos/task.proto',
    protoPackage: 'task',
})
export class TasksContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [{ 
            type: "IsString",
            message: "Invalid label"
        }, { 
            type: "IsNotEmpty",
            message: "Invalid label"
        }]
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Invalid checked type"
        }]
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Invalid removed type"
        }]
    })
    removed: boolean;
}
```

Quando o contrato é processado pelo framework, o seguinte modelo é gerado automaticamente. Os decoradores de ``class-validator`` são adicionados a cada campo para executar a validação de acordo com as regras especificadas:

```typescript
// Generated automatically by CMMV

import { IsString, IsNotEmpty, IsBoolean } from 'class-validator';
        
export interface ITask {
    id?: any;
    label: string;
    checked: boolean;
    removed: boolean;
}

export class Task implements ITask {
    id?: any;

    @IsString({ message: "Invalid label" })
    @IsNotEmpty({ message: "Invalid label" })
    label: string

    @IsBoolean({ message: "Invalid checked type" })
    checked: boolean = false;

    @IsBoolean({ message: "Invalid removed type" })
    removed: boolean = false;
}
```

Essa abordagem garante que os dados sejam validados corretamente e que o modelo seja gerado dinamicamente com base nas definições do contrato, tornando a estrutura altamente adaptável e eficiente para gerenciar validações em vários modelos.

## Validação nos Serviços

Além do modelo gerado, os serviços também passam por modificações para garantir que os dados de entrada sejam validados corretamente antes de serem processados ou armazenados. Tanto o serviço gerado pelo `@cmmv/http` quanto a implementação do repositório de `@cmmv/repository` incluem mecanismos de validação embutidos usando o módulo `class-validator`.

Essa validação assegura que todos os dados de entrada atendam às regras especificadas nos campos do contrato antes que qualquer operação de banco de dados ou lógica de negócio seja executada.

Aqui está um exemplo da função `add` no serviço gerado pelo repositório:

```typescript
async add(item: ITask, req?: any): Promise<TaskEntity> {
    return new Promise(async (resolve, reject) => {
        try {
            Telemetry.start('TaskService::Add', req?.requestId);

            // Converter objeto comum em uma instância de classe e validar os dados
            const newItem = plainToClass(Task, item, { 
                exposeUnsetFields: true,
                enableImplicitConversion: true
            }); 

            // Validar o newItem com class-validator
            const errors = await validate(newItem, { 
                skipMissingProperties: true 
            });

            if (errors.length > 0) {
                // Se a validação falhar, retornar os erros
                Telemetry.end('TaskService::Add', req?.requestId);
                reject(errors);
            } else {
                // Se a validação for bem-sucedida, prosseguir com o repositório
                const result = await Repository.insert<TaskEntity>(
                    TaskEntity, newItem
                );
                Telemetry.end('TaskService::Add', req?.requestId);
                resolve(result);
            }
        } catch (e) {
            Telemetry.end('TaskService::Add', req?.requestId);
            console.log(e);
            reject(e);
        }
    });
}
```

### Detalhes do Processo

* **Passo de Validação:** Antes de realizar qualquer operação no banco de dados, os dados de entrada são convertidos em uma instância de classe usando `plainToClass` e, em seguida, validados usando `validate` do `class-validator`.

* **Manipulação de Erros:** Se a validação falhar, o processo é interrompido e os erros são retornados ao chamador sem prosseguir para a operação no banco de dados.

* **Rastreamento por Telemetria:** As funções de telemetria monitoram o início e o fim da operação `add`, garantindo que todas as ações sejam registradas para desempenho e análise de logs.

* **Inserção no Repositório:** Após a validação, o método `Repository.insert` insere os dados validados no banco de dados.

Essa abordagem garante que todos os dados de entrada sejam validados de forma consistente em todo o sistema, tornando mais fácil gerenciar e impor regras de validação no nível do serviço, protegendo a integridade dos dados e reduzindo o risco de informações inválidas serem armazenadas ou processadas.

## Mais Sobre Validações

Para garantir a robustez e a integridade do sistema, a validação é aplicada em vários níveis dentro do framework CMMV. Essa funcionalidade cobre desde o contrato até o serviço, oferecendo uma abordagem uniforme para o gerenciamento de regras de validação. Isso reduz significativamente a probabilidade de erros de dados e melhora a confiabilidade da aplicação.

Para mais informações e exemplos detalhados, consulte a documentação oficial do módulo `class-validator` ou explore os módulos adicionais do CMMV para descobrir funcionalidades avançadas que podem ser integradas ao seu projeto.
