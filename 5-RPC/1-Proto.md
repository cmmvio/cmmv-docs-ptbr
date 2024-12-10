# Proto

The CMMV framework provides seamless integration between server-side contracts and frontend applications using Protobuf as the communication protocol. By defining contracts in a structured way, CMMV automatically generates [Protobuf](https://protobuf.dev/) files and TypeScript types, allowing for a smooth and efficient integration of RPC-based communication.

In this section, we'll walk through how the contract definition in the backend is transformed into [Protobuf schema](https://protobuf.dev/programming-guides/proto3/), TypeScript types, and integrated into the frontend for real-time binary communication.

In CMMV, contracts are defined using the ``@Contract`` and ``@ContractField`` decorators. Below is an example contract for managing tasks:

```typescript
import { 
	AbstractContract, Contract, 
	ContractField 
} from "@cmmv/core";

@Contract({
    controllerName: "Task",
    protoPath: "src/protos/task.proto",
    protoPackage: "task"
})
export class TasksContract extends AbstractContract {
    @ContractField({ 
        protoType: 'string', 
        unique: true 
    })
    label: string;

    @ContractField({ 
        protoType: 'bool', 
        defaultValue: false 
    })
    checked: boolean;

    @ContractField({ 
        protoType: 'bool', 
        defaultValue: false 
    })
    removed: boolean;
}
```

<br/>

* **@Contract:** Defines the metadata for the contract, including the name of the controller, the database type (mongodb), the path to the Protobuf file, and the Protobuf package name.
* **@ContractField:** Defines individual fields of the contract, specifying their Protobuf type and any additional constraints (e.g., unique values, default values).

## .proto

Based on the contract above, CMMV will automatically generate the corresponding .proto file and associated TypeScript types. This generated Protobuf file allows for binary communication between the server and client.

```protobuf
// Proto generated automatically by CMMV

syntax = "proto3";

package task;

message Task {
  string label = 1;
  bool checked = 2;
  bool removed = 3;
}

message AddTaskRequest {
  Task item = 1;
}

message AddTaskResponse {
  Task item = 1;
}

message UpdateTaskRequest {
  string id = 1;
  Task item = 2;
}

message UpdateTaskResponse {
  Task item = 1;
}

message DeleteTaskRequest {
  string id = 1;
}

message DeleteTaskResponse {
  bool success = 1;
}

message GetAllTaskRequest {}

message GetAllTaskResponse {
  repeated Task items = 1;
}
```

## Types 

CMMV also generates TypeScript types for the Protobuf messages, ensuring strong typing across the entire application. These types are generated alongside the Protobuf files, and look like the following:

```typescript
// Types generated automatically by CMMV

export namespace Task {
  export type label = string;
  export type checked = boolean;
  export type removed = boolean;
}

export interface AddTaskRequest {
  item: Task;
}

export interface AddTaskResponse {
  item: Task;
}

export interface UpdateTaskRequest {
  id: string;
  item: Task;
}

export interface UpdateTaskResponse {
  item: Task;
}

export interface DeleteTaskRequest {
  id: string;
}

export interface DeleteTaskResponse {
  success: boolean;
}

export interface GetAllTaskRequest {}
export interface GetAllTaskResponse {
  items: Task[];
}
```

On the frontend, CMMV automatically integrates the Protobuf schema for seamless communication. With WebSocket support and binary communication through Protobuf, the frontend can make RPC (Remote Procedure Call) requests to the server using the defined contract methods.

By using CMMV's auto-generated functions, such as ``AddTaskRequest``, ``UpdateTaskRequest``, etc., developers can interact with the backend contract without writing additional code for parsing and validation. The functions are available directly in the context of the view, enabling effortless interaction from the UI.

## Frontend

On the frontend, CMMV automatically integrates the Protobuf schema for seamless communication. With WebSocket support and binary communication through Protobuf, the frontend can make RPC (Remote Procedure Call) requests to the server using the defined contract methods.

By using CMMV's auto-generated functions, such as ``AddTaskRequest``, ``UpdateTaskRequest``, etc., developers can interact with the backend contract without writing additional code for parsing and validation. The functions are available directly in the context of the view, enabling effortless interaction from the UI.

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
                  :class="{'todo-item-checked': item.checked }"
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

        AddTaskResponse(data) { 
          this.UpdateTaskResponse(data); 
        },

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

CMMV provides flexibility in loading the Protobuf contracts:

* **PreLoad Contracts:** By default, all contracts are pre-processed, converted into JSON, and included in the main JavaScript bundle. This allows for faster execution at runtime since the contracts are already available, and the frontend can immediately start making RPC calls.
* **On-Demand Loading:** In cases where there are many contracts, or if you want to optimize the initial page load, you can set preLoadContracts = false. This way, contracts are fetched on-demand when they are needed, caching them locally to avoid multiple network requests.

By automatically generating Protobuf contracts, TypeScript types, and integrating them into the frontend, CMMV simplifies communication between the server and client. The use of WebSocket and Protobuf ensures fast, efficient, and type-safe communication, reducing the overhead associated with traditional HTTP/JSON-based systems.