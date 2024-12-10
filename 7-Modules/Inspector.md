# Inspector

The ``@cmmv/inspector`` module provides tools for runtime performance profiling and debugging for Node.js applications. It integrates seamlessly with CMMV-based projects and utilizes the built-in node:inspector module to capture CPU profiles and heap snapshots. The module also provides utility methods to manage and persist profiling data, making it an essential tool for optimizing and debugging your applications.

## Installation

To install the ``@cmmv/inspector`` module, use the following command:

```bash
$ pnpm add @cmmv/inspector
```

## Features

* **CPU Profiling:** Start and stop CPU profiling to capture performance data.
* **Heap Snapshots:** Take and save heap snapshots to analyze memory usage and detect leaks.
* **Process Signal Handling:** Automatically handles cleanup during process termination.
* **Custom Finalization Hooks:** Register cleanup tasks with Inspector.once.
* **Data Persistence:** Save profiling data for analysis with compatible tools like Chrome DevTools.

## Examples

You can start and stop CPU profiling to capture performance metrics during specific operations.

```typescript
import { Inspector } from '@cmmv/inspector';

async function runProfiler() {
    await Inspector.start();

    // Perform some operations
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('CPU Profile saved!');
}

runProfiler();
```

## Bind Process 

Ensure proper cleanup during process termination by binding kill signals.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.bindKillProcess();

// Perform operations
```

## Custom Finalization

Register cleanup tasks to execute before the process exits.

```typescript
import { Inspector } from '@cmmv/inspector';

Inspector.once(async () => {
    console.log('Performing cleanup: Saving heap snapshot...');
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Heap snapshot saved!');
});

// Perform operations
```

## Heap Snapshot

Take and save a heap snapshot to analyze memory usage.

```typescript
import { Inspector } from '@cmmv/inspector';

async function captureHeapSnapshot() {
    await Inspector.takeHeapSnapshot('./snapshots');
    console.log('Heap snapshot saved!');
}

captureHeapSnapshot();
```


## API Reference

### ``Inspector.start(): Promise<void>``

Starts the CPU profiler.

```typescript
await Inspector.start();
```

### ``Inspector.stop(): Promise<void>``

Stops the CPU profiler and disconnects the session.

```typescript
await Inspector.stop();
```

### ``Inspector.saveProfile(dirPath: string, restart: boolean = true): Promise<void>``

Saves the CPU profile to the specified directory. Optionally restarts the profiler after saving.

```typescript
await Inspector.saveProfile('./profiles');
```

### ``Inspector.takeHeapSnapshot(dirPath: string): Promise<void>``

Takes a heap snapshot and saves it to the specified directory for memory analysis.

```typescript
await Inspector.takeHeapSnapshot('./snapshots');
```

### ``Inspector.bindKillProcess(): void``

Binds process termination signals to ensure proper finalization of profiling tasks.

```typescript
Inspector.bindKillProcess();
```

### ``Inspector.once(callback: () => Promise<void>): void``

Registers a one-time finalization hook to execute before process termination.

```typescript
Inspector.once(async () => {
    console.log('Performing cleanup...');
});
```

## Workflow 

The following workflow demonstrates how to use the ``@cmmv/inspector`` module to start profiling, capture a heap snapshot, and stop profiling during process termination.

```typescript
import { Inspector } from '@cmmv/inspector';

async function main() {
    // Register cleanup task
    Inspector.once(async () => {
        console.log('Performing cleanup: Saving heap snapshot...');
        await Inspector.takeHeapSnapshot('./snapshots');
        console.log('Heap snapshot saved!');
    });

    // Bind process signals
    Inspector.bindKillProcess();

    // Start profiling
    await Inspector.start();

    // Perform operations
    for (let i = 0; i < 1e6; i++) {
        Math.sqrt(i);
    }

    // Stop profiling and save CPU profile
    await Inspector.stop();
    await Inspector.saveProfile('./profiles');
    console.log('Profile saved!');
}

main();
```

The ``@cmmv/inspector`` module is an indispensable tool for debugging and performance tuning in CMMV-based projects, providing a streamlined and powerful interface for capturing and managing profiling data.