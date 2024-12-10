# Request

The ``HttpService`` class in the ``@cmmv/http`` package provides a convenient way to make HTTP requests to external APIs using the ``Axios`` [NPM](https://www.npmjs.com/package/axios) library. It extends the ``AbstractService`` and is registered as a service with the name ``'http'``. The service wraps common HTTP request methods such as ``GET``, ``POST``, ``DELETE``, ``PUT``, ``PATCH``, and ``HEAD`` while using Axios for handling requests and responses.

To use the ``HttpService`` in your project, make sure to import it from the ``@cmmv/http`` package:

```typescript
import { HttpService } from '@cmmv/http';
```

The ``HttpService`` provides several HTTP methods for interacting with external APIs. These methods support custom configurations through ``AxiosRequestConfig`` and return a ``Promise`` that resolves with an ``AxiosResponse``.

## Methods

``request<T>(config: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Performs a generic HTTP request based on the given Axios configuration.

**Parameters:**

* **``config:``** An AxiosRequestConfig object that defines the HTTP method, headers, and other request configurations.

* **Returns:** A Promise that resolves with an AxiosResponse.

```typescript
const response = await this.httpService.request({
    method: 'GET',
    url: 'https://api.example.com/data',
});
```

## Get

``get<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``GET`` request to the specified URL.

**Parameters:**

* **``url:``** The URL to send the GET request to.

* **``config:``** An AxiosRequestConfig object that defines the HTTP method, headers, and other request configurations.

* **Returns:** A Promise that resolves with an AxiosResponse.

```typescript
const response = await this.httpService.get('https://api.example.com/data');
```

## Delete

``delete<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``DELETE`` request to the specified URL.

**Parameters:**

* **``url:``** The URL to send the DELETE request to.

* **``config:``** Optional Axios configuration options.

* **Returns:** A Promise that resolves with an AxiosResponse.

```typescript
const response = await httpService.delete('https://api.example.com/data/1');
```

## Delete

``delete<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``DELETE`` request to the specified URL.

**Parameters:**

* **``url:``** The URL to send the DELETE request to.

* **``config:``** Optional Axios configuration options.

* **Returns:** A Promise that resolves with an AxiosResponse.

```typescript
const response = await httpService.delete('https://api.example.com/data/1');
```

## Head

``head<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``HEAD`` request to the specified URL to fetch only headers without the response body.

**Parameters:**

* **``url:``** The URL to send the ``HEAD`` request to.

* **``config:``** Optional Axios configuration options.

* **Returns:** A ``Promise`` that resolves with an ``AxiosResponse``.

```typescript
const response = await this.httpService.head('https://api.example.com/data');
```

## Post

``post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``POST`` request to the specified URL with the given data.

**Parameters:**

* **``url:``** The URL to send the POST request to.

* **``data:``** The data to be sent in the request body.

* **``config:``** Optional Axios configuration options.

* **Returns:** A ``Promise`` that resolves with an ``AxiosResponse``.

```typescript
const response = await httpService.post('https://api.example.com/data', { 
    key: 'value' 
});
```

## Put

``put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``PUT`` request to the specified URL with the given data.

**Parameters:**

* **``url:``** The URL to send the PUT request to.

* **``data:``** The data to be sent in the request body.

* **``config:``** Optional Axios configuration options.

* **Returns:** A ``Promise`` that resolves with an ``AxiosResponse``.

```typescript
const response = await httpService.put('https://api.example.com/data/1', { 
    key: 'newValue' 
});
```

## Patch

``patch<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Description:** Sends an HTTP ``PATCH`` request to the specified URL with the given data.

**Parameters:**

* **``url:``** The URL to send the PATCH request to.

* **``data:``** The data to be sent in the request body.

* **``config:``** Optional Axios configuration options.

* **Returns:** A ``Promise`` that resolves with an ``AxiosResponse``.

```typescript
const response = await httpService.patch('https://api.example.com/data/1', { 
    key: 'patchedValue' 
});
```

# Usage 

```typescript
import { Service } from "@cmmv/core";
import { HttpService } from "@cmmv/http";
import { Cron } from "@cmmv/scheduling";

@Service()
export class CryptoPriceService {
    private readonly apiUrl = "https://api.coingecko.com/api/v3/coins/markets";

    constructor(
        private readonly httpService: HttpService
    ){}

    @Cron("*/1 * * * *")
    async fetchTopCryptos(){
        try {
            const response = await this.httpService.get(this.apiUrl, {
                params: {
                    vs_currency: 'usd',
                    order: 'market_cap_desc', 
                    page: 1
                }
            });

            if (response.status === 200) {
                const cryptos = response.data;
                cryptos.forEach((crypto: any) => {
                    console.log(`
                        Criptomoeda: ${crypto.name}, 
                        Preço: $${crypto.current_price}
                    `);
                });
            } else {
                console.error(`Erro ao obter dados: ${response.statusText}`);
            }
        } catch (error) {
            console.error('Erro ao realizar a requisição:', error.message);
        }
    }
}
```

The ``HttpService`` is a powerful and flexible tool for making HTTP requests within the ``@cmmv/http`` framework. It leverages the popular ``Axios`` library to handle HTTP operations, providing an easy way to interact with external APIs from your CMMV-based application. Each method corresponds to a standard HTTP request type, and additional configurations can be applied using ``AxiosRequestConfig`` to customize behavior like headers, timeouts, and more.