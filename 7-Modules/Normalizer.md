# Normalizer

The ``@cmmv/normalizer`` module provides a robust and efficient way to normalize data from various formats (JSON, XML, YML) using schemas, transformations, and validations. Designed for scalability, it uses streams to process large files with minimal memory consumption. The module seamlessly integrates with other ``@cmmv`` modules, maintaining data consistency through models and contracts. Additionally, it supports tokenization for sensitive data using the ``@cmmv/encryptor``, allowing secure storage of encrypted objects and strings.

## Features

* **Data Normalization:** Normalize and validate data using customizable schemas with transform and validation rules.
* **Stream-Based Processing:** Efficiently handle large files (JSON, XML, YML) with low memory usage.
* **Tokenization:** Encrypt sensitive fields using ``@cmmv/encryptor`` with AES-256 and elliptic curve key generation.
* **Seamless Integration:** Works with @cmmv models and contracts to ensure consistent data handling.
* **Extensibility:** Easily extend with custom transformations and validators.

## Installation

To install the @cmmv/normalizer module:

```bash
$ pnpm add @cmmv/normalizer @cmmv/encryptor
```

## Example 

The following example demonstrates how to normalize a large JSON file using ``JSONParser`` and a schema with custom transformations and validations.

```typescript
import {
    JSONParser,
    AbstractParserSchema,
    ToLowerCase,
    Tokenizer,
    ToDate,
    ToObjectId,
} from '@cmmv/normalizer';

import { CustomersContract } from '../src/contracts/customers.contract';
import { Customers } from '../src/models/customers.model';
import { ECKeys } from '@cmmv/encryptor';

// Generate encryption keys
const keys = ECKeys.generateKeys();
const tokenizer = Tokenizer({
    publicKey: ECKeys.getPublicKey(keys),
});

// Define schema
class CustomerParserSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',
            transform: [ToObjectId],
        },
        name: { to: 'name' },
        email: {
            to: 'email',
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
            transform: [ToLowerCase],
        },
        phoneNumber: {
            to: 'phone',
            transform: [tokenizer],
        },
        registrationDate: {
            to: 'createdAt',
            transform: [ToDate],
        },
    };
}

// Parse JSON file
new JSONParser({
    contract: CustomersContract,
    schema: CustomerParserSchema,
    model: Customers,
    input: './sample/large-customers.json',
})
.pipe(async data => {
    console.log(data);
})
.once('end', () => console.log('End process!'))
.once('error', (error) => console.error(error))
.start();
```

## Custom Parser

Below is an implementation of a custom CSV parser that follows the same structure and methodology as your JSONParser. It processes large CSV files line by line using streams, ensuring low memory usage and efficient processing:

```typescript
import * as fs from 'node:fs';
import * as path from 'node:path';
import * as readline from 'node:readline';
import { AbstractParser, ParserOptions } from '@cmmv/normalizer';

export class CSVParser extends AbstractParser {
    constructor(options?: ParserOptions) {
        super();
        this.options = options;
    }

    public override start() {
        const inputPath = path.resolve(this.options.input);

        if (fs.existsSync(inputPath)) {
            const readStream = fs.createReadStream(inputPath);

            const rl = readline.createInterface({
                input: readStream,
                crlfDelay: Infinity, // Handle all newlines (Windows, Unix)
            });

            let headers: string[] | null = null;

            rl.on('line', (line) => {
                if (!headers) {
                    // First line contains headers
                    headers = line.split(',');
                    return;
                }

                // Map CSV columns to header fields
                const values = line.split(',');
                const record = headers.reduce((acc, header, index) => {
                    acc[header.trim()] = values[index]?.trim();
                    return acc;
                }, {});

                // Process each record
                this.processData.call(this, record);
            });

            rl.on('close', this.finalize.bind(this));
            rl.on('error', (error) => this.error.bind(this, error));
        } else {
            console.error(`File not found: ${inputPath}`);
        }
    }
}
```

### Parsing a CSV File

Here’s how you can use the CSVParser to parse a CSV file and normalize its data:

```csv
id,name,email,phoneNumber,registrationDate
1,John Doe,john.doe@example.com,1234567890,2023-11-01
2,Jane Smith,jane.smith@example.com,0987654321,2023-11-15
```

**Schema and Parser Usage**

```typescript
import { CSVParser } from './parsers/csv.parser';
import { AbstractParserSchema, ToDate, ToLowerCase } from '@cmmv/normalizer';

class CustomerParserSchema extends AbstractParserSchema {
    public field = {
        id: { to: 'id' },
        name: { to: 'name' },
        email: {
            to: 'email',
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
            transform: [ToLowerCase],
        },
        phoneNumber: { to: 'phone' },
        registrationDate: {
            to: 'createdAt',
            transform: [ToDate],
        },
    };
}

new CSVParser({
    schema: CustomerParserSchema,
    model: null, // Optional: Use a model for further validation/mapping
    input: './sample/customers.csv',
})
.pipe(async (data) => {
    console.log(data); // Normalized data
})
.once('end', () => console.log('Parsing completed!'))
.once('error', (error) => console.error('Error during parsing:', error))
.start();
```

**Output** 

```json
{
    "id": "1",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "phone": "1234567890",
    "createdAt": "2023-11-01T00:00:00.000Z"
}
{
    "id": "2",
    "name": "Jane Smith",
    "email": "jane.smith@example.com",
    "phone": "0987654321",
    "createdAt": "2023-11-15T00:00:00.000Z"
}
```

<br/>

* **id:** The id field is directly mapped from the CSV column without any transformation.
* **name:** The name field is also directly mapped.
* **email:** The email field is validated using the regex and converted to lowercase by the ToLowerCase transformation.
* **phone:** The phoneNumber field from the CSV is mapped to phone.
* **createdAt:** The registrationDate field is converted to a JavaScript Date object using the ToDate transformation.

## Schemas

Schemas in the `@cmmv/normalizer` module define how data should be parsed, transformed, and validated. They act as the blueprint for mapping raw data into structured and normalized objects, ensuring consistency, accuracy, and security across the application.

1. **Field Mapping**: Maps raw data fields to desired object properties using the `to` key.
2. **Transformations**: Applies a series of transformations to convert data into the desired format.
3. **Validation**: Validates raw data fields against regular expressions or other criteria to ensure data integrity.
4. **Security**: Supports sensitive data handling via tokenization or encryption.

Schemas are defined by extending the `AbstractParserSchema` class. Each field in the schema corresponds to a property in the normalized object.

```typescript
class CustomerParserSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',                           // Maps the raw field to 'id'
            transform: [ToInt, ToObjectId],     // Converts to integer, then to ObjectId
        },
        name: { 
            to: 'name'                          // Maps the raw field to 'name'
        },
        email: {
            to: 'email',                        // Maps the raw field to 'email'
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/, // Validates email format
            transform: [ToLowerCase],           // Converts email to lowercase
        },
        phoneNumber: {
            to: 'phone',                        // Maps the raw field to 'phone'
            transform: [tokenizer],             // Applies tokenization for sensitive data
        },
        registrationDate: {
            to: 'createdAt',                    // Maps the raw field to 'createdAt'
            transform: [ToDate],                // Converts to a JavaScript Date object
        },
    };
}
```

## Normalizers

Normalizers in the ``@cmmv/normalizer`` module are responsible for converting and transforming data types during the parsing process. They are defined in the schema under the ``transform`` property and are executed sequentially. This sequential processing allows for multiple transformations to be chained together, ensuring flexibility and precision.

For example, if an ID is provided as a string and needs to be converted to a number before being transformed into an ObjectId, you can chain the necessary normalizers in the desired order.

| **Normalizer**   | **Description**                                                         | **Example**                                                   |
|-------------------|-------------------------------------------------------------------------|---------------------------------------------------------------|
| **ToBoolean**     | Converts a value to a boolean.                                         | `"true" → true`, `"false" → false`, `1 → true`, `0 → false`   |
| **ToDate**        | Converts a value to a JavaScript `Date` object.                        | `"2023-11-15" → `Date("2023-11-15T00:00:00.000Z")`            |
| **ToFixed**       | Converts a number to a string with a fixed number of decimal places.   | `123.456 → "123.46"` (if fixed to 2 decimal places)           |
| **ToFloat**       | Converts a value to a floating-point number.                           | `"123.45" → 123.45`                                           |
| **ToInt**         | Converts a value to an integer.                                        | `"123.45" → 123`                                              |
| **Tokenizer**     | Encrypts sensitive data using a tokenizer.                             | `"123-45-6789" → Encrypted token`                            |
| **ToLowerCase**   | Converts a string to lowercase.                                        | `"John Doe" → "john doe"`                                     |
| **ToObjectId**    | Converts a value to a MongoDB `ObjectId`.                              | `"507f1f77bcf86cd799439011" → ObjectId("507f1f77...")`        |
| **ToString**      | Converts a value to a string.                                          | `123 → "123"`                                                 |
| **ToUpperCase**   | Converts a string to uppercase.                                        | `"John Doe" → "JOHN DOE"`                                     |

### Example Usage

In a schema, you can define a chain of normalizers like this:

```typescript
class ExampleSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',
            transform: [ToInt, ToObjectId], // Converts string to int, then to ObjectId
        },
        name: {
            to: 'name',
            transform: [ToLowerCase], // Converts name to lowercase
        },
        email: {
            to: 'email',
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
            transform: [ToLowerCase], // Converts email to lowercase
        },
    };
}
```

### Recommendations

It is highly recommended to implement additional data validations and transformations directly within the **contract**. This approach allows for a robust and centralized mechanism to handle complex data requirements, taking full advantage of the capabilities provided by the CMMV framework.

### Benefits of Using Contracts for Validation and Transformation

<br/>

1. **Centralized Logic**: By placing the logic in the contract, it ensures consistency and reusability across the system.
2. **Comprehensive Options**: Contracts offer a wide range of built-in and customizable validation and transformation methods.
3. **Serialization Support**: The contract’s serialization layer leverages `class-transformer` and `class-validator`, enabling automatic handling of type conversions and validation errors.

### Key Tools for Implementation

<br/>

- **`class-transformer`**: A library for transforming plain objects into class instances and vice versa. It enables the definition of serialization rules directly within the contract fields.
- **`class-validator`**: A library for validating JavaScript objects using decorators. It allows the enforcement of validation rules at the contract level.

```typescript
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'Customer',
    protoPath: 'src/protos/customer.proto',
    protoPackage: 'customer',
})
export class CustomerContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [
            { type: 'IsString', message: 'ID must be a string.' },
            { type: 'IsNotEmpty', message: 'ID cannot be empty.' },
        ],
        transform: [{ type: 'ToString' }],
    })
    id: string;

    @ContractField({
        protoType: 'string',
        validations: [
            { type: 'IsString', message: 'Name must be a string.' },
            { type: 'IsNotEmpty', message: 'Name cannot be empty.' },
        ],
        transform: [{ type: 'ToLowerCase' }],
    })
    name: string;

    @ContractField({
        protoType: 'string',
        validations: [
            { type: 'IsEmail', message: 'Invalid email address.' },
        ],
        transform: [{ type: 'ToLowerCase' }],
    })
    email: string;

    @ContractField({
        protoType: 'date',
        validations: [
            { type: 'IsDate', message: 'Registration date must be a valid date.' },
        ],
        transform: [{ type: 'ToDate' }],
    })
    registrationDate: Date;
}
```

For detailed documentation on available transformations and validations, visit:

* **Transformations:** [https://cmmv.io/docs/contracts/transform](https://cmmv.io/docs/contracts/transform)
* **Validations:** [https://cmmv.io/docs/contracts/validation](https://cmmv.io/docs/contracts/validation)