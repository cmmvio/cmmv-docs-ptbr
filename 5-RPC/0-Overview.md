# RPC

Em aplicações web modernas, reduzir a sobrecarga e aumentar a eficiência na comunicação é essencial. Uma abordagem comum em sistemas tradicionais é o uso de HTTP/JSON, mas esse método introduz ineficiências significativas:

Sobrecarga do HTTP: Os cabeçalhos HTTP em solicitações e respostas são frequentemente maiores do que o próprio payload, especialmente para pequenas trocas de dados. Isso leva a um consumo desnecessário de largura de banda, particularmente em aplicações em tempo real, onde a sobrecarga é incorrida repetidamente.

Ineficiências do JSON: Embora o JSON seja legível por humanos e amplamente utilizado, ele é verboso e carece de desempenho necessário para sistemas de alta taxa de transferência. Analisar e serializar JSON adiciona uma sobrecarga computacional extra em comparação com formatos binários, tornando-o subótimo para sistemas que exigem tempos de resposta rápidos e alta concorrência.

Por que o CMMV usa RPC e WebSocket/Protobuf

O WebSocket fornece comunicação persistente e full-duplex entre o servidor e o cliente, reduzindo a necessidade de estabelecer e encerrar conexões repetidamente, como ocorre no HTTP. Isso, por si só, reduz significativamente a sobrecarga ao eliminar o reestabelecimento de conexões, cabeçalhos e outros metadados exigidos pelo HTTP.

No entanto, o CMMV não se limita ao uso de WebSockets. Ele também utiliza Protocol Buffers (Protobuf) para codificação de dados. Protobuf é um formato de serialização binário que reduz significativamente o tamanho das mensagens trocadas, pois utiliza uma representação binária compacta, ao contrário do formato de texto verboso do JSON. Isso leva a:

* **Tamanhos de Payload Reduzidos:** Protobuf é muito mais eficiente na codificação de dados, reduzindo tanto o tamanho das solicitações quanto das respostas, resultando em transferências de dados mais rápidas.

* **Velocidade Melhorada:** Formatos binários como Protobuf são mais rápidos para serializar e desserializar em comparação com JSON. Isso melhora o desempenho tanto no lado do servidor (processando múltiplas solicitações) quanto no lado do cliente (renderização ou interação mais rápidas).

* **Aplicação de Esquema:** Diferente do JSON, que é flexível mas propenso a erros, o Protobuf aplica um esquema estruturado. Isso garante que os dados enviados e recebidos sejam bem definidos e consistentes, evitando incompatibilidades e tornando mais fácil a manutenção e expansão ao longo do tempo.

**HTTP/JSON vs. WebSocket/Protobuf**

* **Latência:** WebSocket/Protobuf reduz a latência nas interações cliente-servidor, permitindo respostas quase em tempo real. Ele evita a sobrecarga de HTTP sem estado e a verbosidade do JSON.

* **Eficiência:** Conexões WebSocket permanecem abertas, permitindo troca contínua de dados sem reestabelecer conexões para cada transação. O Protobuf melhora ainda mais isso ao garantir que os dados trocados sejam mínimos, levando a uma melhor utilização da largura de banda.

* **Escalabilidade:** Sistemas que utilizam WebSocket/Protobuf escalam de maneira mais eficaz, pois reduzem a sobrecarga tanto de rede quanto computacional. Isso se torna crítico em aplicações que precisam lidar com muitos clientes simultaneamente, como jogos multiplayer ou plataformas de análises em tempo real.

Em cenários onde a comunicação em tempo real, como jogos, plataformas de negociação financeira ou sistemas IoT, é necessária, WebSocket/Protobuf supera o HTTP/JSON tradicional devido à sua capacidade de lidar com muitas conexões simultâneas com menor latência e melhor taxa de transferência de dados.

Ao escolher RPC via WebSocket/Protobuf como protocolo de comunicação padrão, o CMMV garante que os desenvolvedores possam construir aplicações eficientes e escaláveis sem o peso da sobrecarga do HTTP ou da ineficiência do JSON, resultando em sistemas mais rápidos e confiáveis.

## Protobuf

Protocol Buffers (Protobuf) é um formato de serialização binária neutro em linguagem e plataforma desenvolvido pelo Google. No CMMV, o Protobuf foi escolhido como camada de comunicação devido à sua eficiência, estrutura e benefícios de desempenho em relação a alternativas como JSON ou XML.

* **Compacto e Eficiente:** Protobuf codifica dados em um formato binário, reduzindo significativamente o tamanho dos dados transmitidos em comparação com formatos de texto como JSON. Isso resulta em transmissões mais rápidas, cruciais para aplicações em tempo real como jogos ou sistemas financeiros.

* **Velocidade:** A serialização binária no Protobuf é muito mais rápida do que a serialização/desserialização em JSON, proporcionando um processamento de dados mais rápido e reduzindo a carga computacional.

* **Aplicação de Esquema:** Protobuf exige um esquema pré-definido para os dados, garantindo que as estruturas de dados sejam estritamente tipadas e versionadas. Isso evita inconsistências durante a comunicação cliente-servidor, melhorando a confiabilidade em sistemas distribuídos.

* **Agnóstico de Linguagem e Plataforma:** Protobuf suporta múltiplas linguagens de programação e plataformas, permitindo que aplicações CMMV permaneçam flexíveis em diferentes ambientes.

* **Gestão Eficiente de Sobrecarga:** Protobuf permite um uso mais eficiente de largura de banda e menor uso de CPU, tornando-o ideal para sistemas que exigem alta taxa de transferência e baixa latência, como comunicação baseada em RPC no CMMV.

Ao usar Protobuf, o CMMV assegura desempenho ideal na serialização de dados, tornando a comunicação eficiente, escalável e confiável em uma ampla gama de aplicações.

## Instalação

Para implementar comunicação WebSocket utilizando Protobuf no CMMV, siga estes passos:

```bash
$ pnpm add @cmmv/protobuf @cmmv/ws protobufjs
```

* Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/protobuf](https://github.com/cmmvio/cmmv/tree/main/packages/protobuf)
* Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/ws](https://github.com/cmmvio/cmmv/tree/main/packages/ws)

Configuração da aplicação:

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

* **WSAdapter:** Gerencia as conexões WebSocket.
* **ProtobufModule:** Define estruturas de mensagens com Protocol Buffers para comunicação.
* **WSModule:** Gerencia conexões WebSocket, utilizando mensagens Protobuf para transmissão eficiente de dados.

Configuração no arquivo `.cmmv.config.cjs`:

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

No CMMV, contratos são processados no formato .proto e armazenados no diretório `/src/proto`, juntamente com tipos TypeScript. Estes contratos podem ser carregados no frontend de duas maneiras:

* **Carregamento Antecipado de Contratos:** Definir `preLoadContracts = true` converte todos os contratos para JSON, que são então empacotados com a aplicação. Isso permite caching eficiente, especialmente por meio de CDNs.

* **Carregamento Sob Demanda:** Definir `preLoadContracts = false` carrega os arquivos .proto conforme necessário ao receber a primeira mensagem que exige o contrato, armazenando-os em cache local para uso futuro. Este método é útil para aplicações com inúmeros contratos.

## Integração

O framework CMMV simplifica a comunicação no frontend ao vincular métodos Protobuf diretamente ao contexto da view. Isso permite que desenvolvedores invoquem métodos RPC como `AddTaskRequest` e `DeleteTaskRequest` dentro das views de forma fluida, como demonstrado no exemplo de lista de tarefas:

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
```

A integração do Protobuf com o transpiler do CMMV oferece vantagens significativas sobre métodos tradicionais, como o `protoc`. Enquanto o `protoc` gera um grande volume de código boilerplate, muitas vezes desnecessário dependendo do escopo do projeto, o CMMV integra automaticamente os contratos com serviços como repositórios, caches e outros módulos.

Em vez de invocar manualmente o código gerado, o CMMV injeta as funções Protobuf diretamente na camada de visualização, reduzindo a sobrecarga e evitando a geração de código redundante. Isso torna o processo mais eficiente para aplicações web modernas.

Essa abordagem simplificada não só reduz a complexidade, mas também melhora o desempenho ao evitar arquivos intermediários e serviços desnecessários, concentrando-se exclusivamente nos requisitos funcionais.
