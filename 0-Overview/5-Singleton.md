# Singleton

In CMMV, the traditional dependency injection pattern is replaced by singleton registries, which streamline the management of global services. In many applications, modules often share services such as cache, queues, or database connections. Instead of declaring and injecting these services across multiple modules, a singleton ensures that only one instance of a service is created and shared across the entire application.

This approach offers several advantages:

* Simplifies service management by eliminating the need for explicit injection across modules.
* Reduces complexity by avoiding circular dependencies.
* Optimizes performance by ensuring only one instance of each service exists, minimizing overhead.

## Example

```typescript
import { Singleton } from "@cmmv/core";

export class Scope extends Singleton {
    private data: Map<string, any> = new Map();

    public static set(name: string, data: any): boolean {
        const scope = Scope.getInstance();

        if (!scope.data.has(name)) {
            scope.data.set(name, data);
            return true;
        }

        return false;
    }

    public static has(name: string): boolean {
        const scope = Scope.getInstance();
        return scope.data.has(name);
    }

    public static get<T = any>(name: string): T | null {
        const scope = Scope.getInstance();
        return scope.data.has(name) ? (scope.data.get(name) as T) : null;
    }

    public static clear(name: string): void {
        const scope = Scope.getInstance();
        scope.data.delete(name);
    }

    public static addToArray<T = any>(name: string, value: T): boolean {
        const scope = Scope.getInstance();
        const array = scope.data.get(name) || [];
        
        if (Array.isArray(array)) {
            array.push(value);
            scope.data.set(name, array);
            return true;
        }

        return false;
    }

    public static removeFromArray<T = any>(name: string, value: T): boolean {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);
        
        if (Array.isArray(array)) {
            const index = array.indexOf(value);

            if (index > -1) {
                array.splice(index, 1);
                scope.data.set(name, array);
                return true;
            }
        }

        return false;
    }

    public static getArray<T = any>(name: string): T[] | null {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);

        if (Array.isArray(array)) 
            return array as T[];
        
        return null;
    }

    public static getArrayFromIndex<T = any>(name: string, index: number): T | null {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);

        if (Array.isArray(array) && array.length >= index) 
            return array[index] as T;
        
        return null;
    }
}
```

## Advantages

Using singletons provides several advantages over traditional dependency injection systems, especially for larger applications with complex services and multiple modules:

* **Performance Optimization:** Since singletons are instantiated only once, they reduce the overhead associated with creating multiple instances of services. This is particularly beneficial when dealing with high-frequency services like database connections or cache managers.

* **Simplified Architecture:** Instead of managing complex dependency trees and imports in each module, singleton registries allow direct access to services, simplifying module design and reducing the need for boilerplate code.

* **Avoid Circular Dependencies:** Circular dependencies occur when two services depend on each other, causing dependency injection systems to fail or become overly complex. Singletons avoid this issue by being self-contained and globally accessible.

* **Ease of Access:** Services like configuration, caching, and logging can be made globally accessible without requiring them to be explicitly injected into every module or service that needs them.

* **Scalability:** As the application grows, using singletons simplifies service management. Unlike traditional dependency injection systems that require the careful management of imports and global providers, singletons offer a straightforward solution to sharing services across different modules.


## Best Practices

* **Use Singletons for Global Services:** Use singleton registries for services that need to be shared across multiple modules, such as configuration, logging, caching, and database connections.
* **Minimize Overuse:** While singletons simplify service management, overusing them can make testing and debugging more difficult. Use them judiciously for truly global services.
* **Avoid State Mutability:** If possible, keep singleton services stateless or ensure their state is managed carefully to avoid unintended side effects.
* **Testing:** When testing singleton services, remember that they maintain their state throughout the application lifecycle. Reset or mock singleton instances as needed during tests.