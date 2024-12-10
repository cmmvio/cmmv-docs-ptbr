# Telemetry

The telemetry system within the CMMV framework provides a powerful mechanism for tracking and monitoring internal processes both in the **backend** and **frontend**. It is designed to measure the performance of various components such as database queries, service calls, API responses, and rendering times. By utilizing this telemetry system, developers can identify potential bottlenecks or performance issues in the system.

The **Telemetry** class, a singleton, manages telemetry records for processes in both server and client environments. It tracks the start and end times of labeled processes, allowing the calculation of the time spent on each task. These records can be accessed, monitored, and displayed in real-time, particularly in development mode, to help developers analyze where slowdowns or inefficiencies are occurring.

| Method                                      | Description                                                                                           |
|---------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `start(label: string, requestId?: string)`  | Starts a telemetry record for a process with a given label and request ID.                            |
| `end(label: string, requestId?: string)`    | Ends the telemetry record by marking the end time for the process with the given label.               |
| `getTelemetry(requestId: string)`           | Returns all telemetry records for a given request ID.                                                 |
| `clearTelemetry(requestId: string)`         | Clears telemetry records for a specific request ID.                                                   |
| `registerPlugin(plugin: any)`               | Allows external plugins to register and extend the telemetry system with additional capabilities.      |
| `getRecords()`                              | Retrieves all telemetry records currently stored.                                                     |

## Workflow

Start a Telemetry Record: When a request or action starts, the telemetry system logs the start time with the process label.

```typescript
Telemetry.start('TaskService::GetAll', requestId);
```

End the Telemetry Record: When the process completes, the telemetry system logs the end time.

```typescript
Telemetry.end('TaskService::GetAll', requestId);
```

Retrieve the Telemetry Data: After the request completes, you can retrieve all telemetry data associated with the request ID.

```typescript
const records = Telemetry.getTelemetry(requestId);
```

Clear the Telemetry Data: Once telemetry data is no longer needed, you can clear it.

```typescript
Telemetry.clearTelemetry(requestId);
```

In services that communicate with external components (e.g., databases, queues), it is recommended to implement telemetry points to track the execution time of these operations. This is particularly helpful when ``NODE_ENV`` is set to ``dev``, as it will log the telemetry data and display it in the console.

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

The telemetry system supports both backend and frontend processes. By implementing telemetry in both environments, you can get a complete view of the system's performance from request initiation to response and rendering.

**Example console output of telemetry:**

| Index | Process                            | Duration    |
|-------|------------------------------------|-------------|
| 0     | 'Server: Request Process'          | '35.00 ms'  |
| 1     | 'Server: Compile Template'         | '30.00 ms'  |
| 2     | 'Server: Load Includes'            | '0.00 ms'   |
| 3     | 'Server: TaskService::GetAll'      | '10.00 ms'  |
| 4     | 'Server: Process Setup'            | '17.00 ms'  |
| 5     | 'Client: Initialize Frontend'      | '0.30 ms'   |
| 6     | 'Client: WebSocket Initialization' | '0.30 ms'   |
| 7     | 'Client: Load Contracts'           | '1.30 ms'   |
| 8     | 'Client: Process Expressions'      | '5.30 ms'   |
| 9     | 'Client: CreateApp'                | '1.00 ms'   |
| 10    | 'Client: Mount App'                | '4.20 ms'   |

This output provides a detailed timeline of both server-side and client-side processes, allowing you to pinpoint slow areas in the system.

## Best Practices

* **Telemetry in Services:** Always implement telemetry for operations that involve external services or long-running tasks such as database queries or message queue operations.
* **Use in Development:** When ``NODE_ENV`` is set to dev, telemetry data is automatically sent to ``@cmmv/view`` and displayed in the console, allowing for immediate performance analysis.
* **Clear Telemetry:** Clear telemetry records when they are no longer needed to avoid memory leaks or excessive storage.

By using the telemetry system, you can gain full visibility into how long processes take and where slowdowns occur, whether in the backend or frontend of your application. This can help you optimize your system's performance and ensure smooth communication between components.