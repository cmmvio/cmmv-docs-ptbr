# Validation

CMMV supports automatic contract validation using the ``class-validator`` ([NPM](https://www.npmjs.com/package/class-validator)) module. During data insertion and update operations, CMMV can apply validation rules to contract fields, ensuring that data meets the defined criteria before being processed. You can specify validation rules for each field by adding the validations parameter.

These validations are executed automatically during ``insert`` and ``update`` operations, and the system will return errors if any validation fails.

Below is an example of a ``TasksContract``, where each field is validated using different rules:

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
            message: "Label cannot be empty"
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

## Structure 

Each validation rule is configured using the ``validations`` parameter, which contains an array of validation objects. Each object can include the following properties:

* **type:** The type of validation to apply, such as IsString, IsNotEmpty, IsBoolean, etc.
* **message (optional):** A custom error message that will be shown when the validation fails.
* **context (optional):** Additional context information that can be included with the validation.

## Validation Types

``class-validator`` provides a wide range of validation types that can be applied to contract fields. Some common types are:

| **Decorator**                | **Description**                                                                                                                                                                   |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IsDefined        | Checks if value is defined (!== undefined, !== null). This is the only decorator that ignores skipMissingProperties option.                                                       |
| IsOptional                 | Checks if given value is empty (=== null, === undefined) and if so, ignores all the validators on the property.                                                                  |
| Equals      | Checks if value equals ("===") comparison.                                                                                                                                       |
| NotEquals   | Checks if value not equal ("!==") comparison.                                                                                                                                    |
| IsEmpty                    | Checks if given value is empty (=== '', === null, === undefined).                                                                                                                |
| IsNotEmpty                 | Checks if given value is not empty (!== '', !== null, !== undefined).                                                                                                            |
| IsIn          | Checks if value is in an array of allowed values.                                                                                                                                |
| IsNotIn       | Checks if value is not in an array of disallowed values.                                                                                                                         |
| IsBoolean                  | Checks if a value is a boolean.                                                                                                                                                  |
| IsDate                     | Checks if the value is a date.                                                                                                                                                   |
| IsString                   | Checks if the value is a string.                                                                                                                                                 |
| IsNumber | Checks if the value is a number.                                                                                                                                       |
| IsInt                      | Checks if the value is an integer number.                                                                                                                                        |
| IsArray                    | Checks if the value is an array.                                                                                                                                                |
| IsEnum       | Checks if the value is a valid enum.                                                                                                                                             |
| IsDivisibleBy   | Checks if the value is a number that's divisible by another.                                                                                                                     |
| IsPositive                 | Checks if the value is a positive number greater than zero.                                                                                                                      |
| IsNegative                 | Checks if the value is a negative number smaller than zero.                                                                                                                      |
| Min             | Checks if the given number is greater than or equal to given number.                                                                                                             |
| Max             | Checks if the given number is less than or equal to given number.                                                                                                                |
| MinDate | Checks if the value is a date that's after the specified date.                                                                                                            |
| MaxDate | Checks if the value is a date that's before the specified date.                                                                                                           |
| IsBooleanString            | Checks if a string is a boolean (e.g. is "true" or "false" or "1", "0").                                                                                                         |
| IsDateString               | Alias for @IsISO8601().                                                                                                                                                          |
| IsNumberString | Checks if a string is a number.                                                                                                                                  |
| Contains       | Checks if the string contains the seed.                                                                                                                                          |
| NotContains    | Checks if the string not contains the seed.                                                                                                                                      |
| IsAlpha                    | Checks if the string contains only letters (a-zA-Z).                                                                                                                             |
| IsAlphanumeric             | Checks if the string contains only letters and numbers.                                                                                                                          |
| IsDecimal | Checks if the string is a valid decimal value.                                                                                                                          |
| IsAscii                    | Checks if the string contains ASCII chars only.                                                                                                                                  |
| IsBase32                   | Checks if a string is base32 encoded.                                                                                                                                           |
| IsBase58                   | Checks if a string is base58 encoded.                                                                                                                                           |
| IsBase64 | Checks if a string is base64 encoded.                                                                                                                                    |
| IsIBAN                     | Checks if a string is an IBAN (International Bank Account Number).                                                                                                              |
| IsBIC                      | Checks if a string is a BIC (Bank Identification Code) or SWIFT code.                                                                                                           |
| IsByteLength | Checks if the string's length (in bytes) falls in a range.                                                                                                          |
| IsCreditCard               | Checks if the string is a credit card.                                                                                                                                          |
| IsCurrency | Checks if the string is a valid currency amount.                                                                                                                     |
| IsEthereumAddress          | Checks if the string is an Ethereum address using basic regex. Does not validate address checksums.                                                                              |
| IsDataURI                  | Checks if the string is a data uri format.                                                                                                                                       |
| IsEmail | Checks if the string is an email.                                                                                                                                           |
| IsFQDN | Checks if the string is a fully qualified domain name (e.g. domain.com).                                                                                                      |
| IsHexColor                 | Checks if the string is a hexadecimal color.                                                                                                                                     |
| IsISBN | Checks if the string is an ISBN (version 10 or 13).                                                                                                                           |
| IsUUID | Checks if the string is a UUID (version 3, 4, 5 or all).                                                                                                                       |
| ArrayContains | Checks if array contains all values from the given array of values.                                                                                                              |
| ArrayNotEmpty              | Checks if given array is not empty.                                                                                                                                             |
| ArrayMinSize    | Checks if the array's length is greater than or equal to the specified number.                                                                                                   |
| @Allow()                      | Prevents stripping off the property when no other constraint is specified for it.                                                                                               |

When a contract is used in insert or update operations, CMMV automatically converts the incoming data to class instances, applies any transformations, and performs the validation according to the rules defined in the contract. If any field does not meet the validation criteria, the process is halted, and a list of validation errors is returned.

This process ensures that data is always compliant with the rules defined in the contract, protecting the integrity of the system.

## Validation in Models

When validations are defined in the contract fields, the CMMV framework automatically adds the corresponding class-validator decorators to the generated model. These decorators perform runtime validation to ensure the data adheres to the defined rules. The model generation process inserts the correct validation decorators above each field, along with any specified validation options, such as custom error messages.

Here's an example of how to define a contract with validations:

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

When the contract is processed by the framework, the following model is automatically generated. The decorators from ``class-validator`` are added to each field to perform validation according to the specified rules:

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

This approach ensures that data is validated correctly and that the model is generated dynamically based on the contract definitions, making the framework highly adaptable and efficient for managing validations across various models.

## Validation in Services

In addition to the generated model, services also undergo changes to ensure that the data being input is properly validated before it's processed or stored. Both the service generated by ``@cmmv/http`` and the repository implementation from ``@cmmv/repository`` include built-in validation mechanisms using the ``class-validator`` module.

This validation ensures that all incoming data adheres to the rules specified in the contract fields before any database operations or further business logic are executed.

Here’s an example of the add function in the service that is generated by the repository:

```typescript
async add(item: ITask, req?: any): Promise<TaskEntity> {
    return new Promise(async (resolve, reject) => {
        try{
            Telemetry.start('TaskService::Add', req?.requestId);
                    
            // Convert plain object to class and validate the data
            const newItem = plainToClass(Task, item, { 
                exposeUnsetFields: true,
                enableImplicitConversion: true
            }); 

            // Validate the newItem with class-validator
            const errors = await validate(newItem, { 
                skipMissingProperties: true 
            });
            
            if (errors.length > 0) {
                // If validation fails, return the errors
                Telemetry.end('TaskService::Add', req?.requestId);
                reject(errors);
            } 
            else {                   
                // If validation passes, proceed with the repository
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

* **Validation Step:** Before performing any database operations, the input data is converted into a class instance using ``plainToClass`` and then validated using ``validate`` from ``class-validator``.

* **Error Handling:** If validation fails, the process halts, and the errors are returned to the caller without proceeding to the database operation.

* **Telemetry Tracking:** Telemetry functions track the beginning and end of the ``add`` operation, ensuring that all actions are monitored for performance and logging.

* **Repository Insert:** Once the validation passes, the ``Repository.insert`` method inserts the validated data into the database.

This mechanism ensures that all input data is validated in a consistent manner across the system, making it easy to manage and enforce validation rules at the service level, guaranteeing data integrity and reducing the risk of invalid data being stored or processed.