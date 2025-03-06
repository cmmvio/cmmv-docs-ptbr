# Vault

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/vault](https://github.com/cmmvio/cmmv/tree/main/packages/vault)

O módulo **@cmmv/vault** fornece um sistema seguro de armazenamento de dados projetado para gerenciar informações sensíveis usando **criptografia de curva elíptica (ECC)** e **AES-256-GCM**. O módulo permite **geração de chaves, inserção segura de dados, recuperação e exclusão**, garantindo proteção criptográfica forte para os dados armazenados.

## Recursos
<br/>

- **Criptografia de Curva Elíptica (secp256k1)** - Gerenciamento de pares de chaves públicas e privadas.
- **Criptografia AES-256-GCM** - Proteção segura dos dados com autenticação.
- **Armazenamento Seguro Baseado em Chaves** - Armazenamento de chave-valor com UUID v5 e namespaces.
- **Integração com o Framework CMMV** - Suporte nativo para repositório e autenticação.
- **Operações CRUD Completas** - Inserção, recuperação e exclusão de dados seguros.

## Instalação

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso Importante</p>
    <p>
        O módulo <strong>@cmmv/vault</strong> está disponível apenas a partir da versão <strong>0.8.30</strong>, pois depende de recursos de carregamento dinâmico de entidades introduzidos nesta versão.
        Certifique-se de que seu projeto está rodando na <strong>versão 0.8.30 ou superior</strong> para evitar problemas de compatibilidade.
    </p>
</div>

Para instalar o módulo, execute:

```sh
npm install @cmmv/vault
```

## Configuração

O módulo requer um arquivo de configuração (`.cmmv.config.cjs`) para definir o namespace e as chaves criptográficas:

```javascript
module.exports = {
    vault: {
        namespace: "seu-namespace-unico",
        publicKey: "sua-chave-publica",
        privateKey: "sua-chave-privada",
    }
};
```

| **Caminho**          | **Descrição**                           | **Valor Padrão / Exemplo** |
|----------------------|----------------------------------------|--------------------------|
| `vault.namespace`   | Identificador único para hash de chaves | `"seu-namespace-unico"` |
| `vault.publicKey`   | Chave pública para criptografia         | `"sua-chave-publica"` |
| `vault.privateKey`  | Chave privada para descriptografia      | `"sua-chave-privada"` |

## Endpoints da API

### **1. Inserir Dados Seguros**

Insere dados criptografados no cofre.

**Endpoint:** `POST /vault`

**Corpo da Requisição:**
```json
{
  "key": "minha-chave-secreta",
  "payload": "Dados sensíveis a serem criptografados"
}
```

**Resposta:**
```json
{
  "success": true,
  "id": "id-da-entrada-gerada"
}
```

### **2. Recuperar Dados Seguros**

Recupera e descriptografa os dados usando a chave privada armazenada.

**Endpoint:** `GET /vault/:key`

**Resposta:**
```json
{
  "payload": "Dados sensíveis descriptografados"
}
```

### **3. Excluir Dados Seguros**

Remove uma entrada do cofre.

**Endpoint:** `DELETE /vault/:key`

**Resposta:**
```json
{
  "success": true
}
```

## Exemplo de Uso

Você pode usar o Vault Service para criptografar e armazenar dados com segurança.

```typescript
import { VaultService } from '@cmmv/vault';

const vaultService = new VaultService();

// Inserir dados
await vaultService.insert('userToken', 'meu-valor-secreto');

// Recuperar dados
const secretValue = await vaultService.get('userToken');
console.log(secretValue); // Dados descriptografados

// Remover dados
await vaultService.remove('userToken');
```

## Segurança Criptográfica

O módulo Vault implementa **criptografia de curva elíptica secp256k1** combinada com **AES-256-GCM** para segurança robusta. O fluxo de criptografia funciona da seguinte maneira:

1. Uma **chave pública** é usada para criptografar os dados.
2. Os dados são armazenados de forma segura junto com um **IV (vetor de inicialização)** e uma **tag de autenticação**.
3. Ao recuperar os dados, a **chave privada armazenada** é usada para descriptografar as informações de forma segura.

## Geração de Chaves

Você pode gerar novas chaves usando a seguinte função:

```typescript
const vaultService = new VaultService();
const keys = await vaultService.createKeys();

console.log(keys);
/* {
  namespace: 'namespace-gerado',
  privateKey: 'chave-privada-gerada',
  publicKey: 'chave-publica-gerada'
} */
```

## Tratamento de Erros

| **Erro**                    | **Motivo**                            | **Código HTTP** |
|----------------------------|----------------------------------|--------------|
| `Formato de chave pública inválido` | Chave pública ausente ou inválida | 403 Forbidden |
| `Chave 'xyz' não existe`      | A chave solicitada não existe    | 403 Forbidden |

## Resumo

O módulo **@cmmv/vault** fornece uma maneira segura de armazenar e gerenciar dados criptografados, garantindo que informações sensíveis permaneçam protegidas. Com criptografia baseada em **ECC e AES-256-GCM**, ele oferece um alto nível de segurança e integração perfeita com o **CMMV**.

Use este módulo para armazenar com segurança **chaves de API, tokens de usuários, credenciais sensíveis** e qualquer outro dado que exija proteção criptográfica.
