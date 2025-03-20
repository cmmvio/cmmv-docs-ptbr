# Telemetria

O sistema de telemetria dentro do framework CMMV fornece um mecanismo poderoso para rastrear e monitorar processos internos tanto no **backend** quanto no **frontend**. Ele foi projetado para medir o desempenho de diversos componentes, como consultas a bancos de dados, chamadas de serviços, respostas de APIs e tempos de renderização. Ao utilizar esse sistema de telemetria, os desenvolvedores podem identificar possíveis gargalos ou problemas de desempenho no sistema.

A classe **Telemetry**, um singleton, gerencia os registros de telemetria para processos em ambientes de servidor e cliente. Ela rastreia os tempos de início e término de processos rotulados, permitindo o cálculo do tempo gasto em cada tarefa. Esses registros podem ser acessados, monitorados e exibidos em tempo real, especialmente no modo de desenvolvimento, ajudando os desenvolvedores a analisar onde ocorrem lentidões ou ineficiências.

| Método                                    | Descrição                                                                 |
|-------------------------------------------|-------------------------------------------------------------------------|
| `start(label: string, requestId?: string)`  | Inicia um registro de telemetria para um processo com um determinado rótulo e ID de requisição. |
| `end(label: string, requestId?: string)`    | Finaliza o registro de telemetria marcando o tempo de término do processo com o rótulo fornecido. |
| `getTelemetry(requestId: string)`           | Retorna todos os registros de telemetria para um determinado ID de requisição. |
| `clearTelemetry(requestId: string)`         | Limpa os registros de telemetria para um ID de requisição específico. |
| `registerPlugin(plugin: any)`               | Permite que plugins externos registrem e estendam o sistema de telemetria com funcionalidades adicionais. |
| `getRecords()`                              | Recupera todos os registros de telemetria armazenados atualmente. |

## Fluxo de Trabalho

**Iniciar um Registro de Telemetria:** Quando uma requisição ou ação começa, o sistema de telemetria registra o tempo de início com o rótulo do processo.

```typescript
Telemetry.start('TaskService::GetAll', requestId);
```

**Finalizar o Registro de Telemetria:** Quando o processo é concluído, o sistema de telemetria registra o tempo de término.

```typescript
Telemetry.end('TaskService::GetAll', requestId);
```

**Recuperar os Dados de Telemetria:** Após a conclusão da requisição, você pode recuperar todos os dados de telemetria associados ao ID da requisição.

```typescript
const records = Telemetry.getTelemetry(requestId);
```

**Limpar os Dados de Telemetria:** Quando os dados de telemetria não são mais necessários, você pode limpá-los.

```typescript
Telemetry.clearTelemetry(requestId);
```

Em serviços que se comunicam com componentes externos (por exemplo, bancos de dados, filas de mensagens), é recomendável implementar pontos de telemetria para rastrear o tempo de execução dessas operações. Isso é especialmente útil quando o `NODE_ENV` está configurado como `dev`, pois os dados de telemetria serão registrados e exibidos no console.

```typescript
async getAll(req?: any): Promise<TaskEntity[]> {
    try {
        Telemetry.start('TaskService::GetAll', req?.requestId);

        const result = await Repository.findAll(TaskEntity);

        Telemetry.end('TaskService::GetAll', req?.requestId);
        return result;
    } catch (e) {
        Telemetry.end('TaskService::GetAll', req?.requestId);
        throw e;
    }
}
```

O sistema de telemetria suporta processos tanto no backend quanto no frontend. Ao implementar a telemetria em ambos os ambientes, é possível obter uma visão completa do desempenho do sistema, desde a iniciação da requisição até a resposta e renderização.

**Exemplo de saída do console da telemetria:**

| Índice | Processo                            | Duração     |
|--------|------------------------------------|-------------|
| 0      | 'Servidor: Processamento da Requisição'  | '35.00 ms'  |
| 1      | 'Servidor: Compilação de Template' | '30.00 ms'  |
| 2      | 'Servidor: Carregamento de Includes' | '0.00 ms'  |
| 3      | 'Servidor: TaskService::GetAll'   | '10.00 ms'  |
| 4      | 'Servidor: Configuração do Processo' | '17.00 ms'  |
| 5      | 'Cliente: Inicialização do Frontend' | '0.30 ms'   |
| 6      | 'Cliente: Inicialização do WebSocket' | '0.30 ms'   |
| 7      | 'Cliente: Carregamento de Contratos' | '1.30 ms'   |
| 8      | 'Cliente: Processamento de Expressões' | '5.30 ms'   |
| 9      | 'Cliente: Criação da Aplicação' | '1.00 ms'   |
| 10     | 'Cliente: Montagem da Aplicação' | '4.20 ms'   |

Essa saída fornece uma linha do tempo detalhada dos processos do lado do servidor e do cliente, permitindo identificar áreas lentas no sistema.

## Melhores Práticas

* **Telemetria em Serviços:** Sempre implemente telemetria para operações que envolvem serviços externos ou tarefas de longa duração, como consultas a bancos de dados ou operações em filas de mensagens.
* **Uso no Desenvolvimento:** Quando `NODE_ENV` está configurado como `dev`, os dados de telemetria são enviados automaticamente para `@cmmv/view` e exibidos no console, permitindo uma análise imediata de desempenho.
* **Limpeza de Telemetria:** Limpe os registros de telemetria quando eles não forem mais necessários para evitar vazamento de memória ou armazenamento excessivo.

Ao utilizar o sistema de telemetria, você pode obter total visibilidade sobre quanto tempo os processos levam e onde ocorrem lentidões, tanto no backend quanto no frontend da sua aplicação. Isso pode ajudá-lo a otimizar o desempenho do seu sistema e garantir uma comunicação eficiente entre os componentes.
