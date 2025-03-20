# Transpiladores

No framework CMMV, os transpiladores são ferramentas essenciais usadas para gerar arquivos necessários com base em contratos e requisitos de módulos. Essas classes criam automaticamente arquivos de código ou configuração que são vitais para o funcionamento adequado de módulos como HTTP, Protobuf e Repositório.

Sempre que um módulo requer artefatos específicos, como controladores, entidades ou definições Protobuf, um transpilador processa os contratos e gera os arquivos necessários. Essa abordagem garante modularidade e adaptabilidade, pois cada módulo pode definir seu próprio conjunto de transpiladores para lidar com a geração de arquivos conforme necessário.

## Principais Características dos Transpiladores

* **Geração Automática de Código:** Os transpiladores geram arquivos automaticamente com base em contratos, reduzindo o trabalho manual e garantindo consistência entre sua aplicação e o banco de dados, interfaces de API ou outros módulos.
* **Modular:** Cada módulo (por exemplo, HTTP, Protobuf, Repositório) tem seu próprio transpilador, garantindo uma separação clara de responsabilidades.
* **Personalizável:** Novos transpiladores podem ser adicionados para outros módulos personalizados, tornando o sistema extensível.
* **Baseado em Contratos:** Os transpiladores dependem de contratos para definir qual código precisa ser gerado. Os contratos são o mecanismo central para definir a estrutura, comportamento e tipos de dados para entidades e serviços.

## Transpiladores Nativos

* **Transpilador Protobuf:** Gera arquivos .proto com base em contratos para habilitar comunicação RPC.
* **Transpilador de Repositório:** Cria entidades de banco de dados com base no TypeORM, implementando funcionalidades CRUD com base em contratos.
* **Transpilador HTTP:** Gera controladores e rotas para APIs RESTful com base nas definições de contrato.
* **Transpilador Websocket:** Gera gateways de comunicação Websocket para RPC com base nas definições de contrato.

## Exemplo

Abaixo está uma implementação da classe ProtobufTranspile, que gera arquivos `.proto` a partir de contratos para facilitar a comunicação usando Protobuf.

```typescript
import * as fs from 'fs';
import * as path from 'path';
import * as protobufjs from 'protobufjs';
import * as UglifyJS from 'uglify-js';

import { ProtoRegistry } from './protobuf-registry.utils';

import { ITranspile, Logger, Scope } from "@cmmv/core";

export class ProtobufTranspile implements ITranspile {
    private logger: Logger = new Logger('ProtobufTranspile');

    run(): void {
        const contracts = Scope.getArray<any>("__contracts");

        const contractsJson: { [key: string]: any } = {};

        contracts?.forEach((contract: any) => {
            const outputPath = path.resolve(contract.protoPath);
            const outputPathJson = outputPath.replace('.proto', '.json');
            const outputPathTs = outputPath.replace('.proto', '.d.ts');
            const outputDir = path.dirname(outputPath);

            if (!fs.existsSync(outputDir)) {
                fs.mkdirSync(outputDir, { recursive: true });
                this.logger.log(`Criado diretório \${outputDir}`);
            }

            const root = new protobufjs.Root();
            const protoNamespace = root.define(contract.controllerName);

            const itemMessage = new protobufjs.Type(contract.controllerName)
                .add(new protobufjs.Field("id", 1, "int32"));

            contract.fields.forEach((field: any, index: number) => {
                const protoType = this.mapToProtoType(field.protoType);
                itemMessage.add(new protobufjs.Field(field.propertyKey, index + 2, protoType));
            });

            protoNamespace.add(itemMessage);

            if (!contract.directMessage) {
                const listMessage = new protobufjs.Type(`\${contract.controllerName}List`)
                    .add(new protobufjs.Field("items", 1, `\${contract.controllerName}`, "repeated"));

                protoNamespace.add(listMessage);
            }

            const protoContent = this.generateProtoContent(contract);
            fs.writeFileSync(outputPath, protoContent, 'utf8');

            const tsContent = this.generateTypes(contract);
            fs.writeFileSync(outputPathTs, tsContent, 'utf8');

            const contractJSON = root.toJSON();
            contractsJson[contract.controllerName] = contractJSON;
            fs.writeFileSync(outputPathJson, JSON.stringify(contractJSON), 'utf8');
        });

        this.generateContractsJs(contractsJson);
    }

    private generateProtoContent(contract: any): string {
        const packageName = contract.protoPackage || null;
        const lines: string[] = [];
        let includesGoogleAny = false;

        contract.fields.forEach((field: any) => {
            if (field.protoType === 'any')
                includesGoogleAny = true;
        });

        lines.push('// Gerado automaticamente pelo CMMV');
        lines.push(`syntax = "proto3";`);

        if (packageName)
            lines.push(`package \${packageName};`);

        if (includesGoogleAny)
            lines.push('import "google/protobuf/any.proto";');

        lines.push('');

        lines.push(`message \${contract.controllerName} {`);

        contract.fields.forEach((field: any, index: number) => {
            const protoType = this.mapToProtoType(field.protoType);
            const repeatedPrefix = field.protoRepeated ? 'repeated ' : '';
            lines.push(`  \${repeatedPrefix}\${protoType} \${field.propertyKey} = \${index + 1};`);
        });

        lines.push(`}`);

        if (!contract.directMessage) {
            lines.push('');
            lines.push(`message \${contract.controllerName}List {`);
            lines.push(`  repeated \${contract.controllerName} items = 1;`);
            lines.push(`}`);
        }

        lines.push('');
        lines.push(`message Add\${contract.controllerName}Request {`);
        lines.push(`  \${contract.controllerName} item = 1;`);
        lines.push(`}`);
        lines.push(`message Add\${contract.controllerName}Response {`);
        lines.push(`  string id = 1;`);
        lines.push(`  \${contract.controllerName} item = 2;`);
        lines.push(`}`);

        lines.push('');
        lines.push(`message Update\${contract.controllerName}Request {`);
        lines.push(`  string id = 1;`);
        lines.push(`  \${contract.controllerName} item = 2;`);
        lines.push(`}`);
        lines.push(`message Update\${contract.controllerName}Response {`);
        lines.push(`  string id = 1;`);
        lines.push(`  \${contract.controllerName} item = 2;`);
        lines.push(`}`);

        lines.push('');
        lines.push(`message Delete\${contract.controllerName}Request {`);
        lines.push(`  string id = 1;`);
        lines.push(`}`);
        lines.push(`message Delete\${contract.controllerName}Response {`);
        lines.push(`  bool success = 1;`);
        lines.push(`  string id = 2;`);
        lines.push(`}`);

        lines.push('');
        lines.push(`message GetAll\${contract.controllerName}Request {}`);
        lines.push(`message GetAll\${contract.controllerName}Response {`);
        lines.push(`  \${contract.controllerName}List items = 1;`);
        lines.push(`}`);

        lines.push('');
        lines.push(`service \${contract.controllerName}Service {`);
        lines.push(`  rpc Add\${contract.controllerName} (Add\${contract.controllerName}Request) returns (Add\${contract.controllerName}Response);`);
        lines.push(`  rpc Update\${contract.controllerName} (Update\${contract.controllerName}Request) returns (Update\${contract.controllerName}Response);`);
        lines.push(`  rpc Delete\${contract.controllerName} (Delete\${contract.controllerName}Request) returns (Delete\${contract.controllerName}Response);`);
        lines.push(`  rpc GetAll\${contract.controllerName} (GetAll\${contract.controllerName}Request) returns (GetAll\${contract.controllerName}Response);`);
        lines.push(`}`);

        return lines.join('\n');
    }

    private generateTypes(contract: any): string {
        const lines: string[] = [];

        lines.push(`// Tipos gerados automaticamente pelo CMMV`);
        lines.push(`export namespace \${contract.controllerName} {`);

        contract.fields.forEach((field: any) => {
            const tsType = this.mapToTsType(field.protoType);
            lines.push(`  export type \${field.propertyKey} = \${tsType};`);
        });

        lines.push(`}`);

        lines.push(`export interface Add\${contract.controllerName}Request {`);
        lines.push(`  item: \${contract.controllerName};`);
        lines.push(`}`);
        lines.push(`export interface Add\${contract.controllerName}Response {`);
        lines.push(`  item: \${contract.controllerName};`);
        lines.push(`}`);

        lines.push(`export interface Update\${contract.controllerName}Request {`);
        lines.push(`  id: string;`);
        lines.push(`  item: \${contract.controllerName};`);
        lines.push(`}`);
        lines.push(`export interface Update\${contract.controllerName}Response {`);
        lines.push(`  item: \${contract.controllerName};`);
        lines.push(`}`);

        lines.push(`export interface Delete\${contract.controllerName}Request {`);
        lines.push(`  id: string;`);
        lines.push(`}`);
        lines.push(`export interface Delete\${contract.controllerName}Response {`);
        lines.push(`  success: boolean;`);
        lines.push(`}`);

        lines.push(`export interface GetAll\${contract.controllerName}Request {}`);
        lines.push(`export interface GetAll\${contract.controllerName}Response {`);
        lines.push(`  items: \${contract.controllerName}[];`);
        lines.push(`}`);

        return lines.join('\n');
    }

    private async generateContractsJs(contractsJson: { [key: string]: any }): Promise<void> {
        const outputFile = path.resolve('public/core/contracts.min.js');

        await ProtoRegistry.load();
        const contracts = ProtoRegistry.retrieveAll();
        let contractsJSON = {};
        let index = {};
        let pointer = 0;

        for(let key in contracts){
            const contract = ProtoRegistry.retrieve(key);
            contractsJSON[key] = contract.toJSON();
            let types = {};
            let pointerTypes = 0;

            for(let namespace of contract.nestedArray){
                for(let type in namespace.toJSON().nested){
                    types[type] = pointerTypes;
                    pointerTypes++;
                }
            }

            index[key] = { index: pointer, types };
            pointer++;
        }

        const data = {
            index,
            contracts: contractsJSON
        };

        let jsContent = '// Gerado automaticamente pelo CMMV\n';
        jsContent += '(function(global) {\n';
        jsContent += '  try {\n';
        jsContent += '    global.cmmv.addContracts(' + JSON.stringify(data) + ');\n';
        jsContent += '  } catch (e) {\n';
        jsContent += '    console.error("Erro ao carregar contratos:", e);\n';
        jsContent += '  }\n';
        jsContent += '})(typeof window !== "undefined" ? window : global);\n';

        const minifiedJsContent = UglifyJS.minify(jsContent).code;

        fs.writeFileSync(outputFile, minifiedJsContent, 'utf8');
    }

    private mapToProtoType(type: string): string {
        const typeMapping: { [key: string]: string } = {
            string: 'string',
            boolean: 'bool',
            bool: 'bool',
            int: 'int32',
            int32: 'int32',
            int64: 'int64',
            float: 'float',
            double: 'double',
            bytes: 'bytes',
            date: 'string',
            timestamp: 'string',
            text: 'string',
            json: 'string',
            jsonb: 'string',
            uuid: 'string',
            time: 'string',
            simpleArray: 'string',
            simpleJson: 'string',
            bigint: 'int64',
            uint32: 'uint32',
            uint64: 'uint64',
            sint32: 'sint32',
            sint64: 'sint64',
            fixed32: 'fixed32',
            fixed64: 'fixed64',
            sfixed32: 'sfixed32',
            sfixed64: 'sfixed64',
            any: 'google.protobuf.Any'
        };

        return typeMapping[type] || 'string';
    }

    private mapToTsType(protoType: string): string {
        const typeMapping: { [key: string]: string } = {
            string: 'string',
            boolean: 'boolean',
            bool: 'boolean',
            int: 'number',
            int32: 'number',
            int64: 'number',
            float: 'number',
            double: 'number',
            bytes: 'Uint8Array',
            date: 'string',
            timestamp: 'string',
            text: 'string',
            json: 'any',
            jsonb: 'any',
            uuid: 'string',
            time: 'string',
            simpleArray: 'string[]',
            simpleJson: 'any',
            bigint: 'bigint',
            uint32: 'number',
            uint64: 'number',
            sint32: 'number',
            sint64: 'number',
            fixed32: 'number',
            fixed64: 'number',
            sfixed32: 'number',
            sfixed64: 'number',
            any: 'any'
        };

        return typeMapping[protoType] || 'any';
    }
}
```

No exemplo acima, o transpilador Protobuf toma definições de contrato e gera arquivos .proto para serem usados na comunicação RPC com a biblioteca protobufjs. Esses arquivos .proto definem estruturas de mensagens para trocas de dados binários. Além disso, tipos TypeScript são gerados com base nesses contratos para garantir segurança de tipo. Finalmente, os contratos são convertidos em formato JSON e anexados ao framework frontend, que lidará com o envio e recebimento de dados binários no formato Protobuf, possibilitando uma comunicação de dados eficiente entre frontend e backend.

## Chamar um Transpilador

No CMMV, os transpiladores são modulares, o que permite que sejam adicionados dinamicamente aos módulos e invocados durante o processamento da aplicação. O papel de um transpilador é gerar automaticamente arquivos necessários (como entidades, serviços e definições Protobuf) com base nos contratos definidos em sua aplicação.

`/packages/protobuf/src/protobuf.module.ts`
```typescript
import { Module } from "@cmmv/core";
import { ProtobufTranspile } from "./protobuf.transpiler";

export let ProtobufModule = new Module({
    transpilers: [ProtobufTranspile] // Registre o transpilador aqui
});
```

Nesse caso, o ProtobufModule registra o transpilador ProtobufTranspile. A propriedade transpilers na configuração do Module permite listar todos os transpiladores que devem ser executados quando o módulo é processado.

Quando a aplicação é iniciada, o transpilador será invocado automaticamente como parte do processamento do módulo. Os transpiladores de cada módulo serão executados com base nos contratos definidos no sistema, e eles gerarão os arquivos necessários de acordo.

```typescript
import { Application } from "@cmmv/core";
import { ProtobufModule } from "@cmmv/protobuf";

new Application({
    ...
    modules: [ProtobufModule],
});
```

Quando a aplicação é iniciada, ela procurará por todos os módulos, identificará os transpiladores dentro desses módulos e os executará em sequência. Nesse caso, o ProtobufTranspile processará quaisquer contratos registrados no sistema e gerará os arquivos Protobuf correspondentes.

Esse design modular facilita a extensão do sistema ao adicionar novos transpiladores para várias tarefas, como gerar entidades de banco de dados, definições Protobuf ou outros arquivos necessários com base nos contratos definidos em sua aplicação.
