# Decoradores

Os decoradores personalizados no ``@cmmv`` permitem estender e modificar o comportamento de classes dinamicamente usando metadados com ``Reflect``. Veja como criar seus próprios decoradores:

Use as interfaces do TypeScript ``ClassDecorator``, ``MethodDecorator`` ou ``PropertyDecorator`` para definir seus decoradores personalizados.

```typescript
function CustomClassDecorator(options?: any): ClassDecorator {
    return (target: object) => {
        Reflect.defineMetadata('custom_metadata', options, target);
    };
}
```

Isso armazena metadados (``options``) associados à classe.

Para acessar esses metadados posteriormente, use ``Reflect.getMetadata()``:

```typescript
const metadata = Reflect.getMetadata('custom_metadata', MyClass);
```

## Método Personalizado

Você pode definir decoradores para métodos de forma semelhante:

```typescript
function LogExecutionTime(): MethodDecorator {
    return (target, propertyKey, descriptor: PropertyDescriptor) => {
        const originalMethod = descriptor.value;
        
        descriptor.value = function (...args: any[]) {
            const start = Date.now();
            const result = originalMethod.apply(this, args);
            const end = Date.now();
            console.log(`Tempo de execução: \${end - start}ms`);
            return result;
        };
    };
}
```

Este decorador de método registra o tempo de execução do método.

## Decorador de Propriedade

Para decoradores de propriedade, você pode usar o seguinte padrão:

```typescript
function DefaultValue(value: any): PropertyDecorator {
    return (target, propertyKey: string | symbol) => {
        Reflect.defineMetadata(
            'default_value', value, target, propertyKey
        );
    };
}
```

Posteriormente, recupere o valor padrão:

```typescript
const defaultValue = Reflect.getMetadata(
    'default_value', target, propertyKey
);
```

<br/>

* **Crie o Decorador:** Use ``Reflect.defineMetadata()`` para anexar metadados à classe, método ou propriedade.
* **Use o Decorador:** Aplique o decorador a classes, métodos ou propriedades.
* **Recupere os Metadados:** Use ``Reflect.getMetadata()`` para recuperar os metadados anexados em tempo de execução.

## Decorador Contract

O decorador ``@Contract`` é usado para definir um controlador de contrato com várias opções, como autenticação, cache e caminhos de arquivos proto.

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

* **``controllerName (string):``** Define o nome do controlador a ser gerado.
* **``controllerCustomPath (string, opcional):``** Caminho personalizado para o controlador.
* **``protoPath (string):``** Caminho para o arquivo ``.proto`` do contrato.
* **``protoPackage (string, opcional):``** Nome do pacote para o buffer de protocolo.
* **``directMessage (boolean, opcional):``** Se verdadeiro, habilita mensagens diretas via WebSocket.
* **``generateController (boolean, opcional):``** Gera automaticamente o controlador se configurado como verdadeiro.
* **``generateEntities (boolean, opcional):``** Controla se as entidades serão geradas.
* **``auth (boolean, opcional):``** Habilita autenticação para o controlador.
* **``imports (Array<string>, opcional):``** Especifica importações adicionais necessárias pelo controlador.
* **``cache (CacheOptions, opcional):``** Define o comportamento de cache para o contrato.

## Decorador ContractField

O decorador ``@ContractField`` é usado para declarar campos dentro de um contrato e definir regras de validação, transformações e configurações de banco de dados.

```typescript
@ContractField({
  protoType: 'string',
  unique: true,
  validations: [{ type: 'isEmail', message: 'Formato de e-mail inválido' }]
})
email: string;
```

<br/>

* **``protoType (string):``** Define o tipo de dado no buffer de protocolo (por exemplo, ``string``, ``int32``).
* **``protoRepeated (boolean, opcional):``** Indica se o campo é do tipo repetido (array).
* **``defaultValue (any, opcional):``** Especifica o valor padrão para o campo.
* **``index (boolean, opcional):``** Habilita indexação no banco de dados para o campo.
* **``unique (boolean, opcional):``** Garante que o campo tenha valores únicos.
* **``exclude (boolean, opcional):``** Exclui o campo do contrato gerado.
* **``nullable (boolean, opcional):``** Permite valores nulos.
* **``toClassOnly (boolean, opcional):``** Restringe o mapeamento do campo apenas para a classe, excluindo banco de dados/entidade.
* **``transform (Function, opcional):``** Uma função de transformação personalizada para valores de campo.
* **``validations (ValidationOption[], opcional):``** Define regras de validação, como verificações de tipo ou lógica de validação personalizada.
