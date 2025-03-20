# Serviços

No CMMV, o decorador `@ContractService` permite definir métodos personalizados dentro de contratos que serão implementados posteriormente. No contexto do contrato, ele serve para informar aos transpiladores (como RESTful, GraphQL e RPC) sobre a existência dessas funcionalidades, possibilitando a geração automática de chamadas correspondentes nos controladores e serviços. Essa abordagem é especialmente útil quando se deseja implementar funções além das operações CRUD padrão, oferecendo retornos específicos baseados no contexto do contrato, como amplamente utilizado no módulo de autenticação.

O `@ContractService` atua como uma ponte entre a definição do contrato e sua implementação prática nos serviços, gerando *boilerplate* (código base) para facilitar o desenvolvimento. Por exemplo:
- O transpilador RESTful implementará a chamada da função no controlador com a rota especificada.
- O GraphQL gerará os esquemas correspondentes.
- O RPC definirá os métodos para chamadas remotas.
- Um *boilerplate* de função será gerado no serviço dentro de `/src/services` para ser implementado pelo desenvolvedor.

Essa funcionalidade melhora a flexibilidade e a modularidade da aplicação, permitindo que os desenvolvedores criem métodos personalizados que atendam a requisitos específicos do projeto.

## Exemplo de Uso

Aqui está um exemplo de como usar o `@ContractService` em um contrato para definir um método personalizado de login:

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
    imports: ['crypto']
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
        validations: [{ type: 'IsString', message: 'Senha inválida' }],
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
        auth: true,
        createBoilerplate: true,
    })
    Login: Function;
}
```

### O que acontece?
1. **Definição no Contrato:** O método `Login` é declarado com `@ContractService`, especificando sua rota (`path`), método HTTP (`method`), estruturas de requisição (`request`) e resposta (`response`), além de opções como autenticação (`auth`).
2. **Geração no Controlador:** O transpilador RESTful criará uma rota `/login` no controlador associado, utilizando o método `POST` e esperando um `LoginRequest` como entrada, retornando um `LoginResponse`.
3. **Geração no Serviço:** Um *boilerplate* da função será gerado em `/src/services/auth.service.ts` (ou similar), pronto para implementação pelo desenvolvedor.
4. **Suporte Multiplataforma:** GraphQL e RPC também reconhecerão o método e gerarão as respectivas implementações.

## Propriedades do `@ContractService`

O decorador `@ContractService` suporta uma interface específica que define suas propriedades. Abaixo está a interface `IContractService` com suas opções disponíveis:

```typescript
export interface IContractService {
    propertyKey: string;               // Nome interno da propriedade no contrato
    name: string;                      // Nome do serviço/método
    path: string;                      // Caminho da rota (ex.: 'login')
    method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH' | 'OPTIONS'; // Método HTTP suportado
    auth?: boolean;                    // (Opcional) Define se o método requer autenticação
    rootOnly?: boolean;                // (Opcional) Restringe o acesso apenas ao root
    cache?: CacheOptions;              // (Opcional) Configurações de cache para o método
    functionName?: string;             // (Opcional) Nome personalizado da função no serviço
    request: string;                   // Nome da mensagem de requisição (ex.: 'LoginRequest')
    response: string;                  // Nome da mensagem de resposta (ex.: 'LoginResponse')
    createBoilerplate?: boolean;       // (Opcional) Define se o boilerplate será gerado no serviço
}
```

### Descrição das Propriedades
- **`propertyKey`:** Identificador interno usado pelo sistema para referenciar o método no contrato.
- **`name`:** Nome do serviço ou método, usado para identificação e geração de código.
- **`path`:** Define o caminho da rota associada ao método (ex.: `'login'` se tornará `/login` no REST).
- **`method`:** Especifica o método HTTP a ser usado (GET, POST, etc.).
- **`auth`:** Se `true`, exige autenticação para acessar o método.
- **`rootOnly`:** Se `true`, restringe o acesso apenas a usuários com permissões de root.
- **`cache`:** Configurações de cache, como tempo de vida (TTL) ou chave de cache.
- **`functionName`:** Permite personalizar o nome da função gerada no serviço (se não especificado, usa o `name`).
- **`request`:** Refere-se ao nome de uma mensagem de requisição definida com `@ContractMessage`.
- **`response`:** Refere-se ao nome de uma mensagem de resposta definida com `@ContractMessage`.
- **`createBoilerplate`:** Se `true`, gera automaticamente um *boilerplate* no arquivo de serviço para implementação.

## Boilerplate Gerado no Serviço

Com base no exemplo acima, o seguinte *boilerplate* será gerado em `/src/services/auth.service.ts`:

```typescript
import { Telemetry } from '@cmmv/core';
import { LoginRequestDTO, LoginResponseDTO } from '@models/auth/auth.model';
import { Repository } from '@cmmv/repository';

export class AuthServiceGenerated {
    ...
    async Login(payload: LoginRequestDTO): Promise<LoginResponseDTO> {
        throw new Error("Function Login not implemented");
    }
}
```
## Benefícios

Essa abordagem é extremamente útil para:
- **Funcionalidades Personalizadas:** Permite definir métodos além do CRUD, como login, logout ou outras operações específicas.
- **Consistência Multiplataforma:** Garante que REST, GraphQL e RPC implementem as mesmas funcionalidades de forma consistente.
- **Produtividade:** O *boilerplate* gerado reduz o trabalho manual, permitindo que o desenvolvedor foque na lógica de negócios.
- **Escalabilidade:** Facilita a adição de novos métodos ao contrato sem alterar a estrutura existente.

O `@ContractService` é uma ferramenta poderosa para estender os contratos do CMMV, oferecendo flexibilidade e automação para o desenvolvimento de serviços personalizados.
