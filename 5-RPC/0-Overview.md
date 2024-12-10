# RPC

In modern web applications, reducing overhead and increasing efficiency in communication is essential. A common approach in traditional systems is the use of HTTP/JSON, but this method introduces significant inefficiencies:

HTTP Overhead: HTTP headers in both requests and responses are often larger than the payload itself, especially for small data exchanges. This leads to unnecessary bandwidth consumption, particularly in real-time applications where the overhead is repeatedly incurred.

JSON's Inefficiencies: Although JSON is human-readable and widely used, it is verbose and lacks the performance needed for high-throughput systems. Parsing and serializing JSON adds extra computational overhead compared to binary formats, making it suboptimal for systems requiring quick response times and high concurrency.

Why CMMV Uses RPC and WebSocket/Protobuf

WebSocket provides persistent, full-duplex communication between the server and the client, reducing the need to repeatedly establish and tear down connections as seen in HTTP. This alone cuts down significant overhead by eliminating connection re-establishment, headers, and other metadata required by HTTP.

However, CMMV doesn't stop at WebSockets. It also leverages Protocol Buffers (Protobuf) for encoding data. Protobuf is a binary serialization format that significantly reduces the size of the messages exchanged, as it uses a compact binary representation, unlike the verbose text format of JSON. This leads to:

* **Reduced Payload Sizes:** Protobuf is much more efficient at encoding data, reducing both the size of requests and responses, resulting in faster data transfer.

* **Improved Speed:** Binary formats like Protobuf are faster to serialize and deserialize compared to JSON. This improves both performance on the server side (processing multiple requests) and the client side (faster rendering or interaction).

* **Schema Enforcement:** Unlike JSON, which is flexible but prone to errors, Protobuf enforces a structured schema. This ensures that the data being sent and received is well-defined and consistent, preventing mismatches and making it easier to maintain and extend over time.

**HTTP/JSON vs. WebSocket/Protobuf**

* **Latency:** WebSocket/Protobuf reduces latency in client-server interactions, enabling near real-time responses. It avoids the overhead of stateless HTTP and the verbosity of JSON.

* **Efficiency:** WebSocket connections remain open, allowing for continuous data exchange without re-establishing connections for each transaction. Protobuf further enhances this by ensuring the data exchanged is minimal, leading to better bandwidth utilization.

* **Scalability:** Systems using WebSocket/Protobuf scale more effectively because they reduce both network and computational overhead. This becomes critical in applications that need to handle many clients simultaneously, such as multiplayer games or real-time analytics platforms.

In scenarios where real-time communication, such as gaming, financial trading platforms, or IoT systems, is required, WebSocket/Protobuf excels over traditional HTTP/JSON due to its ability to handle many concurrent connections with lower latency and better data throughput.

By choosing RPC via WebSocket/Protobuf as the default communication protocol, CMMV ensures that developers can build efficient, scalable applications without the burden of HTTP’s overhead or JSON’s inefficiency, leading to faster, more reliable systems.

# Protobuf 

Protocol Buffers (Protobuf) is a language-neutral, platform-neutral binary serialization format developed by Google. In CMMV, Protobuf was selected as the communication layer due to its efficiency, structure, and performance benefits over alternatives like JSON or XML.

* **Compact and Efficient:** Protobuf encodes data in a binary format, significantly reducing the size of transmitted data compared to text formats like JSON. This leads to faster transmission, crucial for real-time applications like gaming or financial systems.

* **Speed:** Binary serialization in Protobuf is far quicker than JSON serialization/deserialization, providing faster data processing and reducing computational load.

* **Schema Enforcement:** Protobuf requires a predefined schema for data, ensuring that data structures are strictly typed and versioned. This avoids inconsistencies during client-server communication, improving reliability in distributed systems.

* **Language and Platform Agnostic:** Protobuf supports multiple programming languages and platforms, allowing CMMV applications to remain flexible across different environments.

* **Efficient Overhead Management:** Protobuf allows for more efficient use of bandwidth and lower CPU usage, making it ideal for systems requiring high throughput and low latency, like RPC-based communication in CMMV.

By using Protobuf, CMMV ensures optimal performance in data serialization, making communication efficient, scalable, and reliable across a range of applications.

## Instalation

To implement WebSocket communication using Protobuf in CMMV, follow these steps:

```bash
$ pnpm add @cmmv/protobuf @cmmv/ws protobufjs 
```

Application setup:

```typescript
import { Application } from "@cmmv/core";
import { ExpressAdapter, ExpressModule } from "@cmmv/http";
import { ProtobufModule } from "@cmmv/protobuf";
import { WSModule, WSAdapter } from "@cmmv/ws";
import { ViewModule } from "@cmmv/view";

Application.create({
    httpAdapter: ExpressAdapter,    
    wsAdapter: WSAdapter,
    modules: [
        ExpressModule,
        ProtobufModule,
        WSModule,
        ViewModule
    ],
    contracts: [...],
});
```

<br/>

* **WSAdapter:** Handles WebSocket connections.
* **ProtobufModule:** Defines message structures with Protocol Buffers for communication.
* **WSModule:** Manages WebSocket connections, utilizing Protobuf messages for efficient data transmission.

Settings ``.cmmv.config.js``

```typescript
module.exports = {
    server: {
        host: process.env.HOST || "0.0.0.0",
        port: process.env.PORT || 3000,
        ...
    },

    rpc: {
        enabled: true,
        preLoadContracts: true
    },

    ...
};
```

In CMMV, contracts are processed in .proto format and stored in the /src/proto directory along with TypeScript types. These contracts are loaded into the frontend using protobufjs in two ways:

**Preloading Contracts:** Setting preLoadContracts = true converts all contracts to JSON, which are then bundled with the application. This allows for efficient caching, especially through CDNs.

**On-Demand Loading:** Setting preLoadContracts = false loads .proto files as needed upon receiving the first message that requires the contract, caching them locally for future use. This approach is useful for applications with numerous contracts.

## Integration 

The CMMV framework simplifies communication in the frontend by binding Protobuf methods directly to the view context. This allows developers to invoke RPC methods like AddTaskRequest and DeleteTaskRequest within their views seamlessly, as demonstrated in the example to-do list.

```html
<div class="todo-box" scope>
    <h1 s-i18n="todo"></h1>

    <div 
        c-show="todolist?.length > 0" 
        s:todolist="services.task.getAll()"
    >
        <div 
            c-show="todolist"
            c-for="(item, key) in todolist"
            class="todo-item"
        >
            <div class="todo-item-content">
                <input 
                    type="checkbox" 
                    c-model="item.checked" 
                    @change="UpdateTaskRequest(item)"
                ></input>

                <label 
                    :class="{'todo-item-checked': item.checked}"
                >{{ item.label }}</label>
            </div>
            
            <button 
                class="todo-btn-remove"
                s-i18n="remove" 
                @click="DeleteTaskRequest(item.id)"
            ></button>
        </div>
    </div>

    <div class="todo-input-box">
        <input c-model="label" class="todo-input">

        <button 
            class="todo-btn-add"
            s-i18n="add" 
            @click="addTask"
        ></button>
    </div>

    <pre>{{ todolist }}</pre>
</div>

<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "CMMV Todolist sample" },
            { name: "keywords", content: "cmmv, contract model, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ]
    },

    data(){
        return {
            todolist: [],
            label: ""
        }
    },

    methods: {
        addTask(){
            this.AddTaskRequest({ label: this.label });
            this.label = '';
        },

        DeleteTaskResponse(data){
            if (data.success) {
                const index = this.todolist.findIndex(
                    item => item.id === data.id
                );

                if (index !== -1) 
                    this.todolist.splice(index, 1);
            }
        },

        AddTaskResponse(data) { this.UpdateTaskResponse(data); },

        UpdateTaskResponse(data) {
            const index = this.todolist.findIndex(
                item => item.id === data.id
            );
            
            if (index !== -1) 
                this.todolist[index] = { ...data.item, id: data.id };
            else 
                this.todolist.push({ ...data.item, id: data.id });
        } 
    }
}
</script>
```

The integration of Protobuf with CMMV’s transpiler offers significant advantages over traditional methods like protoc. While protoc generates a large volume of boilerplate code, much of it may be unnecessary depending on the project’s scope. CMMV automatically integrates contracts with services like repositories, caches, and other modules, simplifying the development process. Instead of manually invoking generated code, CMMV injects Protobuf functions directly into the view layer, reducing overhead and avoiding redundant code generation, making it more efficient for modern web applications.

This streamlined approach not only reduces complexity but also enhances performance by avoiding unnecessary intermediate files and services, focusing solely on the functional requirements.