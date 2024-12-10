# Transform

In CMMV, you can ``transform`` data within contracts using the transform parameter in the field decorator. The system utilizes the ``class-transformer`` ([NPM](https://www.npmjs.com/package/class-transformer)) library to handle these transformations. This is particularly useful for converting input data into the appropriate format for entity and model classes, which are necessary for interactions with repositories, among other functionalities.

* **``exclude:``** Removes the field during serialization or deserialization.
* **``toClassOnly:``** Ensures that the transformation is only applied when converting to a class instance (useful for data integrity).
* **``transform:``** Allows for custom transformation logic such as encrypting, hashing, adding metadata, or converting dates.

These transforms enable seamless handling of complex data formats like hashing passwords, generating timestamps, and converting or formatting incoming/outgoing data. Below are a few examples:

```typescript
import * as crypto from 'crypto';
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'User',
    protoPath: 'src/protos/user.proto',
    protoPackage: 'user',
})
export class UsersContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        validations: ["IsString"],
        // Example of simple transformation
        transform: ({ value }) => value.toUpperCase(),  
    })
    name: string;

    @ContractField({
        protoType: 'string',
        // This field will be excluded in output
        exclude: false,  
        // Will be processed only when converting to class
        toClassOnly: true,  
        transform: ({ value }) => 
            crypto.createHash('sha256')
            .update(value).digest('hex'),  // Hash the value
    })
    password: string;
}
```

<br/>

* **Data Security:** You can easily encrypt or hash sensitive fields, such as passwords.
* **Data Formatting:** Automatically format dates, strings, or numbers before they are stored in a repository.
* **Flexibility:** You have full control over how input data is transformed, ensuring compatibility with backend services and databases.

These transforms ensure that your contract and data structure remain flexible while allowing control over how data is processed during the request-response lifecycle.

## Input Data

Assume you have the following plain JavaScript object as input data:

```json
{
    "name": "john doe",
    "password": "mypassword"
}
```

Here’s what happens when plainToClass(UsersContract, input) is applied:

```typescript
{
    // Transformed to uppercase as specified
    name: "JOHN DOE",
    // Not included in output due to `exclude` and `toClassOnly` options   
    password: undefined 
}
```

<br/>

* **Name Transformation:**
* The ``name`` field was converted to uppercase due to the ``transform`` function applied: ``value.toUpperCase()``.
* **Password Transformation:**
* The ``password`` field was hashed using SHA-256, but it was excluded from the resulting class because of the ``exclude: true`` and ``toClassOnly: true`` options. This means the password is hashed when converting to the class instance, but it's not included in the serialized output.

**Example Without Exclude**

If we remove the ``exclude: true`` option for password, it will still be hashed but included in the output:

```typescript
{
    // Transformed to uppercase
    name: "JOHN DOE",   
    // Hashed password
    password: "5e884898da28047151d0e56f..."  
}
```

In this case, the ``transform`` function is applied, and the password is hashed, but it is not excluded from the output anymore. The ``toClassOnly: true`` ensures the transformation is only applied when converting to a class.