# Gateway

When the ``@cmmv/ws`` module is present in the project, it automatically creates WebSocket (WS) gateways based on the defined contracts. These gateways handle **Remote Procedure Call (RPC)** interactions in a binary format, optimizing both the size of the transmitted data and the process of identifying the correct handlers for the messages.

* **RPC Gateways:** Each contract generates a WebSocket gateway that allows interaction between clients and the server using efficient binary communication.

* **Automatic Request Packaging:** The ``@cmmv/ws`` module automatically packages responses and parses incoming requests using the contract definitions, ensuring that all messages conform to the expected structure.

* **WsCall Contract:** The ``WsCall`` contract is a crucial component that serves as an index for system packets and defines the available messages in each contract. It helps minimize the number of bytes sent during interactions and allows easy identification of which handler should process each message.

Here is an example of the ``WsCall`` contract that serves as the foundation for these interactions:

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

**Client Sends a Request:** When the client sends a request, it is formatted according to the ``WsCall`` contract, containing:

* **contract:** A number identifying which contract the message belongs to.
* **message:** A number identifying the specific message or action.
* **data:** A binary payload (usually a serialized object) containing the actual data of the request.

**Parsing the Request:** Upon receiving the request, the system parses the contract and message fields to determine:

* Which contract the request corresponds to.
* Which handler in the gateway should process the message.

* **Calling the Correct Handler:** Once the system identifies the contract and message, it routes the request to the appropriate handler in the WebSocket gateway. The handler then processes the request and sends back a response in the same efficient binary format.

## Example Gateway

When a contract is created, the WebSocket gateway might look like this:

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

**Binary Communication:** Uses binary data (via Uint8Array) for fast and compact message exchanges.
**Efficient Message Handling:** The contract and message fields allow quick identification of the correct handler, minimizing overhead.
**Custom Gateways:** Developers can extend or customize gateways by creating specific handlers for different contract messages.
This approach ensures that communication between clients and the server is not only efficient but also structured and easy to manage.

## WebSocket Interceptor

The interceptor is a key component in the ``@cmmv/ws`` system that handles the parsing and routing of incoming WebSocket messages. When a WebSocket packet is received from the client, the interceptor processes the data, decodes it according to the defined protocol, and dispatches it to the appropriate handler.

Below is an explanation of how the WebSocket interceptor works step-by-step:

The interceptor receives a ``socket`` and the binary ``data``. The first step is to decode the binary data into a structured message using the ``WSCall`` contract:

```typescript
const message = plainToClass(WSCall, ProtoRegistry.
    retrieve("ws")?.
    lookupType("WsCall").
    decode(data)
);
```

Next, the system retrieves the contract and message type from the ``ProtoRegistry``:

```typescript
const contract = ProtoRegistry.retrieveByIndex(message.contract);
const typeName = ProtoRegistry.retrieveTypes(
    message.contract, message.message
);
```
<br/>

* **retrieveByIndex(message.contract):** Retrieves the contract by its index in the registry.
* **retrieveTypes(message.contract, message.message):** Determines the specific type of message based on the contract and message identifiers. This will be used to find the correct handler.

Once the contract and message type are identified, the interceptor checks if the message type has a registered handler:

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

Finally, the handler is executed with the mapped arguments:

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

This process ensures that all WebSocket messages are efficiently routed to the appropriate handler based on the contract and message type.

## Utils

The ``RpcUtils`` class provides utility methods to work with Protobuf serialization and message packaging for RPC communication. It is primarily used to convert JavaScript objects into Protobuf-encoded binary buffers, as well as to pack RPC messages for transmission over WebSocket or other binary protocols.

| Method                             | Description                                                                                               |
|------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `generateBuffer(protoFile, namespace, data)` | Converts a JavaScript object into a Protobuf-encoded buffer using the specified proto file and message type. |
| `pack(contractName, messageName, data)`       | Packs a message into a Protobuf-encoded buffer for RPC communication, including contract and message metadata. |

``generateBuffer(protoFile: string, namespace: string, data: any): Promise<Uint8Array>``

This method generates a Protobuf-encoded buffer from a JavaScript object using a specific proto file and message type.

**Parameters:**
* **protoFile (``string``):** The name of the proto file that defines the message structure.
* **namespace (``string``):** The name of the message type (namespace) within the proto file.
* **data (``any``):** The JavaScript object containing the data to be encoded into Protobuf format.

**Returns:**
* **``Promise<Uint8Array | null>:``** A promise that resolves to a ``Uint8Array`` representing the encoded Protobuf message, or null if an error occurs.

```typescript
const buffer = await RpcUtils.generateBuffer(
    'myProtoFile', 'MyMessage', { id: 1, name: 'example' }
);

if (buffer) {
    // buffer is ready to be sent via WebSocket or saved
} else {
    // handle error
}
```

## Pack

``pack(contractName: string, messageName: string, data?: any): Promise<Uint8Array | null>``

The ``pack`` method prepares a Protobuf-encoded message for RPC communication. It includes metadata such as the contract and message indices, along with the actual message data.

**Parameters:**
* **contractName (``string``):** The name of the contract associated with the message.
* **messageName (``string``):** The name of the message within the contract.
* **data (``any``, optional):** The data to be included in the message, which will be encoded as Protobuf.

**Returns:**
* **``Promise<Uint8Array | null>``:** A promise that resolves to a ``Uint8Array`` representing the packed message, or ``null`` if an error occurs.

```typescript
const packedMessage = await RpcUtils.pack(
    'TaskContract', 'AddTaskRequest', { taskId: 1, taskLabel: 'New Task' }
);

if (packedMessage) {
    // ready to send packed message via WebSocket or other binary protocol
} else {
    // handle error
}
```