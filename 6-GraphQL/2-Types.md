# Tipos

O **Core** (`@cmmv/core`) é responsável por gerar **modelos agnósticos** que podem ser usados em diferentes camadas da aplicação, como **controladores, gateways e resolvers**.
Quando o **módulo GraphQL** (`@cmmv/graphql`) é incluído no projeto, o sistema **aprimora automaticamente os modelos** para serem compatíveis com operações GraphQL.

Essa transformação garante que os modelos permaneçam **totalmente estruturados**, enquanto habilita **decoradores GraphQL, imposição de tipos e regras de validação**.

## Como os Modelos São Modificados para GraphQL

Por padrão, os modelos do CMMV são projetados para uso em múltiplos ambientes.
Quando o GraphQL é habilitado, o sistema **adiciona as seguintes modificações**:

✅ **Decorador `@Field` adicionado**: Garante compatibilidade com definições de tipo GraphQL.
✅ **Configurações de tipo aplicadas**: Os campos recebem tipos GraphQL explícitos como `String`, `Int` e `ID`.
✅ **Suporte a campos somente leitura e obrigatórios**: Introduzido na **v0.8.34**, os modelos agora impõem **campos somente leitura** e campos obrigatórios (`!`).
✅ **Modelos agnósticos permanecem intactos**: A estrutura central é preservada para uso em controladores, gateways e outros serviços.

## Modelo Gerado

Abaixo está um exemplo de um **modelo gerado pelo sistema** com aprimoramentos para GraphQL:

```typescript
/**
    **********************************************
    Este script foi gerado automaticamente pelo CMMV.
    Não é recomendado modificar este arquivo manualmente,
    pois ele pode ser sobrescrito pela aplicação.
    **********************************************
**/

import { fastJson, AbstractModel } from "@cmmv/core";

import {
    ApiSchema, ApiProperty,
    ApiPropertyOptional, ApiResponseProperty
} from "@cmmv/openapi";

import {
    ObjectType, Field, ID, Int, Float
} from "@cmmv/graphql";

import {
    Expose,
    instanceToPlain,
    plainToInstance
} from "@cmmv/core";

import {
    IsOptional,
    IsString,
    MinLength,
    MaxLength,
    IsNotEmpty
} from "@cmmv/core";

export interface IGroups {
    id?: any;
    name: string;
    roles?: string[];
}

// Definição do Modelo
@ApiSchema({ name: "Groups" })
@ObjectType()
export class Groups extends AbstractModel implements IGroups {

    @ApiResponseProperty({ type: String }) // OpenAPI
    @Field(type => ID) // GraphQL
    @Expose({ toClassOnly: true })
    @IsOptional()
    readonly id!: string;

    @Expose()
    @IsNotEmpty()
    @IsString({ message: "Nome inválido" })
    @MinLength(3, { message: "Nome inválido" })
    @MaxLength(40, { message: "Nome inválido" })
    @ApiProperty({
        type: String,
        required: true,
    })
    @Field(() => String, { nullable: false }) // GraphQL
    name: string;

    @Expose()
    @ApiPropertyOptional({
        type: [String],
        readOnly: true,
        default: []
    })
    @Field(() => [String], { nullable: true }) // GraphQL
    readonly roles?: string[] = [];

    constructor(partial: Partial<Groups>) {
        super();
        Object.assign(this, partial);
    }

    public static fromPartial(partial: Partial<Groups>): Groups {
        return plainToInstance(Groups, partial, {
            exposeUnsetFields: false,
            enableImplicitConversion: true,
            excludeExtraneousValues: true
        });
    }

    public static fromEntity(entity: any): any {
        return this.sanitizeEntity(Groups, entity);
    }

    public toString() {
        return GroupsFastSchema(this);
    }
}

// Esquema JSON para Processamento Rápido
export const GroupsFastSchemaStructure = {
    title: "Esquema de Groups",
    type: "object",
    properties: {
        id: { type: "string", nullable: false },
        name: { type: "string", nullable: false },
        roles: {
            type: "array",
            nullable: true,
            items: { type: "string" }
        }
    },
    required: ["id", "name"]
};

export const GroupsFastSchema = fastJson(GroupsFastSchemaStructure);
```

### Aprimoramentos para Compatibilidade com GraphQL

| Recurso | Descrição |
|---------|-----------|
| **Decorador `@Field`** | Cada propriedade do modelo recebe o decorador GraphQL `@Field`, tornando-a acessível em consultas e mutações GraphQL. |
| **Tipos GraphQL Explícitos** | Os campos recebem tipos GraphQL como `String`, `Int`, `Float` e `ID` para segurança de tipo. |
| **Suporte a Campos Somente Leitura** | Campos somente leitura são marcados com `readonly` e tratados adequadamente nas respostas GraphQL. |
| **Campos Obrigatórios** | Campos marcados como obrigatórios no modelo agora impõem tipos GraphQL `!` (não nulos). |
| **Integração Perfeita** | Os modelos permanecem totalmente compatíveis com `@cmmv/openapi`, `@cmmv/http` e outros módulos. |

* **Consistência**: Garante que todos os modelos GraphQL sigam um esquema estruturado e validado.
* **Menos Código Boilerplate**: Os desenvolvedores não precisam decorar manualmente cada modelo com decoradores GraphQL.
* **Modelos Multiuso**: Os modelos funcionam perfeitamente em **APIs REST, WebSockets, GraphQL e outras camadas**.
* **Segurança de Tipo Aprimorada**: As operações GraphQL impõem tipagem estrita, reduzindo erros em tempo de execução.

Ao aprimorar automaticamente os modelos para GraphQL, o **CMMV simplifica o desenvolvimento** enquanto mantém **uma estrutura de modelo unificada** em toda a aplicação.
