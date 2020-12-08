# A small HPKE implemention for Go

HPKE-Compact is a small implementation of the [Hybrid Public Key Encryption](https://www.ietf.org/archive/id/draft-irtf-cfrg-hpke-06.html) draft.

It fits in a single file and doesn't require 3rd party dependencies.

Suites are currently limited to `X25519-HKDF-SHA256/HKDF-SHA-256/AES-GCM` because these are very likely to be the most commonly deployed ones in a forseable future.

## Usage

### Suite instantiation

```go
suite, err := NewSuite(KemX25519HkdfSha256, KdfHkdfSha256, AeadAes128Gcm)
```

### Server key pair creation

```go
serverPk, serverSk, err := ctx.GenerateKeyPair()
```

### Client: creation and encapsulation of the shared secret

```go
clientCtx, encryptedSharedSecret, err := suite.NewClientContext(serverPk, []byte("test"))
```

* `encryptedSharedSecret` needs to be sent to the server.
* `clientCtx` can be used to encrypt/decrypt messages exchanged with the server

### Server: decapsulation of the shared secret

```go
serverCtx, err := suite.NewServerContext(encryptedSharedSecret, serverPk, serverSk, []byte("test"))
```

* `serverCtx` can be used to encrypt/decrypt messages exchanged with the client

### Encryption of a message

```go
ciphertext, err := clientCtx.Encrypt(nil, []byte("message"))
```

Nonces are automatically incremented, so it is safe to call this function multiple times within the same context.

### Verification and decryption of a ciphertext

```go
decrypted, err := serverCtx.Decrypt(nil, ciphertext)
```

## That's it!