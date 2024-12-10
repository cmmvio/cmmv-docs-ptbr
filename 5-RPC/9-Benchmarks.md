# RPC Benchmarks

This report demonstrates a simple benchmark test for making 1000 HTTP requests to a server, including the size of the request and response headers, and estimates for the average time it takes to process each request. The data payload being tested is a simple JSON object { "Hello": "World" }, and the following headers are used for the request and response.

Request and Response Headers:

Response Headers:

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
Date: Wed, 04 Sep 2024 12:41:13 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked
```

Request Headers:
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

For the benchmark test, we analyzed the total size and time for processing 1000 HTTP requests. Here is the detailed breakdown:

Header Sizes:
* **Response Headers Size:** 925 bytes
* **Request Headers Size:** 783 bytes
* **Data Size:** 20 bytes (JSON payload { "Hello": "World" })

Total Size per Request:
* **Total Size per Request:** 1,728 bytes

Total Size for 1000 Requests:
* **Total Size for 1000 Requests:** 1,728,000 bytes (1.73 MB)

Timing Analysis:
Assuming an average internet connection with:

* **DNS Lookup:** 50ms
* **Time to First Byte (TTFB):** 100ms
* **Ping Time:** 100ms
* **Content Download Time:** 50ms

**Total Time per Request:** 300ms
**Total Time for 1000 Requests:** 300 seconds (5 minutes)

# Protobuf + Websocket

For this benchmark, we analyzed the total size and time for 1000 WebSocket requests using Protobuf for binary communication. In this scenario, the Protobuf schema is loaded only once (26 KB), and the data sent over WebSocket is minimal due to the efficient binary format. We assume the same payload, but in a compact, serialized format using Protobuf.

Assumptions:
* **Initial Protobuf Load:** 26 KB (loaded once, cached)
* **Data Size (Binary Format):** Approximately 12 bytes (based on the compact binary representation of { "Hello": "World" })
* **WebSocket Headers:** WebSocket doesn't require large headers like HTTP, leading to much smaller overhead.
* **Persistent Connection:** The WebSocket connection stays open, eliminating the need for repeated DNS lookups, TTFB, etc., which are required for each HTTP request.

Total Size per Request:
* **Binary Data Size per Request:** 12 bytes
* **WebSocket Header Overhead:** 2-6 bytes per frame (depending on the frame structure and opcode)
* **Total Size per Request:** ~18 bytes

Total Size for 1000 Requests:
* **Protobuf Schema Size (only loaded once):** 26 KB (26,000 bytes)
* **Total Data Transferred:** 18 bytes * 1000 requests = 18,000 bytes
* **Total Size for 1000 Requests:** 26,000 bytes (schema) + 18,000 bytes (data) = 44,000 bytes (44 KB)

Timing Analysis:

In a WebSocket connection:

* **No DNS Lookup:** DNS lookup occurs only once during the initial connection.
* **No TTFB:** The connection is persistent, so there's no time-to-first-byte for each request.
* **Ping Time:** 100ms (assumed for network latency)
* **Content Download Time (Binary Data):** Much smaller than HTTP/JSON.
* **Total Time per Request:** ~100ms (ping + minimal processing) Total Time for 1000 Requests: 100 seconds (1 minute 40 seconds)

Conclusion:
* **Total Data Transferred:** Only 44 KB for 1000 requests compared to 1.73 MB in HTTP/JSON.
* **Total Time:** 100 seconds (1 minute 40 seconds) for 1000 WebSocket requests, compared to 5 minutes for HTTP requests.

The binary communication over WebSocket using Protobuf is significantly more efficient in terms of both data size and processing time. The persistent connection eliminates the overhead associated with repeated DNS lookups and TTFB, while the compact binary format reduces the payload size considerably.