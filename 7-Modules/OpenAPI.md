# OpenAPI

Este módulo requer **zero configuração** e **mapeia automaticamente todos os contratos e controladores**, gerando uma especificação completa da API, incluindo:

✅ **Rotas, propriedades de requisição e tipos de resposta**
✅ **Paginação, autenticação e autorização**
✅ **Modelos e esquemas DTO**
✅ **Autenticação com Token Bearer & OAuth2 (se `@cmmv/auth` estiver habilitado)**

Além disso, ele gera **dois arquivos de esquema** dentro de `/public`, que podem ser acessados publicamente:

📂 **openapi.json** → Especificação OpenAPI em formato JSON
📂 **openapi.yml** → Especificação OpenAPI em formato YAML

## Instalação

Para instalar o `@cmmv/openapi`, execute:

```bash
$ pnpm add @cmmv/openapi
```

## Configuração

Para habilitar o suporte ao OpenAPI em sua aplicação CMMV, adicione o `OpenAPIModule` à sua configuração:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter } from "@cmmv/http";
import { OpenAPIModule } from "@cmmv/openapi";
import { AuthModule } from "@cmmv/auth";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        OpenAPIModule,
        AuthModule // Opcional, para documentação de autenticação
    ],
    contracts: [...],
});
```

Isso irá automaticamente:

1️⃣ Gerar a documentação OpenAPI a partir de contratos e controladores
2️⃣ Habilitar a interface Swagger UI em `/docs`
3️⃣ Criar os arquivos `openapi.json` e `openapi.yml` em `/public`

## Arquivos de Esquema OpenAPI

Após iniciar sua aplicação, o módulo gerará:

## Acessando a Interface Swagger UI

O módulo inclui uma interface Swagger UI integrada, acessível em:

```bash
http://localhost:3000/docs
```

* Documentação interativa da API
* Teste requisições diretamente no navegador
* Atualizada automaticamente conforme o sistema evolui

⚠️ Importante:
Atualmente, a interface Swagger UI não é personalizável, mas atualizações futuras adicionarão opções de personalização.

## Autenticação & Segurança

Se o `@cmmv/auth` estiver habilitado, o OpenAPI incluirá automaticamente definições de autenticação:

✅ Autenticação com Token Bearer → Usa o cabeçalho `Authorization: Bearer <token>`
✅ Suporte a OAuth2 → Se habilitado, os fluxos OAuth2 serão incluídos na documentação

Exemplo de definição de autenticação OpenAPI:

```json
"securitySchemes": {
    "bearerAuth": {
        "type": "http",
        "scheme": "bearer"
    },
    "OAuth2": {
        "type": "oauth2",
        "flows": {
            "authorizationCode": {
                "authorizationUrl": "/auth/login",
                "tokenUrl": "/auth/token",
                "scopes": {
                    "read": "Garante acesso de leitura",
                    "write": "Garante acesso de escrita"
                }
            }
        }
    }
}
```

Isso garante que sua API seja segura e bem documentada sem exigir configuração manual.

## Geração Automática de Esquemas

O módulo mapeia automaticamente:

* Rotas (baseadas em controladores `@cmmv/http`)
* Propriedades de Requisição e Resposta
* Parâmetros de Consulta & DTOs
* Suporte a Paginação
* Cabeçalhos de Autenticação
* Tratamento de Erros & Tipos de Resposta

Exemplo de um caminho OpenAPI gerado automaticamente:

```json
"/users": {
    "get": {
        "summary": "Obtém a lista de usuários",
        "parameters": [
            { "name": "limit", "in": "query", "schema": { "type": "integer", "default": 10 } },
            { "name": "offset", "in": "query", "schema": { "type": "integer", "default": 0 } }
        ],
        "responses": {
            "200": {
                "description": "Resposta bem-sucedida",
                "content": {
                    "application/json": {
                        "schema": {
                            "$ref": "#/components/schemas/UserList"
                        }
                    }
                }
            }
        }
    }
}
```

## Decoradores OpenAPI

O módulo `@cmmv/openapi` fornece **dois decoradores principais** para adicionar **metadados OpenAPI** a modelos.
Embora os esquemas OpenAPI sejam **gerados automaticamente a partir de contratos**, os desenvolvedores também podem aplicar esses decoradores **manualmente** em **modelos personalizados**.

| Decorador | Descrição |
|-----------|-----------|
| **`@ApiSchema`** | Define um esquema para um modelo OpenAPI, tornando-o disponível na documentação da API. |
| **`@ApiProperty`** | Marca uma propriedade como parte de um esquema OpenAPI, definindo tipo, requisitos e metadados adicionais. |
| **`@ApiPropertyOptional`** | Similar ao `@ApiProperty`, mas marca o campo como opcional. |
| **`@ApiResponseProperty`** | Define propriedades especificamente para respostas da API. |
| **`@ApiHideProperty`** | Exclui uma propriedade de aparecer na documentação OpenAPI. |

O decorador `@ApiSchema` **declara um modelo** na documentação OpenAPI.
Deve ser aplicado em **definições de classe** que devem ser incluídas no esquema da API.

```typescript
import { ApiSchema, ApiProperty } from "@cmmv/openapi";

@ApiSchema({ name: "User" })
export class User {
    @ApiProperty({ type: String })
    username!: string;

    @ApiProperty({ type: String, required: true })
    email!: string;
}
```

O decorador `@ApiProperty` é usado para anotar propriedades dentro de modelos, definindo:

✅ Tipo → O tipo de dado (String, Number, Boolean, etc.).
✅ Obrigatório/Opcional → Se o campo é mandatório.
✅ Valores Padrão → O valor padrão do campo.
✅ Campos Somente Leitura → Campos que não podem ser modificados.

```typescript
import { ApiSchema, ApiProperty } from "@cmmv/openapi";

@ApiSchema({ name: "Product" })
export class Product {
    @ApiProperty({ type: String, required: true })
    id!: string;

    @ApiProperty({ type: String, required: true })
    name!: string;

    @ApiProperty({ type: Number, required: true })
    price!: number;

    @ApiProperty({ type: Boolean, default: false })
    inStock!: boolean;
}
```

### Modelo Completo com Decoradores OpenAPI

Aqui está um exemplo de um modelo gerado automaticamente com integração GraphQL e OpenAPI.

```typescript
import { fastJson, AbstractModel } from "@cmmv/core";
import { ObjectType, Field, ID } from "@cmmv/graphql";

import {
    ApiSchema, ApiProperty, ApiPropertyOptional,
    ApiResponseProperty, ApiHideProperty
} from "@cmmv/openapi";

@ApiSchema({ name: "Groups" })
@ObjectType()
export class Groups extends AbstractModel {
    @ApiResponseProperty({ type: String })
    @Field(type => ID)
    id!: string;

    @ApiProperty({ type: String, required: true })
    @Field(() => String)
    name!: string;

    @ApiPropertyOptional({ type: [String], readOnly: true, default: [] })
    @Field(() => [String], { nullable: true })
    roles?: string[] = [];

    @ApiHideProperty()
    secretCode!: string; // Isso NÃO aparecerá na documentação OpenAPI.

    constructor(partial: Partial<Groups>) {
        super();
        Object.assign(this, partial);
    }
}
```

## Documentação Baseada em Controladores

Outra maneira de documentar sua API é **adicionar metadados diretamente aos controladores**.
Isso é especialmente útil para **módulos agnósticos** que **não sabem** se o módulo OpenAPI estará incluído no projeto.

🚀 **Vantagens da Documentação Baseada em Controladores**:
- ✅ Funciona **mesmo se `@cmmv/openapi` não estiver instalado**.
- ✅ Integra-se automaticamente ao OpenAPI se o módulo estiver presente.
- ✅ Garante **definições de API claras e estruturadas** nos controladores.

### Como Funciona

Nesta abordagem, os controladores definem:
- **Contratos de Rota** → Vinculam operações da API a contratos do sistema.
- **Tipos de Esquema** → Definem modelos de entrada/saída.
- **Resumos & descrições** → Fornecem documentação clara da API.
- **Regras de autenticação** → Usam decoradores `@Auth()`.

```typescript
import { Application } from "@cmmv/core";
import { Auth } from "@cmmv/auth";
import { ApiTags } from "@cmmv/openapi";

import {
   Controller, Get, Post, Put, Delete,
   Queries, Param, Body, Req, RouterSchema
} from "@cmmv/http";

import { Groups, GroupsFastSchema } from "@models/auth/groups.model";
import { GroupsService } from "@services/auth/groups.service";

@Controller('groups')
export class GroupsControllerGenerated {
    constructor(private readonly groupsservice: GroupsService) {}

    @Get({
        contract: Application.getContract("GroupsContract"),
        schema: RouterSchema.GetAll,
        summary: "Retorna registros de Grupos por filtro",
        exposeFilters: true
    })
    @Auth({ rootOnly: true })
    async get(@Queries() queries: any, @Req() req) {
        return this.groupsservice.getAll(queries, req);
    }

    @Get(':id', {
        contract: Application.getContract("GroupsContract"),
        schema: RouterSchema.GetByID,
        summary: "Retorna registro de Grupo por ID"
    })
    @Auth({ rootOnly: true })
    async getById(@Param('id') id: string) {
        return this.groupsservice.getById(id);
    }

    @Post({
        contract: Application.getContract("GroupsContract"),
        schema: RouterSchema.Insert,
        summary: "Insere novo Grupo"
    })
    @Auth({ rootOnly: true })
    async insert(@Body() item: Groups, @Req() req) {
        return this.groupsservice.insert(item, req);
    }

    @Delete(':id', {
        contract: Application.getContract("GroupsContract"),
        schema: RouterSchema.Delete,
        summary: "Exclui Grupo por ID"
    })
    @Auth({ rootOnly: true })
    async delete(@Param('id') id: string) {
        return this.groupsservice.delete(id);
    }
}
```

| Componente | Descrição |
|-----------|-----------|
| **`contract`** | Vincula o método do controlador a um **contrato existente do sistema**. |
| **`schema`** | Especifica um **tipo de esquema** para documentação OpenAPI (ex.: `RouterSchema.GetAll`). |
| **`summary`** | Fornece uma **descrição curta** do que o endpoint faz. |
| **`@Auth()`** | Define regras de autenticação (ex.: `rootOnly: true`). |

Se o **módulo OpenAPI estiver instalado**, os **metadados do controlador são incluídos automaticamente** na documentação da API gerada.

No entanto, se o OpenAPI **não estiver instalado**, o controlador ainda funcionará **sem nenhuma alteração**.

Isso torna essa abordagem ideal para **aplicações modulares** onde a documentação deve ser **opcional, mas disponível**.

## Esquemas Personalizados de Requisição/Resposta

Além da geração automática de esquemas, o `@cmmv/openapi` permite que os desenvolvedores **definam esquemas personalizados de requisição e resposta** para endpoints da API.

Por padrão, o OpenAPI gera esquemas de requisição e resposta **automaticamente** a partir dos contratos do sistema.
No entanto, os desenvolvedores podem **sobrescrever esses padrões** especificando **esquemas personalizados** dentro dos controladores.

```typescript
import { Controller, Post, Body, Request, Response, Session } from "@cmmv/http";
import { Config } from "@cmmv/core";
import { LoginPayloadSchema, LoginReturnSchema } from "./schemas/auth.schemas";

@Controller("auth")
export class AuthController {
    @Post("login", {
        summary: "Rota para login usando nome de usuário e senha",
        externalDocs: "https://cmmv.io/docs/modules/authentication#login",
        docs: {
            body: LoginPayloadSchema,
            return: LoginReturnSchema,
        },
    })
    async handlerLogin(
        @Body() payload,
        @Request() req,
        @Response() res,
        @Session() session
    ) {
        const localLogin = Config.get("auth.localLogin", false);

        if (localLogin) {
            const { result } = await this.authorizationService.login(
                payload,
                req,
                res,
                session
            );
            return result;
        }

        return false;
    }
}
```

| Propriedade       | Descrição |
|------------------|-----------|
| **`docs.body`**   | Define o **esquema de requisição** (formato do payload). |
| **`docs.return`** | Define o **esquema de resposta**, incluindo cabeçalhos e metadados. |
| **`externalDocs`** | Adiciona um **link para documentação externa** com mais detalhes. |

Um **esquema de requisição personalizado** permite que os desenvolvedores **definam propriedades específicas**, regras de validação e exemplos de payloads.

```typescript
export const LoginPayloadSchema = {
    content: {
        "application/json": {
            schema: {
                type: "object",
                properties: {
                    username: { type: "string", required: true },
                    password: { type: "string", required: true },
                    token: { type: "string", required: false },
                    opt: { type: "string", required: false },
                },
            },
            examples: {
                default: {
                    value: {
                        username: "user@example.com",
                        password: "minhasenha",
                    },
                },
                reCAPTCHA: {
                    value: {
                        username: "user@example.com",
                        password: "minhasenha",
                        token: "token_recaptcha",
                    },
                },
                F2A: {
                    value: {
                        username: "user@example.com",
                        password: "minhasenha",
                        token: "token_2fa",
                        opt: "123456",
                    },
                },
            },
        },
    },
};
```
<br/>

- **Regras de validação personalizadas** (ex.: campos obrigatórios).
- **Formatos de entrada flexíveis** (ex.: diferentes métodos de autenticação).
- **Múltiplos exemplos de requisição** para melhor documentação.

Os esquemas de resposta personalizados **definem o que a API retorna**, incluindo:

```typescript
export const LoginReturnSchema = {
    description: "Esquema de resposta de login",
    headers: {
        "set-cookie": {
            schema: {
                type: "string",
                description:
                    "Cookie de sessão para autenticação se a aplicação estiver habilitada",
            },
        },
    },
    content: {
        "application/json": {
            schema: {
                type: "object",
                properties: {
                    status: { type: "number" },
                    processingTime: { type: "number" },
                    result: {
                        type: "object",
                        properties: {
                            token: { type: "string" },
                            refreshToken: { type: "string" },
                        },
                    },
                },
            },
        },
    },
};
```

Para **APIs complexas**, os desenvolvedores podem querer vincular **documentação externa**.
A propriedade **`externalDocs`** permite adicionar uma **URL** para referências estendidas da API.

```typescript
@Post("register", {
    summary: "Rota para registrar um novo usuário público",
    externalDocs: "https://cmmv.io/docs/modules/authentication#user-registration",
    docs: {
        body: RegistryPayloadSchema,
        return: RegistryReturnSchema,
    },
})
async handlerRegister(@Body() payload) {
    return this.authorizationService.register(payload);
}
```

## Conclusão

O módulo `@cmmv/openapi` oferece suporte ao OpenAPI sem necessidade de configuração, garantindo que sua API seja bem documentada, segura e pronta para integração.

Ao mapear automaticamente contratos, controladores e autenticação, ele simplifica a documentação da API enquanto mantém total compatibilidade com o ecossistema CMMV.

✅ OpenAPI & Swagger gerados automaticamente
✅ Documentação da API ao vivo em /docs
✅ Nenhuma configuração manual necessária
✅ Funciona perfeitamente com `@cmmv/http` & `@cmmv/auth`
✅ **Controle refinado** → Personalize esquemas de entrada e saída para endpoints.
✅ **Melhor documentação da API** → Forneça estrutura detalhada, cabeçalhos e metadados.
✅ **Segurança e validação aprimoradas** → Imponha regras de requisição/resposta explicitamente.
✅ **Suporte a documentação externa** → Vincule a detalhes adicionais da API.

Isso permite que os desenvolvedores foquem na construção de APIs em vez de escrever documentação manualmente.
