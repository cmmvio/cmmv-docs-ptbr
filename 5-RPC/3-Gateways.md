# Gateway

Quando o módulo `@cmmv/ws` está presente no projeto, ele cria automaticamente gateways WebSocket (WS) com base nos contratos definidos. Esses gateways gerenciam interações de **Remote Procedure Call (RPC)** em um formato binário, otimizando tanto o tamanho dos dados transmitidos quanto o processo de identificação dos handlers corretos para as mensagens.

* **Gateways RPC:** Cada contrato gera um gateway WebSocket que permite interação entre clientes e o servidor utilizando comunicação binária eficiente.

* **Empacotamento Automático de Solicitações:** O módulo `@cmmv/ws` empacota automaticamente as respostas e analisa as solicitações recebidas usando as definições dos contratos, garantindo que todas as mensagens estejam em conformidade com a estrutura esperada.

* **Contrato WsCall:** O contrato `WsCall` é um componente crucial que serve como índice para pacotes do sistema e define as mensagens disponíveis em cada contrato. Ele minimiza a quantidade de bytes enviados durante as interações e permite fácil identificação do handler responsável pelo processamento de cada mensagem.

Aqui está um exemplo do contrato `WsCall`, que serve como base para essas interações:

```typescript
import { AbstractContract, Contract, ContractField } from "@cmmv/core";

@Contract({ 
    controllerName: "WsCall",
    protoPath: "src/protos/ws.proto",
    protoPackage: "ws",
    directMessage: true,
    generateController: false 
})
export class WSContract extends AbstractContract {
    @ContractField({ protoType: 'int32' })
    contract: number;

    @ContractField({ protoType: 'int32' })
    message: number;

    @ContractField({ protoType: 'bytes' })
    data: Uint8Array;
}
```

**Cliente Envia uma Solicitação:** Quando o cliente envia uma solicitação, ela é formatada de acordo com o contrato `WsCall`, contendo:

* **contract:** Um número identificando a qual contrato a mensagem pertence.
* **message:** Um número identificando a mensagem ou ação específica.
* **data:** Um payload binário (geralmente um objeto serializado) contendo os dados reais da solicitação.

**Analisando a Solicitação:** Ao receber a solicitação, o sistema analisa os campos `contract` e `message` para determinar:

* A qual contrato a solicitação corresponde.
* Qual handler no gateway deve processar a mensagem.

**Chamando o Handler Correto:** Depois de identificar o contrato e a mensagem, o sistema encaminha a solicitação para o handler apropriado no gateway WebSocket. O handler processa a solicitação e envia uma resposta no mesmo formato binário eficiente.

## Exemplo de Gateway

Quando um contrato é criado, o gateway WebSocket pode ser assim:

```typescript
import { Rpc, Message, Data, Socket, RpcUtils } from "@cmmv/ws";
import { TaskService } from '../services/task.service';

@Rpc("task")
export class TaskGateway {
    constructor(private readonly taskService: TaskService) {}

    @Message("GetAllTaskRequest")
    async getAll(@Socket() socket) {
        const items = await this.taskService.getAll();

        const response = await RpcUtils.pack(
            "task", "GetAllTaskResponse", items
        );

        socket.send(response);
    }

    @Message("AddTaskRequest")
    async add(@Data() data, @Socket() socket) {
        const result = await this.taskService.add(data.item);

        const response = await RpcUtils.pack(
            "task", "AddTaskResponse", { item: result }
        );

        socket.send(response);
    }
}
```

* **Comunicação Binária:** Usa dados binários (via `Uint8Array`) para trocas rápidas e compactas de mensagens.
* **Manuseio Eficiente de Mensagens:** Os campos `contract` e `message` permitem rápida identificação do handler correto, minimizando sobrecarga.
* **Gateways Personalizados:** Desenvolvedores podem estender ou personalizar gateways criando handlers específicos para diferentes mensagens de contrato.

Essa abordagem garante que a comunicação entre clientes e o servidor seja não apenas eficiente, mas também estruturada e fácil de gerenciar.

## Interceptor WebSocket

O interceptor é um componente chave no sistema `@cmmv/ws` que gerencia a análise e roteamento das mensagens WebSocket recebidas. Quando um pacote WebSocket é recebido do cliente, o interceptor processa os dados, decodifica-os de acordo com o protocolo definido e os despacha para o handler apropriado.

Abaixo está uma explicação de como o interceptor WebSocket funciona, passo a passo:

O interceptor recebe um `socket` e os dados binários `data`. O primeiro passo é decodificar os dados binários em uma mensagem estruturada usando o contrato `WsCall`:

```typescript
const message = plainToClass(WSCall, ProtoRegistry.
    retrieve("ws")?.
    lookupType("WsCall").
    decode(data)
);
```

Em seguida, o sistema recupera o contrato e o tipo de mensagem do `ProtoRegistry`:

```typescript
const contract = ProtoRegistry.retrieveByIndex(message.contract);
const typeName = ProtoRegistry.retrieveTypes(
    message.contract, message.message
);
```

* **`retrieveByIndex(message.contract):`** Recupera o contrato pelo índice no registro.
* **`retrieveTypes(message.contract, message.message):`** Determina o tipo específico da mensagem com base nos identificadores de contrato e mensagem. Isso será usado para encontrar o handler correto.

Depois que o contrato e o tipo de mensagem são identificados, o interceptor verifica se o tipo de mensagem tem um handler registrado:

```typescript
if (contract && this.registeredMessages.has(typeName)) {
    const { 
        instance, handlerName, params 
    } = this.registeredMessages.get(typeName);

    const realMessage = contract
    .lookupType(typeName)
    .decode(message.data);
}
```

Finalmente, o handler é executado com os argumentos mapeados:

```typescript
const args = params
    .sort((a, b) => a.index - b.index)
    .map(param => {
        switch (param.paramType) {
            case 'data': return realMessage;
            case 'socket': return socket;
            default: return undefined;
        }
    });

try {
    instance[handlerName](...args);
} catch (e) {
    this.logger.error(e.message);
}
```

Esse processo garante que todas as mensagens WebSocket sejam roteadas de forma eficiente para o handler apropriado com base no contrato e tipo de mensagem.

## Utils

A classe `RpcUtils` fornece métodos utilitários para trabalhar com a serialização Protobuf e empacotamento de mensagens para comunicação RPC. Ela é usada principalmente para converter objetos JavaScript em buffers binários codificados em Protobuf e empacotar mensagens RPC para transmissão via WebSocket ou outros protocolos binários.

| Método                             | Descrição                                                                                               |
|------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `generateBuffer(protoFile, namespace, data)` | Converte um objeto JavaScript em um buffer codificado em Protobuf usando o arquivo proto e o tipo de mensagem especificado. |
| `pack(contractName, messageName, data)`       | Empacota uma mensagem em um buffer codificado em Protobuf para comunicação RPC, incluindo metadados de contrato e mensagem. |

`generateBuffer(protoFile: string, namespace: string, data: any): Promise<Uint8Array>`

Este método gera um buffer codificado em Protobuf a partir de um objeto JavaScript usando um arquivo proto específico e tipo de mensagem.

**Parâmetros:**
* **protoFile (`string`):** O nome do arquivo proto que define a estrutura da mensagem.
* **namespace (`string`):** O nome do tipo de mensagem (namespace) dentro do arquivo proto.
* **data (`any`):** O objeto JavaScript contendo os dados a serem codificados no formato Protobuf.

**Retorna:**
* **`Promise<Uint8Array | null>`:** Uma promise que resolve para um `Uint8Array` representando a mensagem codificada em Protobuf, ou null se ocorrer um erro.

```typescript
const buffer = await RpcUtils.generateBuffer(
    'myProtoFile', 'MyMessage', { id: 1, name: 'example' }
);

if (buffer) {
    // buffer pronto para envio via WebSocket ou armazenamento
} else {
    // tratar erro
}
```

## Pack

`pack(contractName: string, messageName: string, data?: any): Promise<Uint8Array | null>`

O método `pack` prepara uma mensagem codificada em Protobuf para comunicação RPC. Inclui metadados como os índices de contrato e mensagem, junto com os dados reais da mensagem.

**Parâmetros:**
* **contractName (`string`):** O nome do contrato associado à mensagem.
* **messageName (`string`):** O nome da mensagem dentro do contrato.
* **data (`any`, opcional):** Os dados a serem incluídos na mensagem, que serão codificados em Protobuf.

**Retorna:**
* **`Promise<Uint8Array | null>`:** Uma promise que resolve para um `Uint8Array` representando a mensagem empacotada, ou `null` se ocorrer um erro.

```typescript
const packedMessage = await RpcUtils.pack(
    'TaskContract', 'AddTaskRequest', { taskId: 1, taskLabel: 'New Task' }
);

if (packedMessage) {
    // mensagem empacotada pronta para envio via WebSocket ou outro protocolo binário
} else {
    // tratar erro
}
```
