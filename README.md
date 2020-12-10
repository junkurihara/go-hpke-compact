[![CI status](https://github.com/jedisct1/go-hpke-compact/workflows/Go/badge.svg)](https://github.com/jedisct1/go-hpke-compact/actions)
[![Go Reference](https://pkg.go.dev/badge/github.com/jedisct1/go-hpke-compact.svg)](https://pkg.go.dev/github.com/jedisct1/go-hpke-compact)

# A compact HPKE implemention for Go

`hpkecompact` is a small implementation of the [Hybrid Public Key Encryption](https://www.ietf.org/archive/id/draft-irtf-cfrg-hpke-06.html) (HPKE) draft.

It fits in a single file and only uses the Go standard library and `x/crypto`.

Suites are currently limited to `X25519-HKDF-SHA256` / `HKDF-SHA-256` / `{AES-{128,256}-GCM, CHACHA20-POLY1305}`; these are very likely to be the most commonly deployed ones for a forseable future.

## Usage

### Suite instantiation

```go
suite, err := NewSuite(KemX25519HkdfSha256, KdfHkdfSha256, AeadAes128Gcm)
```

### Key pair creation

```go
serverKp, err := ctx.GenerateKeyPair()
```

### Client: creation and encapsulation of the shared secret

A _client_ initiates a connexion by sending an encrypted secret; a _server_ accepts an encrypted secret from a client, and decrypts it, so that both parties have a shared secret they can use for encryption.

```go
clientCtx, encryptedSharedSecret, err :=
    suite.NewClientContext(serverKp.PublicKey, []byte("application name"), nil)
```

* `encryptedSharedSecret` needs to be sent to the server.
* `clientCtx` can be used to encrypt/decrypt messages exchanged with the server.
* The last parameter is an optional pre-shared key (`Psk` type).

### Server: decapsulation of the shared secret

```go
serverCtx, err := suite.NewServerContext(encryptedSharedSecret,
    serverKp, []byte("application name"), nil)
```

* `serverCtx` can be used to encrypt/decrypt messages exchanged with the client
* The last parameter is an optional pre-shared key (`Psk` type).

### Encryption of a message for the server

A message can be encrypted by the client for the server:

```go
ciphertext, err := clientCtx.EncryptToServer([]byte("message"), nil)
```

Nonces are automatically incremented, so it is safe to call this function multiple times within the same context.

Second parameter is optional associated data.

HPKE only supports unidirectional communications from the client to the server.

### Decryption of a ciphertext received by the server

The server can decrypt a message received by the client:

```go
decrypted, err := serverCtx.DecryptFromClient(ciphertext, nil)
```

Second parameter is optional associated data.

### Exporter secret

The exporter secret is directly accessible from the `Context` structure:

```go
exporter := serverCtx.ExporterSecret
```

## Authenticated modes

Authenticated modes, with or without a PSK are supported.

Just replace `NewClientContext()` with `NewAuthenticatedClientContext()` and `NewServerContext()` with `NewAuthenticatedServerContext()` for authentication.

```go
clientKp, err := suite.GenerateKeyPair()
serverKp, err := suite.GenerateKeyPair()

clientCtx, encryptedSharedSecret, err := suite.NewAuthenticatedClientContext(
    clientKp, serverKp.PublicKey, []byte("app"), psk)

serverCtx, err := suite.NewAuthenticatedServerContext(
    clientKp.PublicKey, encryptedSharedSecret, serverKp, []byte("app"), psk)
```

## That's it!