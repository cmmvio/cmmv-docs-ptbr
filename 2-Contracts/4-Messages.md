# Mensagens

A partir da versão 0.9, o framework CMMV introduz o decorador ``@ContractMessage``, que permite aos desenvolvedores definir mensagens de dados estruturadas dentro de contratos. Esse recurso está alinhado com a abordagem do Protocol Buffers (Protobuf), possibilitando a criação de DTOs (Objetos de Transferência de Dados) que definem estruturas de dados usadas em requisições RPC e RESTful, garantindo que apenas os dados necessários sejam transmitidos, sem expor todo o esquema do contrato.

* **Encapsulamento de Estruturas de Requisição e Resposta:** As mensagens permitem a definição explícita dos dados usados para funções específicas, evitando a exposição de dados desnecessários.

* **Geração Automática de DTOs e Interfaces:** O sistema transpila mensagens de contrato em interfaces TypeScript e DTOs, que podem ser usados em controladores gerados, gateways ou qualquer outra parte da aplicação.

* **Segurança e Manutenibilidade Aprimoradas:** Ao estruturar requisições e respostas separadamente, as aplicações podem impor regras rígidas de validação e transformação de dados.

* **Integração com Recursos Existentes do CMMV:** Funciona perfeitamente com recursos existentes do contrato, como transformações, validações e serviços.

O decorador ``@ContractMessage`` é usado dentro de uma classe de contrato para definir estruturas de requisição e resposta explicitamente. Ele aceita os seguintes parâmetros:

```typescript
import * as crypto from 'crypto';

import {
    AbstractContract,
    Contract,
    ContractField,
    ContractMessage,
    ContractService
} from '@cmmv/core';

@Contract({
    controllerName: 'User',
    protoPath: 'src/protos/auth.proto',
    protoPackage: 'auth',
    directMessage: true,
    generateController: false,
    imports: ['crypto'],
    index: [{ name: "idx_user_login", fields: ["username", "password"] }]
})
export class AuthContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        unique: true,
        validations: [{ type: 'IsString', message: 'Nome de usuário inválido' }],
        transform: ({ value }) =>
            crypto.createHash('sha1').update(value).digest('hex'),
    })
    username: string;

    @ContractField({
        protoType: 'string',
        validations: [
            {
                type: 'IsString',
                message: 'Senha inválida',
            },
        ],
        transform: ({ value }) =>
            crypto.createHash('sha256').update(value).digest('hex'),
    })
    password: string;

    @ContractMessage({
        name: 'LoginRequest',
        properties: {
            username: { type: 'string', required: true },
            password: { type: 'string', required: true },
        },
    })
    LoginRequest: {
        username: string;
        password: string;
    };

    @ContractMessage({
        name: 'LoginResponse',
        properties: {
            success: { type: 'bool', required: true },
            token: { type: 'string', required: false },
            message: { type: 'string', required: false },
        },
    })
    LoginResponse: {
        success: boolean;
        token?: string;
        message?: string;
    };

    @ContractService({
        name: 'Login',
        path: 'login',
        method: 'POST',
        request: 'LoginRequest',
        response: 'LoginResponse',
    })
    Login: Function;
}
```

## DTOs Gerados

O transpilador gerará automaticamente classes DTO e interfaces com base nas mensagens do contrato. Por exemplo, o contrato acima resultará nos seguintes DTOs:

```typescript
export interface LoginRequest {
    username: string;
    password: string;
}

export class LoginRequestDTO implements LoginRequest {
    username: string;
    password: string;

    constructor(partial: Partial<LoginRequestDTO>) {
        Object.assign(this, partial);
    }

    public serialize(){
        return instanceToPlain(this);
    }
}

export interface LoginResponse {
    success: boolean;
    token?: string;
    message?: string;
}

export class LoginResponseDTO implements LoginResponse {
    success: boolean;
    token?: string;
    message?: string;

    constructor(partial: Partial<LoginResponseDTO>) {
        Object.assign(this, partial);
    }

    public serialize(){
        return instanceToPlain(this);
    }
}
```

Esses DTOs serão usados dentro dos controladores e gateways gerados, garantindo segurança de tipo e consistência em toda a aplicação.

A introdução do recurso `@ContractMessage` na versão 0.9 oferece uma maneira estruturada de definir e utilizar estruturas de dados específicas dentro dos contratos do CMMV. Essa melhoria aumenta a segurança dos dados, reduz o tamanho da carga útil e permite uma comunicação eficiente de serviços usando DTOs.

Ao aproveitar esse recurso, os desenvolvedores podem criar aplicações mais manuteníveis, seguras e escaláveis sem complexidade desnecessária.

## Integração com Protobuf

Se o módulo ``@cmmv/protobuf`` estiver presente no projeto, o transpilador gerará automaticamente o arquivo .proto correspondente com base nas mensagens de contrato definidas. A estrutura .proto gerada seguirá as definições do contrato, garantindo compatibilidade com gRPC e outros protocolos de comunicação baseados em Protobuf.

Para o exemplo do ``AuthContract`` fornecido, as seguintes definições Protobuf serão geradas:

```proto
message LoginRequest {
    string username = 1;
    string password = 2;
}

message LoginResponse {
    bool success = 1;
    string token = 2;
    string message = 3;
}
```

Ao utilizar o módulo ``@cmmv/protobuf``, os desenvolvedores podem estender suas aplicações com serialização de alto desempenho, garantindo consistência entre as mensagens do contrato e as definições Protobuf.
