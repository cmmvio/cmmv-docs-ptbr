# Encryptor

O módulo ``@cmmv/encryptor`` fornece um conjunto de utilitários criptográficos para aplicações baseadas no CMMV. Ele oferece métodos robustos para criptografia, descriptografia, geração de assinaturas digitais e gerenciamento de chaves usando criptografia de curvas elípticas (``ECC``) com ``secp256k1``, ``AES-256-GCM`` para criptografia, e BIP32/BIP39 para gerenciamento de wallets. Este módulo é projetado para garantir a segurança na transmissão e armazenamento de dados, preservando a confidencialidade, integridade e autenticidade das informações sensíveis.

## Instalação

Para instalar o módulo ``@cmmv/encryptor``, use o seguinte comando:

```bash
$ pnpm add @cmmv/encryptor bip32 bip39 bs58 elliptic tiny-secp256k1
```

### Recursos

<br/>

* **Criptografia baseada em ECC:** Suporte para criptografia de curvas elípticas (``secp256k1``) para gerenciamento seguro de chaves.
* **Criptografia AES-256-GCM:** Fornece criptografia usando ``AES-256-GCM`` para garantir confidencialidade e autenticidade dos dados.
* **Assinaturas Digitais:** Assine e verifique mensagens e objetos usando chaves privadas, garantindo autenticidade.
* **Gerenciamento de Wallets BIP32/BIP39:** Gere frases mnemônicas, derive chaves privadas/públicas e gerencie chaves de acordo com os padrões BIP32 e BIP39.

## Exemplos

### 1. Geração de Chaves

Gere um novo par de chaves privada-pública usando a classe Wallet:

```typescript
import { Wallet } from '@cmmv/encryptor';

const mnemonic = Wallet.generateMnenomic();
const privateKey = Wallet.toPrivate(mnemonic);
const publicKey = Wallet.privateToPublic(privateKey);

console.log(`Mnemonic: \${mnemonic}`);
console.log(`Private Key: \${privateKey}`);
console.log(`Public Key: \${publicKey}`);
```

### 2. Assinatura de Objetos

Assine um objeto com uma chave privada e verifique sua assinatura:

```typescript
import { Signer } from '@cmmv/encryptor';

const privateKey = 'sua-chave-privada-hex';
const transaction = { amount: 100, recipient: 'endereco' };
const { objectHash, signature } = Signer.signObject(privateKey, transaction);

console.log(`Hash: \${objectHash}`);
console.log(`Signature: \${signature}`);

const publicKey = Wallet.privateToPublic(privateKey);
const isValid = Signer.verifySignature(objectHash, signature, publicKey);

console.log(`É válido: \${isValid}`);
```

### 3. Criptografia de Dados

Criptografe e descriptografe uma carga útil de string usando ECC e ``AES-256-GCM``:

```typescript
import { Encryptor } from '@cmmv/encryptor';

// Chaves pública e privada
const recipientPublicKey = 'sua-chave-publica-do-receptor-hex';
const recipientPrivateKey = 'sua-chave-privada-do-receptor-hex';

// Criptografe uma carga útil
const payload = "Mensagem Confidencial";
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, payload);

console.log('Dados Criptografados:', encryptedData);

// Descriptografe a carga útil
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    {
        encrypted: encryptedData.payload,
        iv: encryptedData.iv,
        authTag: encryptedData.authTag,
    },
    encryptedData.ephemeralPublicKey
);

console.log(`Carga Útil Descriptografada: \${decryptedPayload}`);
```

## Wallets

A classe ``Wallet`` fornece métodos para operações criptográficas relacionadas à geração de wallets, derivação de chaves e geração de endereços usando os padrões BIP32 e BIP39. Ela suporta a geração de mnemônicos, derivação de chaves privadas e públicas, e conversão para o formato Wallet Import Format (WIF). Esta classe é projetada para trabalhar com diversas redes blockchain, como Bitcoin e Ethereum.

Para usar a classe Wallet, certifique-se de importá-la:

```typescript
import { Wallet } from '@cmmv/encryptor';
```

### Exemplos de Métodos da Wallet

#### ``generateMnenomic(size: number = 24, wordlists: string[] = bip39.wordlists.english)``
Gera uma frase mnemônica usando um número de palavras especificado.

```typescript
const mnemonic = Wallet.generateMnenomic(12);
```

#### ``toPrivate(mnemonic: string, passphrase: string = '')``
Deriva a chave privada raiz de uma frase mnemônica e uma senha opcional.

```typescript
const privateKey = Wallet.toPrivate(mnemonic, 'minhaSenha');
```

#### ``privateKeyToAddress(privateKey: string | Uint8Array | undefined)``
Gera um endereço público a partir de uma chave privada.

```typescript
const address = Wallet.privateKeyToAddress(privateKey);
```

## Signer

A classe ``Signer`` fornece métodos para assinaturas criptográficas e verificação de objetos e strings usando criptografia de curvas elípticas (``secp256k1``). Este recurso é essencial para lidar com assinaturas digitais e verificar autenticidade em aplicações blockchain e de criptomoedas.

#### ``signObject(privateKeyHex: string, object: any, algorithm: string = "sha3-256")``
Assina um objeto usando a chave privada fornecida.

```typescript
const signature = Signer.signObject(privateKeyHex, { key: "value" });
console.log(signature); // { objectHash: '...', signature: '...' }
```

#### ``verifySignature(objectHash: string, signatureHex: string, publicKeyHex: string)``
Verifica a assinatura de um objeto.

```typescript
const isValid = Signer.verifySignature(objectHash, signature, publicKeyHex);
console.log(isValid); // true ou false
```

## Encryptor

A classe ``Encryptor`` fornece métodos para criptografar e descriptografar dados usando ECC e AES-256-GCM, garantindo confidencialidade e integridade.

#### ``encryptPayload(recipientPublicKeyHex: string, payload: string)``
Criptografa uma carga útil usando a chave pública do receptor.

```typescript
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, "Olá, Mundo!");
console.log(encryptedData);
```

#### ``decryptPayload(recipientPrivateKeyHex: string, encryptedData: object, ephemeralPublicKeyHex: string): string``
Descriptografa uma carga útil criptografada usando a chave privada do receptor.

```typescript
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    { encrypted: '0x...', iv: '0x...', authTag: '0x...' },
    ephemeralPublicKey
);
console.log(decryptedPayload); // "Olá, Mundo!"
```

Este módulo oferece uma solução completa e eficiente para garantir a segurança de dados e operações em ambientes sensíveis, como blockchain e criptomoedas.
