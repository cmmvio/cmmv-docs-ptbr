# Proto

O framework CMMV oferece uma integração perfeita entre contratos do lado do servidor e aplicações frontend usando Protobuf como protocolo de comunicação. Ao definir contratos de forma estruturada, o CMMV gera automaticamente arquivos [Protobuf](https://protobuf.dev/) e tipos TypeScript, permitindo uma integração suave e eficiente da comunicação baseada em RPC.

Nesta seção, vamos explorar como a definição de contratos no backend é transformada em [esquema Protobuf](https://protobuf.dev/programming-guides/proto3/), tipos TypeScript e integrada ao frontend para comunicação binária em tempo real.

No CMMV, os contratos são definidos usando os decoradores ``@Contract`` e ``@ContractField``. Abaixo está um exemplo de contrato para gerenciar tarefas:

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

* **@Contract:** Define os metadados do contrato, incluindo o nome do controlador, o tipo de banco de dados (mongodb), o caminho para o arquivo Protobuf e o nome do pacote Protobuf.
* **@ContractField:** Define campos individuais do contrato, especificando seu tipo Protobuf e quaisquer restrições adicionais (por exemplo, valores únicos, valores padrão).

## .proto

Com base no contrato acima, o CMMV gerará automaticamente o arquivo .proto correspondente e os tipos TypeScript associados. Esse arquivo Protobuf gerado permite a comunicação binária entre o servidor e o cliente.

```protobuf
// Proto gerado automaticamente pelo CMMV

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

## Tipos

O CMMV também gera tipos TypeScript para as mensagens Protobuf, garantindo tipagem forte em toda a aplicação. Esses tipos são gerados junto com os arquivos Protobuf e se parecem com o seguinte:

```typescript
// Tipos gerados automaticamente pelo CMMV

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

No frontend, o CMMV integra automaticamente o esquema Protobuf para uma comunicação fluida. Com suporte ao WebSocket e comunicação binária por meio do Protobuf, o frontend pode fazer requisições RPC (Remote Procedure Call) ao servidor usando os métodos de contrato definidos.

Ao usar as funções geradas automaticamente pelo CMMV, como ``AddTaskRequest``, ``UpdateTaskRequest``, etc., os desenvolvedores podem interagir com o contrato do backend sem escrever código adicional para análise e validação. As funções estão disponíveis diretamente no contexto da visão, permitindo uma interação sem esforço a partir da interface do usuário.

## Frontend

No frontend, o CMMV integra automaticamente o esquema Protobuf para uma comunicação fluida. Com suporte ao WebSocket e comunicação binária por meio do Protobuf, o frontend pode fazer requisições RPC (Remote Procedure Call) ao servidor usando os métodos de contrato definidos.

Ao usar as funções geradas automaticamente pelo CMMV, como ``AddTaskRequest``, ``UpdateTaskRequest``, etc., os desenvolvedores podem interagir com o contrato do backend sem escrever código adicional para análise e validação. As funções estão disponíveis diretamente no contexto da visão, permitindo uma interação sem esforço a partir da interface do usuário.

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

O CMMV oferece flexibilidade no carregamento dos contratos Protobuf:

* **Pré-carregamento de Contratos:** Por padrão, todos os contratos são pré-processados, convertidos em JSON e incluídos no pacote JavaScript principal. Isso permite uma execução mais rápida em tempo de execução, pois os contratos já estão disponíveis, e o frontend pode começar imediatamente a fazer chamadas RPC.
* **Carregamento Sob Demanda:** Em casos onde há muitos contratos ou se você deseja otimizar o carregamento inicial da página, pode definir ``preLoadContracts = false``. Dessa forma, os contratos são buscados sob demanda quando necessários, sendo armazenados em cache localmente para evitar múltiplas requisições de rede.

Ao gerar automaticamente contratos Protobuf, tipos TypeScript e integrá-los ao frontend, o CMMV simplifica a comunicação entre o servidor e o cliente. O uso de WebSocket e Protobuf garante uma comunicação rápida, eficiente e com tipagem segura, reduzindo a sobrecarga associada a sistemas tradicionais baseados em HTTP/JSON.
