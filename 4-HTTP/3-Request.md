# Requisição

A classe ``HttpService`` no pacote ``@cmmv/http`` oferece uma maneira conveniente de fazer requisições HTTP para APIs externas usando a biblioteca ``Axios`` [NPM](https://www.npmjs.com/package/axios). Ela estende a ``AbstractService`` e é registrada como um serviço com o nome ``'http'``. O serviço encapsula métodos comuns de requisição HTTP, como ``GET``, ``POST``, ``DELETE``, ``PUT``, ``PATCH`` e ``HEAD``, enquanto utiliza o Axios para lidar com requisições e respostas.

Para usar o ``HttpService`` no seu projeto, certifique-se de importá-lo do pacote ``@cmmv/http``:

```typescript
import { HttpService } from '@cmmv/http';
```

O ``HttpService`` fornece vários métodos HTTP para interagir com APIs externas. Esses métodos suportam configurações personalizadas por meio do ``AxiosRequestConfig`` e retornam uma ``Promise`` que resolve com um ``AxiosResponse``.

## Métodos

### ``request<T>(config: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Realiza uma requisição HTTP genérica com base na configuração fornecida do Axios.

**Parâmetros:**

* **``config:``** Um objeto ``AxiosRequestConfig`` que define o método HTTP, cabeçalhos e outras configurações da requisição.

* **Retorno:** Uma Promise que resolve com um ``AxiosResponse``.

```typescript
const response = await this.httpService.request({
    method: 'GET',
    url: 'https://api.example.com/data',
});
```

## Get

### ``get<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``GET`` para a URL especificada.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição GET.

* **``config:``** Um objeto ``AxiosRequestConfig`` que define o método HTTP, cabeçalhos e outras configurações da requisição.

* **Retorno:** Uma Promise que resolve com um ``AxiosResponse``.

```typescript
const response = await this.httpService.get('https://api.example.com/data');
```

## Delete

### ``delete<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``DELETE`` para a URL especificada.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição DELETE.

* **``config:``** Opções de configuração opcionais do Axios.

* **Retorno:** Uma Promise que resolve com um ``AxiosResponse``.

```typescript
const response = await httpService.delete('https://api.example.com/data/1');
```

## Head

### ``head<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``HEAD`` para a URL especificada para buscar apenas cabeçalhos sem o corpo da resposta.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição ``HEAD``.

* **``config:``** Opções de configuração opcionais do Axios.

* **Retorno:** Uma ``Promise`` que resolve com um ``AxiosResponse``.

```typescript
const response = await this.httpService.head('https://api.example.com/data');
```

## Post

### ``post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``POST`` para a URL especificada com os dados fornecidos.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição POST.

* **``data:``** Os dados a serem enviados no corpo da requisição.

* **``config:``** Opções de configuração opcionais do Axios.

* **Retorno:** Uma ``Promise`` que resolve com um ``AxiosResponse``.

```typescript
const response = await httpService.post('https://api.example.com/data', {
    chave: 'valor'
});
```

## Put

### ``put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``PUT`` para a URL especificada com os dados fornecidos.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição PUT.

* **``data:``** Os dados a serem enviados no corpo da requisição.

* **``config:``** Opções de configuração opcionais do Axios.

* **Retorno:** Uma ``Promise`` que resolve com um ``AxiosResponse``.

```typescript
const response = await httpService.put('https://api.example.com/data/1', {
    chave: 'novoValor'
});
```

## Patch

### ``patch<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>``

* **Descrição:** Envia uma requisição HTTP ``PATCH`` para a URL especificada com os dados fornecidos.

**Parâmetros:**

* **``url:``** A URL para a qual enviar a requisição PATCH.

* **``data:``** Os dados a serem enviados no corpo da requisição.

* **``config:``** Opções de configuração opcionais do Axios.

* **Retorno:** Uma ``Promise`` que resolve com um ``AxiosResponse``.

```typescript
const response = await httpService.patch('https://api.example.com/data/1', {
    chave: 'valorCorrigido'
});
```

# Uso

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

O ``HttpService`` é uma ferramenta poderosa e flexível para fazer requisições HTTP dentro do framework ``@cmmv/http``. Ele utiliza a popular biblioteca ``Axios`` para lidar com operações HTTP, fornecendo uma maneira fácil de interagir com APIs externas a partir da sua aplicação baseada em CMMV. Cada método corresponde a um tipo de requisição HTTP padrão, e configurações adicionais podem ser aplicadas usando ``AxiosRequestConfig`` para personalizar comportamentos como cabeçalhos, tempos limite e mais.
