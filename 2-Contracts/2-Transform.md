# Transformação

No CMMV, você pode ``transformar`` dados dentro de contratos usando o parâmetro `transform` no decorador de campo. O sistema utiliza a biblioteca ``class-transformer`` ([NPM](https://www.npmjs.com/package/class-transformer)) para lidar com essas transformações. Isso é particularmente útil para converter dados de entrada no formato apropriado para classes de entidade e modelo, que são necessárias para interações com repositórios, entre outras funcionalidades.

* **``exclude:``** Remove o campo durante a serialização ou desserialização.
* **``toClassOnly:``** Garante que a transformação seja aplicada apenas ao converter para uma instância de classe (útil para integridade de dados).
* **``transform:``** Permite lógica de transformação personalizada, como criptografia, hash, adição de metadados ou conversão de datas.

Essas transformações permitem o manejo contínuo de formatos de dados complexos, como hashing de senhas, geração de timestamps e conversão ou formatação de dados de entrada/saída. Abaixo estão alguns exemplos:

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
        // Exemplo de transformação simples
        transform: ({ value }) => value.toUpperCase(),
    })
    name: string;

    @ContractField({
        protoType: 'string',
        // Este campo será excluído na saída
        exclude: false,
        // Será processado apenas ao converter para classe
        toClassOnly: true,
        transform: ({ value }) =>
            crypto.createHash('sha256')
            .update(value).digest('hex'),  // Hash do valor
    })
    password: string;
}
```

<br/>

* **Segurança de Dados:** Você pode facilmente criptografar ou fazer hash de campos sensíveis, como senhas.
* **Formatação de Dados:** Formate automaticamente datas, strings ou números antes de serem armazenados em um repositório.
* **Flexibilidade:** Você tem controle total sobre como os dados de entrada são transformados, garantindo compatibilidade com serviços de backend e bancos de dados.

Essas transformações garantem que seu contrato e estrutura de dados permaneçam flexíveis, permitindo controle sobre como os dados são processados durante o ciclo de requisição-resposta.

## Dados de Entrada

Suponha que você tenha o seguinte objeto JavaScript simples como dado de entrada:

```json
{
    "name": "john doe",
    "password": "mypassword"
}
```

Aqui está o que acontece quando `plainToClass(UsersContract, input)` é aplicado:

```typescript
{
    // Transformado para maiúsculas conforme especificado
    name: "JOHN DOE",
    // Não incluído na saída devido às opções `exclude` e `toClassOnly`
    password: undefined
}
```

<br/>

* **Transformação do Nome:**
  * O campo ``name`` foi convertido para maiúsculas devido à função ``transform`` aplicada: ``value.toUpperCase()``.
* **Transformação da Senha:**
  * O campo ``password`` foi transformado em hash usando SHA-256, mas foi excluído da classe resultante devido às opções ``exclude: true`` e ``toClassOnly: true``. Isso significa que a senha é transformada em hash ao converter para a instância da classe, mas não é incluída na saída serializada.

**Exemplo Sem Exclude**

Se removermos a opção ``exclude: true`` para a senha, ela ainda será transformada em hash, mas será incluída na saída:

```typescript
{
    // Transformado para maiúsculas
    name: "JOHN DOE",
    // Senha com hash
    password: "5e884898da28047151d0e56f..."
}
```

Nesse caso, a função ``transform`` é aplicada, e a senha é transformada em hash, mas não é mais excluída da saída. A opção ``toClassOnly: true`` garante que a transformação seja aplicada apenas ao converter para uma classe.

## toPlain

A partir da versão 0.9, o decorador ``@ContractField`` agora suporta o parâmetro ``toPlain``, que permite reverter a transformação aplicada na função ``transform`` ao serializar o objeto para saída. Esse recurso é particularmente útil para manter compatibilidade com bancos de dados relacionais que não suportam campos de objeto nativamente. Em vez de criar tabelas relacionais e realizar operações JOIN complexas, os dados agora podem ser armazenados como strings e convertidos de volta para objetos de forma transparente ao serem recuperados.

Considere a seguinte definição de campo de contrato:

```typescript
@ContractField({
    protoType: 'string',
    defaultValue: '"[]"',
    objectType: 'string',
    transform: ({ value }) => JSON.stringify(value),
    toPlain: ({ value }) => (value ? JSON.parse(value) : []),
})
groups: Array<string>;
```

Modelo Gerado:

Com o novo recurso ``toPlain``, o modelo TypeScript gerado incluirá os seguintes decoradores:

```typescript
@Expose()
@Transform(({ value }) => JSON.stringify(value), { toClassOnly: true })
@Transform(({ value }) => (value ? JSON.parse(value) : []), { toPlainOnly: true })
groups: string = "[]";
```

* **Compatibilidade com bancos de dados relacionais:** Armazenar arrays ou objetos como strings JSON elimina a necessidade de tabelas adicionais e consultas complexas.

* **Transformação contínua durante a serialização:** Ao serializar o objeto para saída, a transformação `toPlain` garante que a string JSON armazenada seja convertida de volta ao seu formato original.

* **Gerenciamento de dados simplificado:** Não é necessário processamento adicional ao recuperar os dados, pois a transformação é tratada automaticamente.

Esse recurso garante que as aplicações possam lidar com estruturas de dados complexas de forma eficiente, sem comprometer a compatibilidade com bancos de dados relacionais.
