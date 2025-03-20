# RPC

Em aplicações web modernas, reduzir a sobrecarga e aumentar a eficiência na comunicação é essencial. Uma abordagem comum em sistemas tradicionais é o uso de HTTP/JSON, mas esse método introduz ineficiências significativas:

**Sobrecarga do HTTP:** Os cabeçalhos HTTP, tanto em requisições quanto em respostas, frequentemente são maiores que o próprio payload, especialmente em trocas de dados pequenas. Isso leva a um consumo desnecessário de largura de banda, particularmente em aplicações em tempo real onde a sobrecarga é incorrida repetidamente.

**Ineficiências do JSON:** Embora o JSON seja legível por humanos e amplamente utilizado, ele é verboso e carece do desempenho necessário para sistemas de alto rendimento. Analisar e serializar JSON adiciona uma sobrecarga computacional extra em comparação com formatos binários, tornando-o suboptimal para sistemas que exigem tempos de resposta rápidos e alta concorrência.

### Por que o CMMV usa RPC e WebSocket/Protobuf

O WebSocket fornece uma comunicação persistente e bidirecional entre o servidor e o cliente, reduzindo a necessidade de estabelecer e encerrar conexões repetidamente, como visto no HTTP. Isso por si só reduz significativamente a sobrecarga ao eliminar a reestabelecimento de conexão, cabeçalhos e outros metadados exigidos pelo HTTP.

No entanto, o CMMV não para no WebSocket. Ele também utiliza Protocol Buffers (Protobuf) para codificação de dados. O Protobuf é um formato de serialização binária que reduz significativamente o tamanho das mensagens trocadas, pois usa uma representação binária compacta, ao contrário do formato de texto verboso do JSON. Isso leva a:

* **Tamanhos de Payload Reduzidos:** O Protobuf é muito mais eficiente na codificação de dados, reduzindo o tamanho tanto das requisições quanto das respostas, resultando em uma transferência de dados mais rápida.

* **Velocidade Aprimorada:** Formatos binários como o Protobuf são mais rápidos para serializar e desserializar em comparação com o JSON. Isso melhora o desempenho tanto no lado do servidor (processando múltiplas requisições) quanto no lado do cliente (renderização ou interação mais rápidas).

* **Imposição de Esquema:** Diferente do JSON, que é flexível mas propenso a erros, o Protobuf impõe um esquema estruturado. Isso garante que os dados enviados e recebidos sejam bem definidos e consistentes, prevenindo discrepâncias e facilitando a manutenção e extensão ao longo do tempo.

### HTTP/JSON vs. WebSocket/Protobuf

* **Latência:** WebSocket/Protobuf reduz a latência nas interações cliente-servidor, permitindo respostas quase em tempo real. Ele evita a sobrecarga do HTTP sem estado e a verbosidade do JSON.

* **Eficiência:** Conexões WebSocket permanecem abertas, permitindo a troca contínua de dados sem reestabelecer conexões para cada transação. O Protobuf aprimora isso ao garantir que os dados trocados sejam mínimos, levando a uma melhor utilização da largura de banda.

* **Escalabilidade:** Sistemas que utilizam WebSocket/Protobuf escalam de forma mais eficaz porque reduzem tanto a sobrecarga de rede quanto a computacional. Isso se torna crítico em aplicações que precisam lidar com muitos clientes simultaneamente, como jogos multiplayer ou plataformas de análise em tempo real.

Em cenários onde a comunicação em tempo real, como jogos, plataformas de negociação financeira ou sistemas IoT, é necessária, o WebSocket/Protobuf supera o HTTP/JSON tradicional devido à sua capacidade de lidar com muitas conexões simultâneas com menor latência e melhor throughput de dados.

Ao escolher RPC via WebSocket/Protobuf como o protocolo de comunicação padrão, o CMMV garante que os desenvolvedores possam construir aplicações eficientes e escaláveis sem o peso da sobrecarga do HTTP ou da ineficiência do JSON, levando a sistemas mais rápidos e confiáveis.

## Protobuf

Protocol Buffers (Protobuf) é um formato de serialização binária neutro em relação a linguagem e plataforma, desenvolvido pelo Google. No CMMV, o Protobuf foi selecionado como a camada de comunicação devido à sua eficiência, estrutura e benefícios de desempenho em relação a alternativas como JSON ou XML.

* **Compacto e Eficiente:** O Protobuf codifica dados em formato binário, reduzindo significativamente o tamanho dos dados transmitidos em comparação com formatos de texto como JSON. Isso leva a uma transmissão mais rápida, crucial para aplicações em tempo real como jogos ou sistemas financeiros.

* **Velocidade:** A serialização binária no Protobuf é muito mais rápida que a serialização/desserialização do JSON, proporcionando um processamento de dados mais rápido e reduzindo a carga computacional.

* **Imposição de Esquema:** O Protobuf exige um esquema predefinido para os dados, garantindo que as estruturas de dados sejam fortemente tipadas e versionadas. Isso evita inconsistências durante a comunicação cliente-servidor, melhorando a confiabilidade em sistemas distribuídos.

* **Agnóstico a Linguagem e Plataforma:** O Protobuf suporta múltiplas linguagens de programação e plataformas, permitindo que as aplicações CMMV permaneçam flexíveis em diferentes ambientes.

* **Gerenciamento Eficiente de Sobrecarga:** O Protobuf permite um uso mais eficiente da largura de banda e menor uso de CPU, tornando-o ideal para sistemas que requerem alto rendimento e baixa latência, como a comunicação baseada em RPC no CMMV.

Ao usar o Protobuf, o CMMV garante um desempenho otimizado na serialização de dados, tornando a comunicação eficiente, escalável e confiável em uma variedade de aplicações.

## Instalação

Para implementar comunicação WebSocket usando Protobuf no CMMV, siga estes passos:

```bash
$ pnpm add @cmmv/protobuf @cmmv/ws protobufjs
```

* Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/protobuf](https://github.com/cmmvio/cmmv/tree/main/packages/protobuf)
* Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/ws](https://github.com/cmmvio/cmmv/tree/main/packages/ws)
<br/>

Configuração da aplicação:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { ProtobufModule } from "@cmmv/protobuf";
import { WSModule, WSAdapter } from "@cmmv/ws";

Application.create({
    httpAdapter: DefaultAdapter,
    wsAdapter: WSAdapter,
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule
    ],
    contracts: [...],
});
```

<br/>

* **WSAdapter:** Gerencia conexões WebSocket.
* **ProtobufModule:** Define estruturas de mensagens com Protocol Buffers para comunicação.
* **WSModule:** Gerencia conexões WebSocket, utilizando mensagens Protobuf para transmissão eficiente de dados.

Configurações ``.cmmv.config.cjs``

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

No CMMV, os contratos são processados no formato .proto e armazenados no diretório ``/src/proto`` junto com os tipos TypeScript. Esses contratos são carregados no frontend usando ``protobufjs`` de duas maneiras:

**Pré-carregamento de Contratos:** Definir ``preLoadContracts = true`` converte todos os contratos em JSON, que são então incluídos no pacote da aplicação. Isso permite um cache eficiente, especialmente por meio de CDNs.

**Carregamento Sob Demanda:** Definir ``preLoadContracts = false`` carrega os arquivos .proto conforme necessário ao receber a primeira mensagem que exige o contrato, armazenando-os em cache localmente para uso futuro. Essa abordagem é útil para aplicações com numerosos contratos.

## Integração

O framework CMMV simplifica a comunicação no frontend ao vincular métodos Protobuf diretamente ao contexto da visão. Isso permite que os desenvolvedores invoquem métodos RPC como ``AddTaskRequest`` e ``DeleteTaskRequest`` dentro de suas visões de forma transparente, como demonstrado no exemplo de lista de tarefas.

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
            { name: "description", content: "Amostra de Lista de Tarefas CMMV" },
            { name: "keywords", content: "cmmv, modelo de contrato, websocket" }
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

A integração do Protobuf com o transpilador do CMMV oferece vantagens significativas sobre métodos tradicionais como o ``protoc``. Enquanto o ``protoc`` gera um grande volume de código repetitivo, muito dele pode ser desnecessário dependendo do escopo do projeto. O CMMV integra automaticamente os contratos com serviços como repositórios, caches e outros módulos, simplificando o processo de desenvolvimento. Em vez de invocar manualmente o código gerado, o CMMV injeta funções Protobuf diretamente na camada de visão, reduzindo a sobrecarga e evitando a geração de código redundante, tornando-o mais eficiente para aplicações web modernas.

Essa abordagem simplificada não apenas reduz a complexidade, mas também melhora o desempenho ao evitar arquivos intermediários e serviços desnecessários, focando exclusivamente nos requisitos funcionais.
