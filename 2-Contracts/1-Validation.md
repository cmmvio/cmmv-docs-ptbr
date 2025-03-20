# Validação

O CMMV suporta validação automática de contratos usando o módulo `class-validator` ([NPM](https://www.npmjs.com/package/class-validator)). Durante operações de inserção e atualização de dados, o CMMV pode aplicar regras de validação aos campos do contrato, garantindo que os dados atendam aos critérios definidos antes de serem processados. Você pode especificar regras de validação para cada campo adicionando o parâmetro `validations`.

Essas validações são executadas automaticamente durante operações de `insert` e `update`, e o sistema retornará erros se alguma validação falhar.

Abaixo está um exemplo de um `TasksContract`, onde cada campo é validado usando diferentes regras:

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
            message: "Rótulo inválido"
        }, {
            type: "IsNotEmpty",
            message: "O rótulo não pode estar vazio"
        }]
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de 'checked' inválido"
        }]
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de 'removed' inválido"
        }]
    })
    removed: boolean;
}
```

## Estrutura

Cada regra de validação é configurada usando o parâmetro `validations`, que contém um array de objetos de validação. Cada objeto pode incluir as seguintes propriedades:

* **type:** O tipo de validação a ser aplicado, como IsString, IsNotEmpty, IsBoolean, etc.
* **message (opcional):** Uma mensagem de erro personalizada que será exibida quando a validação falhar.
* **context (opcional):** Informações adicionais de contexto que podem ser incluídas com a validação.

## Tipos de Validação

O `class-validator` oferece uma ampla gama de tipos de validação que podem ser aplicados aos campos do contrato. Alguns tipos comuns são:

| **Decorador**                | **Descrição**                                                                                                                                                                   |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IsDefined        | Verifica se o valor está definido (!== undefined, !== null). Este é o único decorador que ignora a opção skipMissingProperties.                                                       |
| IsOptional                 | Verifica se o valor dado está vazio (=== null, === undefined) e, se sim, ignora todos os validadores na propriedade.                                                                  |
| Equals      | Verifica se o valor é igual ("===") em uma comparação.                                                                                                                                       |
| NotEquals   | Verifica se o valor não é igual ("!==") em uma comparação.                                                                                                                                    |
| IsEmpty                    | Verifica se o valor dado está vazio (=== '', === null, === undefined).                                                                                                                |
| IsNotEmpty                 | Verifica se o valor dado não está vazio (!== '', !== null, !== undefined).                                                                                                            |
| IsIn          | Verifica se o valor está em um array de valores permitidos.                                                                                                                                |
| IsNotIn       | Verifica se o valor não está em um array de valores proibidos.                                                                                                                         |
| IsBoolean                  | Verifica se o valor é um booleano.                                                                                                                                                  |
| IsDate                     | Verifica se o valor é uma data.                                                                                                                                                   |
| IsString                   | Verifica se o valor é uma string.                                                                                                                                                 |
| IsNumber | Verifica se o valor é um número.                                                                                                                                       |
| IsInt                      | Verifica se o valor é um número inteiro.                                                                                                                                        |
| IsArray                    | Verifica se o valor é um array.                                                                                                                                                |
| IsEnum       | Verifica se o valor é um enum válido.                                                                                                                                             |
| IsDivisibleBy   | Verifica se o valor é um número divisível por outro.                                                                                                                     |
| IsPositive                 | Verifica se o valor é um número positivo maior que zero.                                                                                                                      |
| IsNegative                 | Verifica se o valor é um número negativo menor que zero.                                                                                                                      |
| Min             | Verifica se o número dado é maior ou igual ao número especificado.                                                                                                             |
| Max             | Verifica se o número dado é menor ou igual ao número especificado.                                                                                                                |
| MinDate | Verifica se o valor é uma data posterior à data especificada.                                                                                                            |
| MaxDate | Verifica se o valor é uma data anterior à data especificada.                                                                                                           |
| IsBooleanString            | Verifica se uma string é um booleano (por exemplo, "true", "false", "1" ou "0").                                                                                                         |
| IsDateString               | Alias para @IsISO8601().                                                                                                                                                          |
| IsNumberString | Verifica se uma string é um número.                                                                                                                                  |
| Contains       | Verifica se a string contém a semente.                                                                                                                                          |
| NotContains    | Verifica se a string não contém a semente.                                                                                                                                      |
| IsAlpha                    | Verifica se a string contém apenas letras (a-zA-Z).                                                                                                                             |
| IsAlphanumeric             | Verifica se a string contém apenas letras e números.                                                                                                                          |
| IsDecimal | Verifica se a string é um valor decimal válido.                                                                                                                          |
| IsAscii                    | Verifica se a string contém apenas caracteres ASCII.                                                                                                                                  |
| IsBase32                   | Verifica se a string é codificada em base32.                                                                                                                                           |
| IsBase58                   | Verifica se a string é codificada em base58.                                                                                                                                           |
| IsBase64 | Verifica se a string é codificada em base64.                                                                                                                                    |
| IsIBAN                     | Verifica se a string é um IBAN (Número de Conta Bancária Internacional).                                                                                                              |
| IsBIC                      | Verifica se a string é um BIC (Código de Identificação Bancária) ou código SWIFT.                                                                                                           |
| IsByteLength | Verifica se o comprimento da string (em bytes) está dentro de um intervalo.                                                                                                          |
| IsCreditCard               | Verifica se a string é um cartão de crédito.                                                                                                                                          |
| IsCurrency | Verifica se a string é um valor de moeda válido.                                                                                                                     |
| IsEthereumAddress          | Verifica se a string é um endereço Ethereum usando regex básico. Não valida somas de verificação de endereço.                                                                              |
| IsDataURI                  | Verifica se a string está no formato de URI de dados.                                                                                                                                       |
| IsEmail | Verifica se a string é um e-mail.                                                                                                                                           |
| IsFQDN | Verifica se a string é um nome de domínio totalmente qualificado (por exemplo, domain.com).                                                                                                      |
| IsHexColor                 | Verifica se a string é uma cor hexadecimal.                                                                                                                                     |
| IsISBN | Verifica se a string é um ISBN (versão 10 ou 13).                                                                                                                           |
| IsUUID | Verifica se a string é um UUID (versão 3, 4, 5 ou todos).                                                                                                                       |
| ArrayContains | Verifica se o array contém todos os valores do array de valores fornecido.                                                                                                              |
| ArrayNotEmpty              | Verifica se o array fornecido não está vazio.                                                                                                                                             |
| ArrayMinSize    | Verifica se o comprimento do array é maior ou igual ao número especificado.                                                                                                   |
| @Allow()                      | Impede a remoção da propriedade quando nenhuma outra restrição é especificada para ela.                                                                                               |

Quando um contrato é usado em operações de inserção ou atualização, o CMMV converte automaticamente os dados recebidos em instâncias de classe, aplica quaisquer transformações e realiza a validação de acordo com as regras definidas no contrato. Se algum campo não atender aos critérios de validação, o processo é interrompido e uma lista de erros de validação é retornada.

Esse processo garante que os dados estejam sempre em conformidade com as regras definidas no contrato, protegendo a integridade do sistema.

## Validação em Modelos

Quando as validações são definidas nos campos do contrato, o framework CMMV adiciona automaticamente os decoradores correspondentes do `class-validator` ao modelo gerado. Esses decoradores realizam validação em tempo de execução para garantir que os dados sigam as regras definidas. O processo de geração do modelo insere os decoradores de validação corretos acima de cada campo, juntamente com quaisquer opções de validação especificadas, como mensagens de erro personalizadas.

Aqui está um exemplo de como definir um contrato com validações:

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
            message: "Rótulo inválido"
        }, {
            type: "IsNotEmpty",
            message: "Rótulo inválido"
        }]
    })
    label: string;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de 'checked' inválido"
        }]
    })
    checked: boolean;

    @ContractField({
        protoType: 'bool',
        defaultValue: false,
        validations: [{
            type: "IsBoolean",
            message: "Tipo de 'removed' inválido"
        }]
    })
    removed: boolean;
}
```

Quando o contrato é processado pelo framework, o seguinte modelo é gerado automaticamente. Os decoradores do `class-validator` são adicionados a cada campo para realizar a validação de acordo com as regras especificadas:

```typescript
// Gerado automaticamente pelo CMMV

import { IsString, IsNotEmpty, IsBoolean } from 'class-validator';

export interface ITask {
    id?: any;
    label: string;
    checked: boolean;
    removed: boolean;
}

export class Task implements ITask {
    id?: any;

    @IsString({ message: "Rótulo inválido" })
    @IsNotEmpty({ message: "Rótulo inválido" })
    label: string

    @IsBoolean({ message: "Tipo de 'checked' inválido" })
    checked: boolean = false;

    @IsBoolean({ message: "Tipo de 'removed' inválido" })
    removed: boolean = false;
}
```

Essa abordagem garante que os dados sejam validados corretamente e que o modelo seja gerado dinamicamente com base nas definições do contrato, tornando o framework altamente adaptável e eficiente para gerenciar validações em vários modelos.

## Validação em Serviços

Além do modelo gerado, os serviços também passam por alterações para garantir que os dados de entrada sejam devidamente validados antes de serem processados ou armazenados. Tanto o serviço gerado pelo `@cmmv/http` quanto a implementação do repositório do `@cmmv/repository` incluem mecanismos de validação integrados usando o módulo `class-validator`.

Essa validação garante que todos os dados recebidos sigam as regras especificadas nos campos do contrato antes que qualquer operação de banco de dados ou lógica de negócios adicional seja executada.

Aqui está um exemplo da função `add` no serviço gerado pelo repositório:

```typescript
async add(item: ITask, req?: any): Promise<TaskEntity> {
    return new Promise(async (resolve, reject) => {
        try{
            Telemetry.start('TaskService::Add', req?.requestId);

            // Converte objeto simples em classe e valida os dados
            const newItem = plainToClass(Task, item, {
                exposeUnsetFields: true,
                enableImplicitConversion: true
            });

            // Valida o newItem com class-validator
            const errors = await validate(newItem, {
                skipMissingProperties: true
            });

            if (errors.length > 0) {
                // Se a validação falhar, retorna os erros
                Telemetry.end('TaskService::Add', req?.requestId);
                reject(errors);
            }
            else {
                // Se a validação passar, prossegue com o repositório
                const result = await Repository.insert<TaskEntity>(
                    TaskEntity, newItem
                );
                Telemetry.end('TaskService::Add', req?.requestId);
                resolve(result);
            }
        }
        catch(e){
            Telemetry.end('TaskService::Add', req?.requestId);
            console.log(e);
            reject(e);
        }
    });
}
```

* **Etapa de Validação:** Antes de realizar qualquer operação no banco de dados, os dados de entrada são convertidos em uma instância de classe usando `plainToClass` e, em seguida, validados usando `validate` do `class-validator`.

* **Tratamento de Erros:** Se a validação falhar, o processo é interrompido e os erros são retornados ao chamador sem prosseguir para a operação no banco de dados.

* **Rastreamento de Telemetria:** As funções de telemetria rastreiam o início e o fim da operação `add`, garantindo que todas as ações sejam monitoradas para desempenho e registro.

* **Inserção no Repositório:** Uma vez que a validação é aprovada, o método `Repository.insert` insere os dados validados no banco de dados.

Esse mecanismo garante que todos os dados de entrada sejam validados de maneira consistente em todo o sistema, facilitando o gerenciamento e a aplicação de regras de validação no nível do serviço, garantindo a integridade dos dados e reduzindo o risco de dados inválidos serem armazenados ou processados.
