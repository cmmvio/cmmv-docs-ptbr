# Transpiles

In the CMMV framework, transpilers are essential tools used to generate necessary files based on contracts and module requirements. These classes automatically create code or configuration files that are vital for the proper functioning of modules such as HTTP, Protobuf, and Repository.

Whenever a module requires specific artifacts, like controllers, entities, or Protobuf definitions, a transpiler processes the contracts and generates the required files. This approach ensures modularity and adaptability, as each module can define its own set of transpilers to handle file generation as needed.

## Key Features of Transpilers

* **Automatic Code Generation:** Transpilers generate files automatically based on contracts, reducing manual work and ensuring consistency between your application and the database, API interfaces, or other modules.
* **Modular:** Each module (e.g., HTTP, Protobuf, Repository) has its own transpiler, ensuring a clean separation of concerns.
* **Customizable:** New transpilers can be added for other custom modules, making the system extensible.
* **Contract-Based:** Transpilers rely on contracts to define what code needs to be generated. Contracts are the central mechanism for defining the structure, behavior, and data types for entities and services.

## Native Transpilers

* **Protobuf Transpiler:** Generates .proto files based on contracts to enable RPC communication.
* **Repository Transpiler:** Creates database entities based on TypeORM, implementing CRUD functionality based on contracts.
* **HTTP Transpiler:** Generates controllers and routes for RESTful APIs based on the contract definitions.
* **Websocket Transpiler:** Generates Websocket communication gateways for RPC based on contract definitions.

## Example

Below is an implementation of the ProtobufTranspile class, which generates ``.proto`` files from contracts to facilitate communication using Protobuf.

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
                this.logger.log(`Created directory ${outputDir}`);
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
                const listMessage = new protobufjs.Type(`${contract.controllerName}List`)
                    .add(new protobufjs.Field("items", 1, `${contract.controllerName}`, "repeated"));

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
    
        lines.push('// Generated automatically by CMMV');
        lines.push(`syntax = "proto3";`);
    
        if (packageName)
            lines.push(`package ${packageName};`);
    
        if (includesGoogleAny) 
            lines.push('import "google/protobuf/any.proto";');
            
        lines.push('');
    
        lines.push(`message ${contract.controllerName} {`);
    
        contract.fields.forEach((field: any, index: number) => {
            const protoType = this.mapToProtoType(field.protoType);
            const repeatedPrefix = field.protoRepeated ? 'repeated ' : '';
            lines.push(`  ${repeatedPrefix}${protoType} ${field.propertyKey} = ${index + 1};`);
        });
    
        lines.push(`}`);
    
        if (!contract.directMessage) {
            lines.push('');
            lines.push(`message ${contract.controllerName}List {`);
            lines.push(`  repeated ${contract.controllerName} items = 1;`);
            lines.push(`}`);
        }
    
        lines.push('');
        lines.push(`message Add${contract.controllerName}Request {`);
        lines.push(`  ${contract.controllerName} item = 1;`);
        lines.push(`}`);
        lines.push(`message Add${contract.controllerName}Response {`);
        lines.push(`  string id = 1;`); 
        lines.push(`  ${contract.controllerName} item = 2;`);
        lines.push(`}`);
        
        lines.push('');
        lines.push(`message Update${contract.controllerName}Request {`);
        lines.push(`  string id = 1;`); 
        lines.push(`  ${contract.controllerName} item = 2;`);
        lines.push(`}`);
        lines.push(`message Update${contract.controllerName}Response {`);
        lines.push(`  string id = 1;`); 
        lines.push(`  ${contract.controllerName} item = 2;`);
        lines.push(`}`);
    
        lines.push('');
        lines.push(`message Delete${contract.controllerName}Request {`);
        lines.push(`  string id = 1;`);
        lines.push(`}`);
        lines.push(`message Delete${contract.controllerName}Response {`);
        lines.push(`  bool success = 1;`);
        lines.push(`  string id = 2;`);
        lines.push(`}`);
    
        lines.push('');
        lines.push(`message GetAll${contract.controllerName}Request {}`);
        lines.push(`message GetAll${contract.controllerName}Response {`);
        lines.push(`  ${contract.controllerName}List items = 1;`);
        lines.push(`}`);
    
        lines.push('');
        lines.push(`service ${contract.controllerName}Service {`);
        lines.push(`  rpc Add${contract.controllerName} (Add${contract.controllerName}Request) returns (Add${contract.controllerName}Response);`);
        lines.push(`  rpc Update${contract.controllerName} (Update${contract.controllerName}Request) returns (Update${contract.controllerName}Response);`);
        lines.push(`  rpc Delete${contract.controllerName} (Delete${contract.controllerName}Request) returns (Delete${contract.controllerName}Response);`);
        lines.push(`  rpc GetAll${contract.controllerName} (GetAll${contract.controllerName}Request) returns (GetAll${contract.controllerName}Response);`);
        lines.push(`}`);
    
        return lines.join('\n');
    }

    private generateTypes(contract: any): string {
        const lines: string[] = [];
    
        lines.push(`// Types generated automatically by CMMV`);
        lines.push(`export namespace ${contract.controllerName} {`);
    
        contract.fields.forEach((field: any) => {
            const tsType = this.mapToTsType(field.protoType);
            lines.push(`  export type ${field.propertyKey} = ${tsType};`);
        });
    
        lines.push(`}`);
        
        lines.push(`export interface Add${contract.controllerName}Request {`);
        lines.push(`  item: ${contract.controllerName};`);
        lines.push(`}`);
        lines.push(`export interface Add${contract.controllerName}Response {`);
        lines.push(`  item: ${contract.controllerName};`);
        lines.push(`}`);
        
        lines.push(`export interface Update${contract.controllerName}Request {`);
        lines.push(`  id: string;`);
        lines.push(`  item: ${contract.controllerName};`);
        lines.push(`}`);
        lines.push(`export interface Update${contract.controllerName}Response {`);
        lines.push(`  item: ${contract.controllerName};`);
        lines.push(`}`);
        
        lines.push(`export interface Delete${contract.controllerName}Request {`);
        lines.push(`  id: string;`);
        lines.push(`}`);
        lines.push(`export interface Delete${contract.controllerName}Response {`);
        lines.push(`  success: boolean;`);
        lines.push(`}`);
        
        lines.push(`export interface GetAll${contract.controllerName}Request {}`);
        lines.push(`export interface GetAll${contract.controllerName}Response {`);
        lines.push(`  items: ${contract.controllerName}[];`);
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
    
        let jsContent = '// Generated automatically by CMMV\n';
        jsContent += '(function(global) {\n';
        jsContent += '  try {\n';
        jsContent += '    global.cmmv.addContracts(' + JSON.stringify(data) + ');\n';
        jsContent += '  } catch (e) {\n';
        jsContent += '    console.error("Error loading contracts:", e);\n';
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

In the example above, the Protobuf transpiler takes contract definitions and generates .proto files to be used for RPC communication with the protobufjs library. These .proto files define message structures for binary data exchanges. Additionally, TypeScript types are generated based on these contracts to ensure type safety. Finally, the contracts are converted into JSON format and attached to the frontend framework, which will handle the sending and receiving of binary data in the Protobuf format, enabling efficient data communication between frontend and backend.

## Call a Transpiler

In CMMV, transpilers are modular, which allows them to be dynamically added to modules and invoked during the application processing. A transpiler's role is to automatically generate necessary files (such as entities, services, and Protobuf definitions) based on the contracts defined in your application.

``/packages/protobuf/src/protobuf.module.ts``
```typescript
import { Module } from "@cmmv/core";
import { ProtobufTranspile } from "./protobuf.transpiler";

export let ProtobufModule = new Module({
    transpilers: [ProtobufTranspile] // Register the transpiler here
});
```

In this case, ProtobufModule registers the ProtobufTranspile transpiler. The transpilers property in the Module configuration allows you to list all the transpilers that should be executed when the module is processed.

When the application is started, the transpiler will be invoked automatically as part of the module processing. Each module's transpilers will run based on the contracts defined in the system, and they will generate the required files accordingly.

```typescript
import { Application } from "@cmmv/core";
import { ProtobufModule } from "@cmmv/protobuf";

new Application({
    ...
    modules: [ProtobufModule], 
});
```

When the application starts, it will look for all modules, identify the transpilers within those modules, and run them in sequence. In this case, the ProtobufTranspile will process any contracts registered in the system and generate the corresponding Protobuf files.

This modular design makes it easy to extend the system by adding new transpilers for various tasks, such as generating database entities, Protobuf definitions, or other required files based on the contracts defined in your application.