# Decorators

Custom decorators in ``@cmmv`` allow you to extend and modify class behavior dynamically by using ``Reflect`` metadata. Here's how you can create your own decorators:

Use TypeScript's ``ClassDecorator``, ``MethodDecorator``, or ``PropertyDecorator`` interfaces to define your custom decorators.

```typescript
function CustomClassDecorator(options?: any): ClassDecorator {
    return (target: object) => {
        Reflect.defineMetadata('custom_metadata', options, target);
    };
}
```

This stores metadata (``options``) associated with the class.

To access this metadata later, use ``Reflect.getMetadata()``:

```typescript
const metadata = Reflect.getMetadata('custom_metadata', MyClass);
```

## Custom Method

You can define decorators for methods similarly:

```typescript
function LogExecutionTime(): MethodDecorator {
    return (target, propertyKey, descriptor: PropertyDescriptor) => {
        const originalMethod = descriptor.value;
        
        descriptor.value = function (...args: any[]) {
            const start = Date.now();
            const result = originalMethod.apply(this, args);
            const end = Date.now();
            console.log(`Execution time: ${end - start}ms`);
            return result;
        };
    };
}
```

This method decorator logs the execution time of the method.

## Property Decorator

For property decorators, you can use the following pattern:

```typescript
function DefaultValue(value: any): PropertyDecorator {
    return (target, propertyKey: string | symbol) => {
        Reflect.defineMetadata(
            'default_value', value, target, propertyKey
        );
    };
}
```

Later, retrieve the default value:

```typescript
const defaultValue = Reflect.getMetadata(
    'default_value', target, propertyKey
);
```

<br/>

* **Create the Decorator:** Use ``Reflect.defineMetadata()`` to attach metadata to the class, method, or property.
* **Use the Decorator:** Apply the decorator to classes, methods, or properties.
* **Retrieve Metadata:** Use ``Reflect.getMetadata()`` to retrieve the attached metadata at runtime.

## Contract Decorator

The ``@Contract`` decorator is used to define a contract controller with various options, such as authentication, caching, and proto file paths.

```typescript
@Contract({
  controllerName: 'UserContract',
  protoPath: 'user.proto',
  generateEntities: true,
  auth: true,
  cache: { key: 'userCache', ttl: 300 }
})
class UserContract {}
```

<br/>

* **``controllerName (string):``** Defines the name of the controller to be generated.
* **``controllerCustomPath (string, optional):``** Custom path for the controller.
* **``protoPath (string):``** Path to the .proto file for the contract.
* **``protoPackage (string, optional):``** Package name for the protocol buffer.
* **``directMessage (boolean, optional):``** If true, enables direct messaging via WebSocket.
* **``generateController (boolean, optional):``** Automatically generates the controller if set to true.
* **``generateEntities (boolean, optional):``** Controls whether entities are generated.
* **``auth (boolean, optional):``** Enables authentication for the controller.
* **``imports (Array<string>, optional):``** Specifies additional imports required by the controller.
* **``cache (CacheOptions, optional):``** Defines caching behavior for the contract.

## ContractField Decorator

The ``@ContractField`` decorator is used to declare fields within a contract and define validation rules, transformations, and database configurations.

```typescript
@ContractField({
  protoType: 'string',
  unique: true,
  validations: [{ type: 'isEmail', message: 'Invalid email format' }]
})
email: string;
```

<br/>

* **``protoType (string):``** Defines the data type in the protocol buffer (e.g., ``string``, ``int32``).
* **``protoRepeated (boolean, optional):``** Indicates if the field is a repeated (array) type.
* **``defaultValue (any, optional):``** Specifies the default value for the field.
* **``index (boolean, optional):``** Enables database indexing for the field.
* **``unique (boolean, optional):``** Ensures the field has unique values.
* **``exclude (boolean, optional):``** Excludes the field from the generated contract.
* **``nullable (boolean, optional):``** Allows null values.
toClassOnly (boolean, optional): Restricts field mapping to the class, excluding database/entity.
* **``transform (Function, optional):``** A custom transformation function for field values.
* **``validations (ValidationOption[], optional):``** Defines validation rules, such as type checks or custom validation logic.

