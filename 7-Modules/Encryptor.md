# Encryptor

The ``@cmmv/encryptor`` module provides a set of cryptographic utilities for CMMV-based applications. It offers robust methods for encryption, decryption, digital signature generation, and key management using elliptic curve cryptography (``ECC``) with ``secp256k1``, ``AES-256-GCM`` for encryption, and BIP32/BIP39 for wallet management. This module is designed to secure data transmission and storage, ensuring confidentiality, integrity, and authenticity of sensitive information.

## Installation

To install the ``@cmmv/encryptor`` module, use the following command:

```bash
$ pnpm add @cmmv/encryptor bip32 bip39 bs58 elliptic tiny-secp256k1
```

### Features

<br/>

* **ECC-based Encryption:** Supports elliptic curve cryptography (secp256k1) for secure key management and encryption.
* **AES-256-GCM Encryption:** Provides encryption using AES-256-GCM to ensure data confidentiality and authenticity.
* **Digital Signatures:** Sign and verify messages and objects using private keys, ensuring authenticity.
* **BIP32/BIP39 Wallet Management:** Generate mnemonic phrases, derive private/public keys, and manage keys according to BIP32 and BIP39 standards.

## Examples

### 1. Key Generation

Generate a new private-public key pair using Wallet:

```typescript
import { Wallet } from '@cmmv/encryptor';

const mnemonic = Wallet.generateMnenomic();
const privateKey = Wallet.toPrivate(mnemonic);
const publicKey = Wallet.privateToPublic(privateKey);

console.log(`Mnemonic: ${mnemonic}`);
console.log(`Private Key: ${privateKey}`);
console.log(`Public Key: ${publicKey}`);
```

### 2. Object Signing

Sign an object with a private key and verify its signature:

```typescript
import { Signer } from '@cmmv/encryptor';

const privateKey = 'your-private-key-hex';
const transaction = { amount: 100, recipient: 'address' };
const { objectHash, signature } = Signer.signObject(privateKey, transaction);

console.log(`Hash: ${objectHash}`);
console.log(`Signature: ${signature}`);

const publicKey = Wallet.privateToPublic(privateKey);
const isValid = Signer.verifySignature(objectHash, signature, publicKey);

console.log(`Is valid: ${isValid}`);
```

### 3. Data Encryption

Encrypt and decrypt a string payload using ECC and ``AES-256-GCM``:

```typescript
import { Encryptor } from '@cmmv/encryptor';

// Public and private keys
const recipientPublicKey = 'your-recipient-public-key-hex';
const recipientPrivateKey = 'your-recipient-private-key-hex';

// Encrypt a payload
const payload = "Confidential message";
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, payload);

console.log('Encrypted Data:', encryptedData);

// Decrypt the payload
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    {
        encrypted: encryptedData.payload,
        iv: encryptedData.iv,
        authTag: encryptedData.authTag,
    },
    encryptedData.ephemeralPublicKey
);

console.log(`Decrypted Payload: ${decryptedPayload}`);
```

## Wallets

The ``Wallet`` class provides methods for handling cryptographic operations related to wallet generation, key derivation, and address generation using BIP32 and BIP39 standards. It supports mnemonic generation, private and public key derivation, and conversion to Wallet Import Format (WIF). This class is designed to work with various blockchain networks such as Bitcoin and Ethereum.

To use the Wallet class, make sure to import it:

```typescript
import { Wallet } from '@cmmv/encryptor';
```

### ``getEntropyForWordCount(size: number)``

Returns the entropy size in bits based on the word count of a mnemonic phrase.

```typescript
const entropy = Wallet.getEntropyForWordCount(12); // Returns 128 for 12 words
```

### ``generateMnenomic(size: number = 24, wordlists: string[] = bip39.wordlists.english)``

Generates a mnemonic phrase using a specified word count and wordlist.

```typescript
const mnemonic = Wallet.generateMnenomic(12);
```

### ``entropyToMnemonic(entropy: Buffer | string, wordlists: string[] = bip39.wordlists.english)``

Converts a given entropy value to a mnemonic phrase.

```typescript
const mnemonic = Wallet.entropyToMnemonic('000102030405060708090a0b0c0d0e0f');
```

### ``randomByteMnemonic(wordlists: string[] = bip39.wordlists.english)``

Generates a mnemonic phrase using 32 bytes of random entropy.

```typescript
const mnemonic = Wallet.randomByteMnemonic();
```

### ``getSeed(mnemonic: string)``

Converts a mnemonic phrase to a seed in hexadecimal format.

```typescript
const seed = Wallet.getSeed(mnemonic);
```

### ``getSeedBuffer(mnemonic: string)``

Converts a mnemonic phrase to a seed as a Buffer.

```typescript
const seedBuffer = Wallet.getSeedBuffer(mnemonic);
```

### ``toPrivate(mnemonic: string, passphrase: string = '')``

Derives the root private key from a mnemonic phrase and optional passphrase.

```typescript
const privateKey = Wallet.toPrivate(mnemonic, 'myPassphrase');
```

### ``createDerivationPath(bip: number = 44, coinType: number = 0, account: number = 0, change: number = 0, addressIndex: number = 0)``

Creates a BIP derivation path.

```typescript
const path = Wallet.createDerivationPath();
```

### ``toDerivatationPrivateKey(mnemonic: string, derivationPath: string = "m/44'/0'/0'/0/0", passphrase: string = '')``

Derives the private key using a mnemonic phrase and a specific derivation path.

```typescript
const privateKey = Wallet.toDerivatationPrivateKey(mnemonic, "m/44'/0'/0'/0/0");
```

### ``toRootKey(mnemonic: string, passphrase: string = '', network?: Network)``

Derives the root key in Base58 format.

```typescript
const rootKey = Wallet.toRootKey(mnemonic);
```

### ``toPublic(mnemonic: string, derivationPath: string = "m/44'/0'/0'/0/0", passphrase: string = '')``

Derives the public key from a mnemonic phrase and a derivation path.

```typescript
const publicKey = Wallet.toPublic(mnemonic);
```

### ``privateToPublic(privateKey: string | Uint8Array)``

Derives the public key from a private key.

```typescript
const publicKey = Wallet.privateToPublic(privateKey);
```

### ``bip32ToPublic(bip32Key: any)``

Gets the public key directly from a BIP32 key.

```typescript
const publicKey = Wallet.bip32ToPublic(bip32Key);
```

### ``privateKeyToWIF(privateKey: string, compressed: boolean = true)``

Converts a private key to Wallet Import Format (WIF).

```typescript
const wif = Wallet.privateKeyToWIF(privateKey);
```

### ``wifToPrivateKey(wif: string)``

Converts a WIF key to a private key.

```typescript
const { privateKey, compressed } = Wallet.wifToPrivateKey(wif);
```

### ``privateKeyToAddress(privateKey: string | Uint8Array | undefined)``

Generates a public address from a private key.

```typescript
const address = Wallet.privateKeyToAddress(privateKey);
```

### ``publicKeyToAddress(publicKey: string)``

Generates a public address from a public key.

```typescript
const address = Wallet.publicKeyToAddress(publicKey);
```

## Signer

The ``Signer`` class provides methods for cryptographic signing and verification of objects and strings using elliptic curve cryptography (``secp256k1``). This class is essential for handling digital signatures and verifying authenticity in blockchain and cryptocurrency applications. It uses ``tiny-secp256k1`` for cryptographic operations and supports hashing algorithms like ``sha256`` and ``sha3-256``.

To use the ``Signer`` class, make sure to import it:

```typescript
import { Signer } from '@cmmv/encryptor';
```

This class provides methods for signing, verifying, and recovering public keys based on signed messages and objects. Below is a detailed explanation of each method, how to use them, and example usage.

### ``signObject(privateKeyHex: string, object: any, algorithm: string = "sha3-256")``

Signs an object using the given private key and returns the object hash and signature.

```typescript
const signature = Signer.signObject(privateKeyHex, { key: "value" });
console.log(signature); // { objectHash: '...', signature: '...' }
```

### ``verifySignature(objectHash: string, signatureHex: string, publicKeyHex: string | Uint8Array | undefined)``

Verifies a signature against the hash of an object using the provided public key.

```typescript
const isValid = Signer.verifySignature(objectHash, signature, publicKeyHex);
console.log(isValid); // true or false
```

### ``recoverPublicKey(objectHash: string, signatureHex: string, recoveryId: 0 | 1 = 1)``

Recovers the public key from a given signature and object hash.

```typescript
const publicKey = Signer.recoverPublicKey(objectHash, signatureHex, 1);
console.log(publicKey); // Compressed public key in hex format
```

### ``signString(privateKeyHex: string, message: string, algorithm: string = "sha256")``

Signs a string by hashing it with the specified algorithm and returns the signature along with the hash.

```typescript
const signedMessage = Signer.signString(privateKeyHex, "message to sign");
console.log(signedMessage); // 'hash:signature'
```

### ``verifyHashSignature(signature: string, publicKeyHex: string)``

Verifies if the signature matches the given hash and public key.

```typescript
const isValid = Signer.verifyHashSignature(signedMessage, publicKeyHex);
console.log(isValid); // true or false
```

### ``recoverPublicKeyFromHash(signature: string, recoveryId: 0 | 1 = 1)``

Recovers the public key from a given signature and hash.

```typescript
const publicKey = Signer.recoverPublicKeyFromHash(signedMessage);
console.log(publicKey); // Compressed public key in hex format
```

### ``signedBy(signature: string, publicKeyHex: string)``

Verifies if the message was signed by the provided public key by checking both recovery IDs.

```typescript
const isSignedBy = Signer.signedBy(signedMessage, publicKeyHex);
console.log(isSignedBy); // true or false
```

The ``Signer`` class provides essential methods for digital signature operations, making it suitable for use in blockchain and cryptocurrency applications where verifying the authenticity of transactions and messages is critical.

## Encryptor

The ``Encryptor`` class provides methods for encrypting and decrypting data using elliptic curve cryptography (``secp256k1``) and AES-256-GCM for encryption. It is designed for secure transmission of data using Elliptic Curve Diffie-Hellman (ECDH) to derive shared keys for symmetric encryption. This class is essential for ensuring the confidentiality and authenticity of sensitive information exchanged between parties.

To use the ``Encryptor`` class, import it as follows:

```typescript
import { Encryptor } from '@cmmv/encryptor';
```

### ``encryptPayload(recipientPublicKeyHex: string, payload: string)``

Encrypts a payload using the recipient's public key. This method generates an ephemeral key pair and derives a shared secret using ECDH. The payload is encrypted using AES-256-GCM with the derived shared key, ensuring confidentiality and authenticity.

```typescript
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, "Hello, World!");
console.log(encryptedData);
// {
//     payload: '0x...',
//     iv: '0x...',
//     authTag: '0x...',
//     ephemeralPublicKey: '0x...'
// }
```

### ``decryptPayload(recipientPrivateKeyHex: string, encryptedData: { encrypted: string, iv: string, authTag: string }, ephemeralPublicKeyHex: string): string``

Decrypts an encrypted payload using the recipient's private key. This method derives the shared secret using the recipient's private key and the sender's ephemeral public key. The shared key is then used to decrypt the payload using AES-256-GCM.

```typescript
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    { encrypted: '0x...', iv: '0x...', authTag: '0x...' },
    ephemeralPublicKey
);
console.log(decryptedPayload); // "Hello, World!"
```

The ``Encryptor`` class provides an efficient and secure mechanism for encrypting and decrypting data using elliptic curve cryptography combined with AES-256-GCM. This ensures that both confidentiality and integrity of the data are maintained during transmission.