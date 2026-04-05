---
node_id: "crypto-action"
title: "Crypto Action"
description: "Generate random tokens, hash and HMAC data, sign payloads with RSA, and encrypt/decrypt with AES or RSA — powered by Node.js native crypto"
category: "utilities"
subcategory: "security"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - crypto
  - security
  - hash
  - encryption
  - signing
  - hmac
  - aes
  - rsa
  - utility
  - action
related_nodes:
  - function
  - webhook-trigger
  - http-request
---

<!-- SECTION: header -->
# Crypto Action

> **Category:** Utilities | **Type:** Action Node

Perform cryptographic operations natively within your workflow — generate random tokens, hash or HMAC data, sign payloads with an RSA private key, or encrypt and decrypt values using AES-CBC or RSA.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Crypto Action** node wraps Node.js's built-in `crypto` module into a configurable workflow node. No external dependencies or API keys are required — all operations run locally within the Fusion runtime.

It supports 8 operations organized into four categories: token generation, hashing, message authentication (HMAC), digital signing, symmetric encryption/decryption (AES-CBC), and asymmetric encryption/decryption (RSA).

### Key Features

- **8 Operations:** Full cryptographic toolkit from random generation to RSA decryption
- **Native Performance:** Uses Node.js `crypto` — no network calls, no external services
- **Configurable Encodings:** Input and output encodings are configurable per operation
- **Custom Output Property:** Hash, HMAC, and Sign operations write their result to a named property you define
- **Symmetric + Asymmetric:** AES-128/192/256-CBC for symmetric; RSA public/private key for asymmetric
- **UUID Generation:** Shortcut for `crypto.randomUUID()` built into the Generate operation

### Use Cases

- Generate a secure random API token or session ID for a new user registration workflow
- Hash a password or sensitive field before storing it in a database
- Compute an HMAC signature to verify the integrity of a webhook payload
- Sign a JWT-like assertion with an RSA private key before calling an external API
- Encrypt sensitive configuration values before storing them in a workflow result
- Decrypt an encrypted value received from an external partner system

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Cryptographic operation to execute (see below) |
| `generate` | `object` | Conditional | Config for the `Generate` operation |
| `hash` | `object` | Conditional | Config for the `Hash` operation |
| `hmac` | `object` | Conditional | Config for the `Hmac` operation |
| `sign` | `object` | Conditional | Config for the `Sign` operation |
| `symmetricEncrypt` | `object` | Conditional | Config for the `Symmetric Encrypt` operation |
| `symmetricDecrypt` | `object` | Conditional | Config for the `Symmetric Decrypt` operation |
| `asymmetricEncrypt` | `object` | Conditional | Config for the `Asymmetric Encrypt` operation |
| `asymmetricDecrypt` | `object` | Conditional | Config for the `Asymmetric Decrypt` operation |

Each conditional object is only active when its corresponding `operation` is selected.

---

### Operation: Generate

Produces a cryptographically secure random value.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `type` | `enum` | ✅ Yes | — | Output format: `ASCII`, `BASE64`, `HEX`, or `UUID` |
| `length` | `number` | ❌ No | `16` | Number of random bytes to generate (ignored for `UUID`) |

**Output:** A plain string — the generated value.

> `UUID` uses `crypto.randomUUID()` and ignores `length`.

---

### Operation: Hash

Computes a one-way hash of a value.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `type` | `enum` | ✅ Yes | — | Hash algorithm: `MD5`, `SHA256`, `SHA3-256`, `SHA3-384`, `SHA3-512`, `SHA384`, `SHA512` |
| `value` | `string` | ✅ Yes | — | Input string to hash |
| `outputPropertyName` | `string` | ✅ Yes | `hash` | Property name in the output object |
| `encoding` | `enum` | ✅ Yes | — | Output encoding: `hex`, `base64`, `base64url`, `ascii`, `utf-8`, `utf-16le`, `ucs-2`, `latin1`, `binary` |

**Output:** `{ "<outputPropertyName>": "<digest>" }`

---

### Operation: Hmac

Computes a keyed Hash-based Message Authentication Code.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `type` | `enum` | ✅ Yes | — | HMAC algorithm: same options as `Hash` |
| `value` | `string` | ✅ Yes | — | Input string to authenticate |
| `secret` | `string` | ✅ Yes | — | Secret key for the HMAC |
| `outputPropertyName` | `string` | ✅ Yes | `hmac` | Property name in the output object |
| `encoding` | `enum` | ✅ Yes | — | Output encoding (same options as `Hash`) |

**Output:** `{ "<outputPropertyName>": "<hmac>" }`

---

### Operation: Sign

Digitally signs a value using an RSA private key.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | `string` | ✅ Yes | The data to sign |
| `algorithm` | `enum` | ✅ Yes | Signing algorithm: `RSA-SHA256`, `RSA-SHA384`, `RSA-SHA512` |
| `privateKey` | `string` | ✅ Yes | PEM-encoded RSA private key |
| `outputPropertyName` | `string` | ✅ Yes | Property name in the output object |
| `encoding` | `enum` | ✅ Yes | Signature encoding: `hex`, `base64`, `base64url`, etc. |

**Output:** `{ "<outputPropertyName>": "<signature>" }`

---

### Operation: Symmetric Encrypt

Encrypts a value using AES in CBC mode.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `algorithm` | `enum` | ✅ Yes | AES variant: `aes-128-cbc`, `aes-192-cbc`, `aes-256-cbc` |
| `key` | `string` | ✅ Yes | Base64-encoded AES key (16, 24, or 32 bytes depending on variant) |
| `initializationVector` | `string` | ✅ Yes | Base64-encoded IV (16 bytes) |
| `value` | `string` | ✅ Yes | Plaintext string to encrypt |
| `outputEncoding` | `enum` | ✅ Yes | Output encoding: `hex` or `base64` |

**Output:** The encrypted ciphertext as a plain string in the requested encoding.

---

### Operation: Symmetric Decrypt

Decrypts an AES-CBC ciphertext.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `algorithm` | `enum` | ✅ Yes | AES variant: `aes-128-cbc`, `aes-192-cbc`, `aes-256-cbc` |
| `key` | `string` | ✅ Yes | Base64-encoded AES key |
| `initializationVector` | `string` | ✅ Yes | Base64-encoded IV (must match the one used to encrypt) |
| `value` | `string` | ✅ Yes | Ciphertext to decrypt |
| `inputEncoding` | `enum` | ✅ Yes | Encoding of the ciphertext: `hex` or `base64` |
| `outputEncoding` | `enum` | ✅ Yes | Encoding of the decrypted output: `utf-8` or `ascii` |

**Output:** The decrypted plaintext as a plain string.

---

### Operation: Asymmetric Encrypt

Encrypts a value using an RSA public key.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | `string` | ✅ Yes | Plaintext string to encrypt |
| `publicKey` | `string` | ✅ Yes | PEM-encoded RSA public key |
| `outputEncoding` | `enum` | ✅ Yes | Output encoding: `hex` or `base64` |

**Output:** The RSA-encrypted value as a plain string.

---

### Operation: Asymmetric Decrypt

Decrypts an RSA-encrypted value using an RSA private key.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | `string` | ✅ Yes | Base64-encoded ciphertext to decrypt |
| `privateKey` | `string` | ✅ Yes | PEM-encoded RSA private key |
| `passphrase` | `string` | ❌ No | Passphrase for an encrypted private key |
| `outputEncoding` | `enum` | ✅ Yes | Decrypted output encoding: `utf-8` or `ascii` |

**Output:** The decrypted plaintext as a plain string.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow data — used to compose `value` or `secret` fields via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `string` or `object` | The cryptographic result (see per-operation output above) |
| `error` | `Error` | Emitted if the operation fails (e.g. invalid key format, unsupported algorithm) |

### Output Summary by Operation

| Operation | Output Type | Example |
|-----------|-------------|---------|
| `Generate` | `string` | `"a3f8c2d1..."` |
| `Hash` | `object` | `{ "hash": "e3b0c44298fc..." }` |
| `Hmac` | `object` | `{ "hmac": "sha256-hmac-value" }` |
| `Sign` | `object` | `{ "signature": "base64-signature" }` |
| `Symmetric Encrypt` | `string` | `"4f2a8b..."` (hex/base64) |
| `Symmetric Decrypt` | `string` | `"hello world"` |
| `Asymmetric Encrypt` | `string` | `"base64-encrypted-value"` |
| `Asymmetric Decrypt` | `string` | `"original plaintext"` |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Generate a Secure HEX Token

Generate a 32-byte random token to use as an API key or session ID.

**Configuration:**
```json
{
  "operation": "Generate",
  "generate": {
    "type": "HEX",
    "length": 32
  }
}
```

**Output:** `"a3f8c2d1e4b7090f2e1c3d5a6f8e0b1d4c7a2f9e3d1b0c5a8f7e6d2b4c1a3f9"`

---

### Example: Generate a UUID

**Configuration:**
```json
{
  "operation": "Generate",
  "generate": {
    "type": "UUID"
  }
}
```

**Output:** `"550e8400-e29b-41d4-a716-446655440000"`

---

### Example: SHA-256 Hash a Value

Hash a user's email for anonymous analytics.

**Configuration:**
```json
{
  "operation": "Hash",
  "hash": {
    "type": "SHA256",
    "value": "{{input.email}}",
    "outputPropertyName": "emailHash",
    "encoding": "hex"
  }
}
```

**Output:**
```json
{ "emailHash": "b94d27b9934d3e08a52e52d7da7dabfac484efe04294e576b4b9bdfaf9cca891" }
```

---

### Example: HMAC Webhook Signature Verification

Compute the expected HMAC and compare it against an incoming webhook's `X-Signature` header.

**Configuration:**
```json
{
  "operation": "Hmac",
  "hmac": {
    "type": "SHA256",
    "value": "{{input.rawBody}}",
    "secret": "my-webhook-secret",
    "outputPropertyName": "expectedSignature",
    "encoding": "hex"
  }
}
```

**Output:**
```json
{ "expectedSignature": "3d9f2ab1c8e4f7d0..." }
```

---

### Example: AES-256-CBC Encrypt a Value

Encrypt a sensitive config value before storing it.

**Configuration:**
```json
{
  "operation": "Symmetric Encrypt",
  "symmetricEncrypt": {
    "algorithm": "aes-256-cbc",
    "key": "BASE64_ENCODED_32_BYTE_KEY==",
    "initializationVector": "BASE64_ENCODED_16_BYTE_IV==",
    "value": "super-secret-api-key",
    "outputEncoding": "base64"
  }
}
```

**Output:** `"U2FsdGVkX1+abc...=="` (base64-encoded ciphertext)

---

### Example: AES-256-CBC Decrypt

Decrypt the value stored in the previous step.

**Configuration:**
```json
{
  "operation": "Symmetric Decrypt",
  "symmetricDecrypt": {
    "algorithm": "aes-256-cbc",
    "key": "BASE64_ENCODED_32_BYTE_KEY==",
    "initializationVector": "BASE64_ENCODED_16_BYTE_IV==",
    "value": "{{input.ciphertext}}",
    "inputEncoding": "base64",
    "outputEncoding": "utf-8"
  }
}
```

**Output:** `"super-secret-api-key"`

---

### Example: RSA Sign a Payload

Sign a payload with an RSA-SHA256 private key for API authentication.

**Configuration:**
```json
{
  "operation": "Sign",
  "sign": {
    "value": "{{input.requestBody}}",
    "algorithm": "RSA-SHA256",
    "privateKey": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----",
    "outputPropertyName": "signature",
    "encoding": "base64"
  }
}
```

**Output:**
```json
{ "signature": "base64EncodedRSASignature==" }
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Webhook Payload Verification → Process → Store

Verify an incoming webhook's HMAC signature, then process and store the event if valid.

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "compute-hmac",
      "type": "crypto-action",
      "config": {
        "operation": "Hmac",
        "hmac": {
          "type": "SHA256",
          "value": "{{input.rawBody}}",
          "secret": "webhook-secret-key",
          "outputPropertyName": "expectedSignature",
          "encoding": "hex"
        }
      }
    },
    {
      "id": "verify-signature",
      "type": "function",
      "config": {
        "code": "if (input.hmac.expectedSignature !== input.headers['x-signature']) throw new Error('Invalid signature'); return input;"
      }
    },
    {
      "id": "store-event",
      "type": "mongodb-action",
      "config": {
        "operation": "Insert One",
        "collection": "events",
        "document": "{{input.body}}"
      }
    }
  ]
}
```

### Common Patterns

- **Token Generation:** Cron → Crypto `Generate` (HEX 32B) → store as new API key
- **Password Hashing:** User registration webhook → Crypto `Hash` (SHA256) → insert into DB
- **Signed Assertions:** HTTP trigger → Crypto `Sign` (RSA-SHA256) → call downstream API with signature header
- **Encrypt Before Store:** Sensitive data → Crypto `Symmetric Encrypt` → MongoDB insert
- **Decrypt on Read:** MongoDB find → Crypto `Symmetric Decrypt` → return plaintext to caller

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unsupported operation: <name>`

**Cause:** An invalid value was passed to `operation`.

**Solution:** Ensure `operation` is one of the exact enum values: `Generate`, `Hash`, `Hmac`, `Sign`, `Symmetric Encrypt`, `Symmetric Decrypt`, `Asymmetric Encrypt`, `Asymmetric Decrypt`.

#### `Invalid key length` (Symmetric Encrypt/Decrypt)

**Cause:** The AES key length does not match the algorithm — `aes-128-cbc` requires 16 bytes, `aes-192-cbc` 24 bytes, `aes-256-cbc` 32 bytes.

**Solution:** Ensure the base64-decoded key is the correct byte length for the chosen algorithm.

#### `Invalid IV length`

**Cause:** The initialization vector is not exactly 16 bytes.

**Solution:** Generate a 16-byte random IV and base64-encode it: `crypto.randomBytes(16).toString('base64')`.

#### `error:0906D06C:PEM routines:PEM_read_bio` (Sign/Asymmetric)

**Cause:** The PEM-encoded key is malformed, missing newlines, or has extra whitespace.

**Solution:** Paste the full PEM key including `-----BEGIN...-----` and `-----END...-----` headers. Preserve line breaks (`\n`) in the key string.

#### `Decryption failed` / garbled output (Symmetric Decrypt)

**Cause:** The key or IV does not match the one used during encryption, or `inputEncoding` is wrong.

**Solution:** Verify that `algorithm`, `key`, `initializationVector`, and `inputEncoding` exactly match the values used during encryption.

### Key Size Reference

| Algorithm | Key Size | Key (Base64-decoded bytes) | IV Size |
|-----------|----------|---------------------------|---------|
| `aes-128-cbc` | 128-bit | 16 bytes | 16 bytes |
| `aes-192-cbc` | 192-bit | 24 bytes | 16 bytes |
| `aes-256-cbc` | 256-bit | 32 bytes | 16 bytes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) - Custom JavaScript for advanced data transformations
- [Webhook Trigger](./whatsapp-webhook.md) - Receive and verify signed incoming webhooks
- [HTTP Request](./http-request.md) - Call external APIs that require signed or encrypted payloads

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
