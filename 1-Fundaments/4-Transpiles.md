# Transpiladores

No framework CMMV, os transpiladores são ferramentas essenciais usadas para gerar arquivos necessários com base nos contratos e nos requisitos dos módulos. Essas classes criam automaticamente códigos ou arquivos de configuração vitais para o funcionamento adequado de módulos como HTTP, Protobuf e Repository.

Sempre que um módulo requer artefatos específicos, como controladores, entidades ou definições Protobuf, um transpilador processa os contratos e gera os arquivos necessários. Essa abordagem garante modularidade e adaptabilidade, pois cada módulo pode definir seu próprio conjunto de transpiladores para lidar com a geração de arquivos conforme necessário.

## Características

* **Geração Automática de Código:** Os transpiladores geram arquivos automaticamente com base nos contratos, reduzindo o trabalho manual e garantindo consistência entre sua aplicação e o banco de dados, interfaces de API ou outros módulos.
* **Modular:** Cada módulo (por exemplo, HTTP, Protobuf, Repository) possui seu próprio transpilador, garantindo uma separação clara de responsabilidades.
* **Customizável:** Novos transpiladores podem ser adicionados para outros módulos personalizados, tornando o sistema extensível.
* **Baseado em Contratos:** Os transpiladores dependem de contratos para definir qual código precisa ser gerado. Contratos são o mecanismo central para definir a estrutura, comportamento e tipos de dados de entidades e serviços.

## Transpiladores Nativos

* **Transpilador Protobuf:** Gera arquivos .proto com base nos contratos para habilitar a comunicação via RPC.
* **Transpilador Repository:** Cria entidades de banco de dados baseadas no TypeORM, implementando funcionalidades CRUD com base nos contratos.
* **Transpilador HTTP:** Gera controladores e rotas para APIs RESTful com base nas definições dos contratos.
* **Transpilador Websocket:** Gera gateways de comunicação Websocket para RPC com base nas definições dos contratos.

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
                this.logger.log(`Created directory \${outputDir}`);
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
    
        lines.push('}');
    
        if (!contract.directMessage) {
            lines.push('');
            lines.push(`message \${contract.controllerName}List {`);
            lines.push(`  repeated \${contract.controllerName} items = 1;`);
            lines.push('}');
        }
    
        lines.push('');
        lines.push(`service \${contract.controllerName}Service {`);
        lines.push(`  rpc Add\${contract.controllerName} (Add\${contract.controllerName}Request) returns (Add\${contract.controllerName}Response);`);
        lines.push(`  rpc Update\${contract.controllerName} (Update\${contract.controllerName}Request) returns (Update\${contract.controllerName}Response);`);
        lines.push(`  rpc Delete\${contract.controllerName} (Delete\${contract.controllerName}Request) returns (Delete\${contract.controllerName}Response);`);
        lines.push(`  rpc GetAll\${contract.controllerName} (GetAll\${contract.controllerName}Request) returns (GetAll\${contract.controllerName}Response);`);
        lines.push('}');
    
        return lines.join('\\n');
    }
}
```

## Executando um Transpilador

Os transpiladores são registrados em um módulo e executados automaticamente durante o processamento do aplicativo.

```typescript
import { Application } from "@cmmv/core";
import { ProtobufModule } from "@cmmv/protobuf";

new Application({
    ...
    modules: [ProtobufModule], 
});
```

Quando a aplicação é iniciada, o transpilador processa os contratos registrados e gera os arquivos necessários automaticamente, garantindo uma integração perfeita entre diferentes módulos e contratos.
