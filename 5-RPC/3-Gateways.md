# Gateway

Quando o módulo ``@cmmv/ws`` está presente no projeto, ele cria automaticamente gateways WebSocket (WS) com base nos contratos definidos. Esses gateways gerenciam interações de **Chamadas de Procedimento Remoto (RPC)** em formato binário, otimizando tanto o tamanho dos dados transmitidos quanto o processo de identificação dos manipuladores corretos para as mensagens.

* **Gateways RPC:** Cada contrato gera um gateway WebSocket que permite a interação entre clientes e o servidor usando uma comunicação binária eficiente.

* **Empacotamento Automático de Requisições:** O módulo ``@cmmv/ws`` empacota automaticamente as respostas e analisa as requisições recebidas usando as definições de contrato, garantindo que todas as mensagens estejam em conformidade com a estrutura esperada.

* **Contrato WsCall:** O contrato ``WsCall`` é um componente crucial que serve como índice para os pacotes do sistema e define as mensagens disponíveis em cada contrato. Ele ajuda a minimizar o número de bytes enviados durante as interações e permite a identificação fácil de qual manipulador deve processar cada mensagem.

Aqui está um exemplo do contrato ``WsCall`` que serve como base para essas interações:

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

**Cliente Envia uma Requisição:** Quando o cliente envia uma requisição, ela é formatada de acordo com o contrato ``WsCall``, contendo:

* **contract:** Um número que identifica a qual contrato a mensagem pertence.
* **message:** Um número que identifica a mensagem ou ação específica.
* **data:** Um payload binário (geralmente um objeto serializado) contendo os dados reais da requisição.

**Análise da Requisição:** Ao receber a requisição, o sistema analisa os campos ``contract`` e ``message`` para determinar:

* A qual contrato a requisição corresponde.
* Qual manipulador no gateway deve processar a mensagem.

* **Chamada do Manipulador Correto:** Uma vez que o sistema identifica o contrato e a mensagem, ele roteia a requisição para o manipulador apropriado no gateway WebSocket. O manipulador então processa a requisição e envia de volta uma resposta no mesmo formato binário eficiente.

## Exemplo de Gateway

Quando um contrato é criado, o gateway WebSocket pode se parecer com isto:

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

**Comunicação Binária:** Usa dados binários (via Uint8Array) para trocas de mensagens rápidas e compactas.
**Manuseio Eficiente de Mensagens:** Os campos ``contract`` e ``message`` permitem a identificação rápida do manipulador correto, minimizando a sobrecarga.
**Gateways Personalizados:** Os desenvolvedores podem estender ou personalizar gateways criando manipuladores específicos para diferentes mensagens de contrato.
Essa abordagem garante que a comunicação entre clientes e o servidor seja não apenas eficiente, mas também estruturada e fácil de gerenciar.

## Interceptador WebSocket

O interceptador é um componente chave no sistema ``@cmmv/ws`` que lida com a análise e o roteamento de mensagens WebSocket recebidas. Quando um pacote WebSocket é recebido do cliente, o interceptador processa os dados, decodifica-os de acordo com o protocolo definido e os despacha para o manipulador apropriado.

Abaixo está uma explicação passo a passo de como o interceptador WebSocket funciona:

O interceptador recebe um ``socket`` e os dados binários ``data``. O primeiro passo é decodificar os dados binários em uma mensagem estruturada usando o contrato ``WSCall``:

```typescript
const message = plainToClass(WSCall, ProtoRegistry.
    retrieve("ws")?.
    lookupType("WsCall").
    decode(data)
);
```

Em seguida, o sistema recupera o contrato e o tipo de mensagem do ``ProtoRegistry``:

```typescript
const contract = ProtoRegistry.retrieveByIndex(message.contract);
const typeName = ProtoRegistry.retrieveTypes(
    message.contract, message.message
);
```
<br/>

* **retrieveByIndex(message.contract):** Recupera o contrato pelo seu índice no registro.
* **retrieveTypes(message.contract, message.message):** Determina o tipo específico de mensagem com base nos identificadores de contrato e mensagem. Isso será usado para encontrar o manipulador correto.

Uma vez que o contrato e o tipo de mensagem são identificados, o interceptador verifica se o tipo de mensagem possui um manipulador registrado:

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

Por fim, o manipulador é executado com os argumentos mapeados:

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

Esse processo garante que todas as mensagens WebSocket sejam roteadas eficientemente para o manipulador apropriado com base no tipo de contrato e mensagem.

## Utilitários

A classe ``RpcUtils`` fornece métodos utilitários para trabalhar com serialização Protobuf e empacotamento de mensagens para comunicação RPC. Ela é usada principalmente para converter objetos JavaScript em buffers binários codificados em Protobuf, bem como para empacotar mensagens RPC para transmissão via WebSocket ou outros protocolos binários.

| Método                             | Descrição                                                                                               |
|------------------------------------|---------------------------------------------------------------------------------------------------------|
| `generateBuffer(protoFile, namespace, data)` | Converte um objeto JavaScript em um buffer codificado em Protobuf usando o arquivo proto e tipo de mensagem especificados. |
| `pack(contractName, messageName, data)`       | Empacota uma mensagem em um buffer codificado em Protobuf para comunicação RPC, incluindo metadados de contrato e mensagem. |

### ``generateBuffer(protoFile: string, namespace: string, data: any): Promise<Uint8Array>``

Este método gera um buffer codificado em Protobuf a partir de um objeto JavaScript usando um arquivo proto específico e tipo de mensagem.

**Parâmetros:**
* **protoFile (``string``):** O nome do arquivo proto que define a estrutura da mensagem.
* **namespace (``string``):** O nome do tipo de mensagem (namespace) dentro do arquivo proto.
* **data (``any``):** O objeto JavaScript contendo os dados a serem codificados no formato Protobuf.

**Retorno:**
* **``Promise<Uint8Array | null>:``** Uma promessa que resolve em um ``Uint8Array`` representando a mensagem Protobuf codificada, ou ``null`` se ocorrer um erro.

```typescript
const buffer = await RpcUtils.generateBuffer(
    'myProtoFile', 'MyMessage', { id: 1, name: 'exemplo' }
);

if (buffer) {
    // buffer está pronto para ser enviado via WebSocket ou salvo
} else {
    // lidar com erro
}
```

## Pack

### ``pack(contractName: string, messageName: string, data?: any): Promise<Uint8Array | null>``

O método ``pack`` prepara uma mensagem codificada em Protobuf para comunicação RPC. Ele inclui metadados como os índices de contrato e mensagem, junto com os dados reais da mensagem.

**Parâmetros:**
* **contractName (``string``):** O nome do contrato associado à mensagem.
* **messageName (``string``):** O nome da mensagem dentro do contrato.
* **data (``any``, opcional):** Os dados a serem incluídos na mensagem, que serão codificados como Protobuf.

**Retorno:**
* **``Promise<Uint8Array | null>``:** Uma promessa que resolve em um ``Uint8Array`` representando a mensagem empacotada, ou ``null`` se ocorrer um erro.

```typescript
const packedMessage = await RpcUtils.pack(
    'TaskContract', 'AddTaskRequest', { taskId: 1, taskLabel: 'Nova Tarefa' }
);

if (packedMessage) {
    // mensagem empacotada pronta para ser enviada via WebSocket ou outro protocolo binário
} else {
    // lidar com erro
}
```
