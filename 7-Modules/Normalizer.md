# Normalizador

O módulo ``@cmmv/normalizer`` oferece uma maneira robusta e eficiente de normalizar dados de vários formatos (JSON, XML, YML) utilizando esquemas, transformações e validações. Projetado para escalabilidade, utiliza streams para processar grandes arquivos com consumo mínimo de memória. O módulo se integra perfeitamente com outros módulos ``@cmmv``, mantendo a consistência dos dados por meio de modelos e contratos. Além disso, suporta tokenização para dados sensíveis usando o ``@cmmv/encryptor``, permitindo armazenamento seguro de objetos e strings criptografados.

## Recursos

* **Normalização de Dados:** Normaliza e valida dados usando esquemas personalizáveis com regras de transformação e validação.
* **Processamento Baseado em Streams:** Manipula grandes arquivos (JSON, XML, YML) de forma eficiente com baixo uso de memória.
* **Tokenização:** Criptografa campos sensíveis utilizando ``@cmmv/encryptor`` com AES-256 e geração de chaves de curva elíptica.
* **Integração Transparente:** Funciona com modelos e contratos ``@cmmv`` para garantir a manipulação consistente de dados.
* **Extensibilidade:** Facilmente extensível com transformações e validadores personalizados.

## Instalação

Para instalar o módulo ``@cmmv/normalizer``:

```bash
$ pnpm add @cmmv/normalizer @cmmv/encryptor
```

## Exemplo 

O exemplo a seguir demonstra como normalizar um grande arquivo JSON usando ``JSONParser`` e um esquema com transformações e validações personalizadas.

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

// Gerar chaves de criptografia
const keys = ECKeys.generateKeys();
const tokenizer = Tokenizer({
    publicKey: ECKeys.getPublicKey(keys),
});

// Definir esquema
class CustomerParserSchema extends AbstractParserSchema {
    public field = {
        id: {
            to: 'id',
            transform: [ToObjectId],
        },
        name: { to: 'name' },
        email: {
            to: 'email',
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$/,
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

// Analisar arquivo JSON
new JSONParser({
    contract: CustomersContract,
    schema: CustomerParserSchema,
    model: Customers,
    input: './sample/large-customers.json',
})
.pipe(async data => {
    console.log(data);
})
.once('end', () => console.log('Processo concluído!'))
.once('error', (error) => console.error(error))
.start();
```

## Parser Personalizado

Abaixo está uma implementação de um parser CSV personalizado que segue a mesma estrutura e metodologia do ``JSONParser``. Ele processa grandes arquivos CSV linha por linha usando streams, garantindo baixo uso de memória e processamento eficiente:

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
                crlfDelay: Infinity, // Suporta todos os tipos de nova linha (Windows, Unix)
            });

            let headers: string[] | null = null;

            rl.on('line', (line) => {
                if (!headers) {
                    // Primeira linha contém os cabeçalhos
                    headers = line.split(',');
                    return;
                }

                // Mapear colunas do CSV para campos dos cabeçalhos
                const values = line.split(',');
                const record = headers.reduce((acc, header, index) => {
                    acc[header.trim()] = values[index]?.trim();
                    return acc;
                }, {});

                // Processar cada registro
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

### Analisando um Arquivo CSV

Veja como usar o ``CSVParser`` para analisar um arquivo CSV e normalizar seus dados:

```csv
id,name,email,phoneNumber,registrationDate
1,John Doe,john.doe@example.com,1234567890,2023-11-01
2,Jane Smith,jane.smith@example.com,0987654321,2023-11-15
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
            validation: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$/,
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
.once('end', () => console.log('Análise concluída!'))
.once('error', (error) => console.error('Erro durante a análise:', error))
.start();
```

**Saída**

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

Para mais informações sobre transformações e validações, visite os documentos oficiais do CMMV.
