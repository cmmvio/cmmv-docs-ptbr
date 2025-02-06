# Telemetria

O sistema de telemetria no framework **CMMV** oferece uma poderosa ferramenta para monitorar e analisar processos internos tanto no **backend** quanto no **frontend**. Ele é projetado para medir o desempenho de componentes como consultas a bancos de dados, chamadas de serviços, respostas de APIs e tempos de renderização. Utilizando este sistema de telemetria, os desenvolvedores podem identificar possíveis gargalos ou problemas de desempenho no sistema.

A classe **Telemetry**, implementada como um singleton, gerencia registros de telemetria para processos nos ambientes de servidor e cliente. Ela acompanha os tempos de início e fim de processos rotulados, permitindo calcular o tempo gasto em cada tarefa. Esses registros podem ser acessados, monitorados e exibidos em tempo real, especialmente no modo de desenvolvimento, ajudando os desenvolvedores a analisar onde estão ocorrendo lentidões ou ineficiências.

| Método                                     | Descrição                                                                 |
|-------------------------------------------|---------------------------------------------------------------------------|
| `start(label: string, requestId?: string)` | Inicia um registro de telemetria para um processo com o rótulo especificado e ID opcional. |
| `end(label: string, requestId?: string)`   | Finaliza o registro de telemetria marcando o tempo de término para o processo. |
| `getTelemetry(requestId: string)`        | Retorna todos os registros de telemetria associados a um ID de solicitação. |
| `clearTelemetry(requestId: string)`      | Limpa os registros de telemetria para um ID de solicitação específico. |
| `registerPlugin(plugin: any)`            | Permite que plugins externos se registrem e estendam o sistema de telemetria com funcionalidades adicionais. |
| `getRecords()`                           | Recupera todos os registros de telemetria armazenados atualmente. |

## Fluxo de Trabalho

### Iniciar um Registro de Telemetria
Quando uma solicitação ou ação começa, o sistema de telemetria registra o tempo de início com o rótulo do processo.

```typescript
Telemetry.start('TaskService::GetAll', requestId);
```

### Finalizar o Registro de Telemetria
Quando o processo é concluído, o sistema de telemetria registra o tempo de término.

```typescript
Telemetry.end('TaskService::GetAll', requestId);
```

### Recuperar os Dados de Telemetria
Após a conclusão da solicitação, você pode recuperar todos os dados de telemetria associados ao ID da solicitação.

```typescript
const records = Telemetry.getTelemetry(requestId);
```

### Limpar os Dados de Telemetria
Quando os dados de telemetria não são mais necessários, você pode limpá-los.

```typescript
Telemetry.clearTelemetry(requestId);
```

Em serviços que se comunicam com componentes externos (ex.: bancos de dados, filas), é recomendável implementar pontos de telemetria para rastrear o tempo de execução dessas operações. Isso é especialmente útil quando `NODE_ENV` está configurado como `dev`, pois os dados de telemetria serão exibidos no console.

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

O sistema de telemetria suporta processos tanto no backend quanto no frontend. Implementando a telemetria em ambos os ambientes, você obtém uma visão completa do desempenho do sistema desde a solicitação inicial até a resposta e renderização.

### Exemplo de saída no console da telemetria:

| Índice | Processo                            | Duração     |
|--------|-------------------------------------|-------------|
| 0      | 'Server: Processamento da Solicitação' | '35.00 ms'  |
| 1      | 'Server: Compilar Template'         | '30.00 ms'  |
| 2      | 'Server: Carregar Includes'         | '0.00 ms'   |
| 3      | 'Server: TaskService::GetAll'       | '10.00 ms'  |
| 4      | 'Server: Configuração do Processo'  | '17.00 ms'  |
| 5      | 'Client: Inicializar Frontend'      | '0.30 ms'   |
| 6      | 'Client: Inicialização do WebSocket'| '0.30 ms'   |
| 7      | 'Client: Carregar Contratos'        | '1.30 ms'   |
| 8      | 'Client: Processar Expressões'      | '5.30 ms'   |
| 9      | 'Client: Criar Aplicação'           | '1.00 ms'   |
| 10     | 'Client: Montar Aplicação'          | '4.20 ms'   |

Essa saída fornece um detalhamento cronológico dos processos no servidor e no cliente, permitindo identificar áreas lentas no sistema.

## Melhores Práticas

* **Telemetria em Serviços:** Sempre implemente telemetria para operações que envolvam serviços externos ou tarefas de longa duração, como consultas ao banco de dados ou operações em filas.
* **Use no Desenvolvimento:** Quando `NODE_ENV` estiver configurado como `dev`, os dados de telemetria são automaticamente enviados para `@cmmv/view` e exibidos no console para análise imediata de desempenho.
* **Limpar Telemetria:** Limpe os registros de telemetria quando eles não forem mais necessários para evitar vazamentos de memória ou armazenamento excessivo.

Utilizando o sistema de telemetria, você pode obter total visibilidade sobre o tempo gasto nos processos e onde ocorrem lentidões, seja no backend ou no frontend da aplicação. Isso ajuda a otimizar o desempenho do sistema e garantir uma comunicação eficiente entre os componentes.
