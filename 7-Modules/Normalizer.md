# Normalizador

Repositório: [https://github.com/cmmvio/cmmv-normalizer](https://github.com/cmmvio/cmmv-normalizer)

O módulo ``@cmmv/normalizer`` oferece uma maneira robusta e eficiente de normalizar dados de vários formatos (JSON, XML, YML) usando esquemas, transformações e validações. Projetado para escalabilidade, ele utiliza fluxos (streams) para processar arquivos grandes com consumo mínimo de memória. O módulo se integra perfeitamente a outros módulos ``@cmmv``, mantendo a consistência dos dados por meio de modelos e contratos. Além disso, suporta tokenização para dados sensíveis usando o ``@cmmv/encryptor``, permitindo o armazenamento seguro de objetos e strings criptografados.

## Recursos

* **Normalização de Dados:** Normaliza e valida dados usando esquemas personalizáveis com regras de transformação e validação.
* **Processamento Baseado em Fluxos:** Lida eficientemente com arquivos grandes (JSON, XML, YML) com baixo uso de memória.
* **Tokenização:** Criptografa campos sensíveis usando o ``@cmmv/encryptor`` com AES-256 e geração de chaves de curva elíptica.
* **Integração Perfeita:** Funciona com modelos e contratos do @cmmv para garantir um manuseio consistente dos dados.
* **Extensibilidade:** Pode ser facilmente estendido com transformações e validadores personalizados.

## Instalação

Para instalar o módulo `@cmmv/normalizer`:

```bash
$ pnpm add @cmmv/normalizer @cmmv/encryptor
```

## Exemplo

O exemplo a seguir demonstra como normalizar um arquivo JSON grande usando o ``JSONParser`` e um esquema com transformações e validações personalizadas.

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

// Gera chaves de criptografia
const keys = ECKeys.generateKeys();
const tokenizer = Tokenizer({
    publicKey: ECKeys.getPublicKey(keys),
});

// Define o esquema
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

// Processa o arquivo JSON
new JSONParser({
    contract: CustomersContract,
    schema: CustomerParserSchema,
    model: Customers,
    input: './sample/large-customers.json',
})
.pipe(async data => {
    console.log(data);
})
.once('end', () => console.log('Fim do processo!'))
.once('error', (error) => console.error(error))
.start();
```

## Parser Personalizado

Abaixo está uma implementação de um parser CSV personalizado que segue a mesma estrutura e metodologia do seu ``JSONParser``. Ele processa arquivos CSV grandes linha por linha usando fluxos, garantindo baixo uso de memória e processamento eficiente:

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
                crlfDelay: Infinity, // Lida com todas as quebras de linha (Windows, Unix)
            });

            let headers: string[] | null = null;

            rl.on('line', (line) => {
                if (!headers) {
                    // A primeira linha contém os cabeçalhos
                    headers = line.split(',');
                    return;
                }

                // Mapeia as colunas CSV para os campos do cabeçalho
                const values = line.split(',');
                const record = headers.reduce((acc, header, index) => {
                    acc[header.trim()] = values[index]?.trim();
                    return acc;
                }, {});

                // Processa cada registro
                this.processData.call(this, record);
            });

            rl.on('close', this.finalize.bind(this));
            rl.on('error', (error) => this.error.bind(this, error));
        } else {
            console.error(`Arquivo não encontrado: \${inputPath}`);
        }
    }
}
```

### Processando um Arquivo CSV

Aqui está como você pode usar o ``CSVParser`` para processar um arquivo CSV e normalizar seus dados:

```csv
id,name,email,phoneNumber,registrationDate
1,João Silva,joao.silva@example.com,1234567890,2023-11-01
2,Ana Souza,ana.souza@example.com,0987654321,2023-11-15
```

**Esquema e Uso do Parser**

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
    model: null, // Opcional: Use um modelo para validação/mapeamento adicional
    input: './sample/customers.csv',
})
.pipe(async (data) => {
    console.log(data); // Dados normalizados
})
.once('end', () => console.log('Processamento concluído!'))
.once('error', (error) => console.error('Erro durante o processamento:', error))
.start();
```

**Saída**

```json
{
    "id": "1",
    "name": "João Silva",
    "email": "joao.silva@example.com",
    "phone": "1234567890",
    "createdAt": "2023-11-01T00:00:00.000Z"
}
{
    "id": "2",
    "name": "Ana Souza",
    "email": "ana.souza@example.com",
    "phone": "0987654321",
    "createdAt": "2023-11-15T00:00:00.000Z"
}
```
<br/>

* **id:** O campo `id` é mapeado diretamente da coluna CSV sem transformação.
* **name:** O campo `name` também é mapeado diretamente.
* **email:** O campo `email` é validado usando uma expressão regular e convertido para minúsculas pela transformação `ToLowerCase`.
* **phone:** O campo `phoneNumber` do CSV é mapeado para `phone`.
* **createdAt:** O campo `registrationDate` é convertido em um objeto `Date` do JavaScript usando a transformação `ToDate`.

## Esquemas

Os esquemas no módulo `@cmmv/normalizer` definem como os dados devem ser processados, transformados e validados. Eles funcionam como o blueprint para mapear dados brutos em objetos estruturados e normalizados, garantindo consistência, precisão e segurança em toda a aplicação.

1. **Mapeamento de Campos**: Mapeia campos de dados brutos para propriedades desejadas usando a chave `to`.
2. **Transformações**: Aplica uma série de transformações para converter dados no formato desejado.
3. **Validação**: Valida campos de dados brutos contra expressões regulares ou outros critérios para garantir a integridade dos dados.
4. **Segurança**: Suporta o manuseio de dados sensíveis por meio de tokenização ou criptografia.

Os esquemas são definidos estendendo a classe `AbstractParserSchema`. Cada campo no esquema corresponde a uma propriedade no objeto normalizado.

```typescript
class CustomerParserSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',                           // Mapeia o campo bruto para 'id'
            transform: [ToInt, ToObjectId],     // Converte para inteiro, depois para ObjectId
        },
        name: {
            to: 'name'                          // Mapeia o campo bruto para 'name'
        },
        email: {
            to: 'email',                        // Mapeia o campo bruto para 'email'
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/, // Valida o formato do email
            transform: [ToLowerCase],           // Converte o email para minúsculas
        },
        phoneNumber: {
            to: 'phone',                        // Mapeia o campo bruto para 'phone'
            transform: [tokenizer],             // Aplica tokenização para dados sensíveis
        },
        registrationDate: {
            to: 'createdAt',                    // Mapeia o campo bruto para 'createdAt'
            transform: [ToDate],                // Converte para um objeto Date do JavaScript
        },
    };
}
```

## Normalizadores

Os normalizadores no módulo ``@cmmv/normalizer`` são responsáveis por converter e transformar tipos de dados durante o processo de parsing. Eles são definidos no esquema sob a propriedade ``transform`` e são executados sequencialmente. Esse processamento sequencial permite que múltiplas transformações sejam encadeadas, garantindo flexibilidade e precisão.

Por exemplo, se um ID for fornecido como string e precisar ser convertido para um número antes de ser transformado em um ObjectId, você pode encadear os normalizadores necessários na ordem desejada.

| **Normalizador**   | **Descrição**                                                         | **Exemplo**                                                   |
|-------------------|----------------------------------------------------------------------|-------------------------------------------------------------|
| **ToBoolean**     | Converte um valor em booleano.                                       | `"true" → true`, `"false" → false`, `1 → true`, `0 → false` |
| **ToDate**        | Converte um valor em um objeto `Date` do JavaScript.                  | `"2023-11-15" → `Date("2023-11-15T00:00:00.000Z")`          |
| **ToFixed**       | Converte um número em uma string com um número fixo de casas decimais.| `123.456 → "123.46"` (se fixado em 2 casas decimais)        |
| **ToFloat**       | Converte um valor em um número de ponto flutuante.                    | `"123.45" → 123.45`                                         |
| **ToInt**         | Converte um valor em um inteiro.                                      | `"123.45" → 123`                                            |
| **Tokenizer**     | Criptografa dados sensíveis usando um tokenizador.                    | `"123-45-6789" → Token criptografado`                       |
| **ToLowerCase**   | Converte uma string para minúsculas.                                  | `"João Silva" → "joão silva"`                               |
| **ToObjectId**    | Converte um valor em um `ObjectId` do MongoDB.                        | `"507f1f77bcf86cd799439011" → ObjectId("507f1f77...")`      |
| **ToString**      | Converte um valor em uma string.                                      | `123 → "123"`                                               |
| **ToUpperCase**   | Converte uma string para maiúsculas.                                  | `"João Silva" → "JOÃO SILVA"`                               |

### Exemplo de Uso

Em um esquema, você pode definir uma cadeia de normalizadores como esta:

```typescript
class ExampleSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',
            transform: [ToInt, ToObjectId], // Converte string para inteiro, depois para ObjectId
        },
        name: {
            to: 'name',
            transform: [ToLowerCase], // Converte o nome para minúsculas
        },
        email: {
            to: 'email',
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
            transform: [ToLowerCase], // Converte o email para minúsculas
        },
    };
}
```

### Recomendações

É altamente recomendável implementar validações e transformações de dados adicionais diretamente no **contrato**. Essa abordagem permite um mecanismo robusto e centralizado para lidar com requisitos de dados complexos, aproveitando ao máximo as capacidades fornecidas pelo framework CMMV.

### Benefícios de Usar Contratos para Validação e Transformação

<br/>

1. **Lógica Centralizada**: Ao colocar a lógica no contrato, garante-se consistência e reutilização em todo o sistema.
2. **Opções Abrangentes**: Os contratos oferecem uma ampla gama de métodos de validação e transformação embutidos e personalizáveis.
3. **Suporte a Serialização**: A camada de serialização do contrato utiliza `class-transformer` e `class-validator`, permitindo o manuseio automático de conversões de tipo e erros de validação.

### Ferramentas-Chave para Implementação

<br/>

- **`class-transformer`**: Uma biblioteca para transformar objetos simples em instâncias de classes e vice-versa. Permite a definição de regras de serialização diretamente nos campos do contrato.
- **`class-validator`**: Uma biblioteca para validar objetos JavaScript usando decoradores. Permite a imposição de regras de validação no nível do contrato.

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
            { type: 'IsString', message: 'O ID deve ser uma string.' },
            { type: 'IsNotEmpty', message: 'O ID não pode estar vazio.' },
        ],
        transform: [{ type: 'ToString' }],
    })
    id: string;

    @ContractField({
        protoType: 'string',
        validations: [
            { type: 'IsString', message: 'O nome deve ser uma string.' },
            { type: 'IsNotEmpty', message: 'O nome não pode estar vazio.' },
        ],
        transform: [{ type: 'ToLowerCase' }],
    })
    name: string;

    @ContractField({
        protoType: 'string',
        validations: [
            { type: 'IsEmail', message: 'Endereço de email inválido.' },
        ],
        transform: [{ type: 'ToLowerCase' }],
    })
    email: string;

    @ContractField({
        protoType: 'date',
        validations: [
            { type: 'IsDate', message: 'A data de registro deve ser uma data válida.' },
        ],
        transform: [{ type: 'ToDate' }],
    })
    registrationDate: Date;
}
```

Para documentação detalhada sobre transformações e validações disponíveis, visite:

* **Transformações:** [https://cmmv.io/docs/contracts/transform](https://cmmv.io/docs/contracts/transform)
* **Validações:** [https://cmmv.io/docs/contracts/validation](https://cmmv.io/docs/contracts/validation)
