# Benchmarks RPC

Este relatório demonstra um teste de benchmark simples para realizar 1000 requisições HTTP a um servidor, incluindo o tamanho dos cabeçalhos de requisição e resposta, e estimativas para o tempo médio necessário para processar cada requisição. O payload de dados testado é um objeto JSON simples { "Hello": "World" }, e os seguintes cabeçalhos são usados para a requisição e resposta.

### Cabeçalhos de Requisição e Resposta:

**Cabeçalhos de Resposta:**

```yaml
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Origin-Agent-Cluster: ?1
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=15552000; includeSubDomains
X-Content-Type-Options: nosniff
X-DNS-Prefetch-Control: off
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-XSS-Protection: 0
Content-Security-Policy: default-src 'self' 'nonce-f0954b9e'; script-src 'self' 'unsafe-eval' 'unsafe-inline' 'nonce-f0954b9e'; style-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com 'nonce-f0954b9e'; font-src 'self' https://cdnjs.cloudflare.com 'nonce-f0954b9e'
Content-Type: text/html; charset=utf-8
ETag: W/"5d67-lioHfOkcIYpcQS7nNQKGH5y6XRk"
Vary: Accept-Encoding
Content-Encoding: gzip
Date: Qua, 04 Set 2024 12:41:13 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked
```

**Cabeçalhos de Requisição:**

```yaml
GET /docs/3-RPC%2F0-Overview HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7,es;q=0.6
Cache-Control: max-age=0
Connection: keep-alive
Host: localhost:3000
If-None-Match: W/"5cee-HeytSiLTPcs5WSj3kRViDNpXpt0"
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari
```

Para o teste de benchmark, analisamos o tamanho total e o tempo para processar 1000 requisições HTTP. Aqui está a análise detalhada:

### Tamanhos dos Cabeçalhos:
* **Tamanho dos Cabeçalhos de Resposta:** 925 bytes
* **Tamanho dos Cabeçalhos de Requisição:** 783 bytes
* **Tamanho dos Dados:** 20 bytes (payload JSON { "Hello": "World" })

### Tamanho Total por Requisição:
* **Tamanho Total por Requisição:** 1.728 bytes

### Tamanho Total para 1000 Requisições:
* **Tamanho Total para 1000 Requisições:** 1.728.000 bytes (1,73 MB)

### Análise de Tempo:
Assumindo uma conexão média de internet com:

* **Consulta DNS:** 50ms
* **Tempo até o Primeiro Byte (TTFB):** 100ms
* **Tempo de Ping:** 100ms
* **Tempo de Download do Conteúdo:** 50ms

**Tempo Total por Requisição:** 300ms
**Tempo Total para 1000 Requisições:** 300 segundos (5 minutos)

# Protobuf + WebSocket

Para este benchmark, analisamos o tamanho total e o tempo para 1000 requisições WebSocket usando Protobuf para comunicação binária. Neste cenário, o esquema Protobuf é carregado apenas uma vez (26 KB), e os dados enviados pelo WebSocket são mínimos devido ao formato binário eficiente. Assumimos o mesmo payload, mas em um formato compacto e serializado usando Protobuf.

### Suposições:
* **Carregamento Inicial do Protobuf:** 26 KB (carregado uma vez, em cache)
* **Tamanho dos Dados (Formato Binário):** Aproximadamente 12 bytes (baseado na representação binária compacta de { "Hello": "World" })
* **Cabeçalhos WebSocket:** O WebSocket não exige cabeçalhos grandes como o HTTP, resultando em uma sobrecarga muito menor.
* **Conexão Persistente:** A conexão WebSocket permanece aberta, eliminando a necessidade de consultas DNS repetidas, TTFB, etc., que são necessárias para cada requisição HTTP.

### Tamanho Total por Requisição:
* **Tamanho dos Dados Binários por Requisição:** 12 bytes
* **Sobrecarga dos Cabeçalhos WebSocket:** 2-6 bytes por quadro (dependendo da estrutura do quadro e do opcode)
* **Tamanho Total por Requisição:** ~18 bytes

### Tamanho Total para 1000 Requisições:
* **Tamanho do Esquema Protobuf (carregado apenas uma vez):** 26 KB (26.000 bytes)
* **Total de Dados Transferidos:** 18 bytes * 1000 requisições = 18.000 bytes
* **Tamanho Total para 1000 Requisições:** 26.000 bytes (esquema) + 18.000 bytes (dados) = 44.000 bytes (44 KB)

### Análise de Tempo:

Em uma conexão WebSocket:

* **Sem Consulta DNS:** A consulta DNS ocorre apenas uma vez durante a conexão inicial.
* **Sem TTFB:** A conexão é persistente, então não há tempo até o primeiro byte para cada requisição.
* **Tempo de Ping:** 100ms (assumido para latência de rede)
* **Tempo de Download do Conteúdo (Dados Binários):** Muito menor que HTTP/JSON.
* **Tempo Total por Requisição:** ~100ms (ping + processamento mínimo)
* **Tempo Total para 1000 Requisições:** 100 segundos (1 minuto e 40 segundos)

### Conclusão:
* **Total de Dados Transferidos:** Apenas 44 KB para 1000 requisições em comparação com 1,73 MB em HTTP/JSON.
* **Tempo Total:** 100 segundos (1 minuto e 40 segundos) para 1000 requisições WebSocket, em comparação com 5 minutos para requisições HTTP.

A comunicação binária por WebSocket usando Protobuf é significativamente mais eficiente em termos de tamanho de dados e tempo de processamento. A conexão persistente elimina a sobrecarga associada a consultas DNS repetidas e TTFB, enquanto o formato binário compacto reduz consideravelmente o tamanho do payload.
