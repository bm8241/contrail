## 1 SSL Connection
Client: CA public key
Server: server private key, server certificate

Client connects to server, requires server certificate, uses CA public key to verify server certificate and get server public key.

## 2 CA Signing Key and Certificate
A signing certificate and private key are required to handle the certificate signing request from server and issue the server certificate.
```
                                                     | CA root |    | CA root |
                                                     |   key   |<-->|  cert   |
                                                          |                |
                       | CA signing |   | CA signing |    v                v  | CA signing | 
                       |    key     |-->|    CSR     |----------------------->|   cert     |
                             |                                                     |
| server |    | server |     v                                                     v       | server |
|  key   |--> |  CSR   |------------------------------------------------------------------>|  cert  |
```

Here is an example to build CA certificate chain.

### 2.1 CA Root Key and Certificate
Generate root key pair. If root key pair is encrypted (with argument -des3), a pass phrase is required. CA root certificate is self-signed by CA root private key. The pass phrase is required if key pair is encrypted. Argument -x509 is for self-singing. Generated CA root certificate is not encrypted (PEM format).
```
openssl genrsa -des3 -out ca-root.key 1024 
openssl req -new -x509 -days 3650 -key ca-root.key -out ca-root.crt  
```

Here is an alternative to generate decrypted key pair and self-signed certificate together.
```
openssl req -newkey rsa:1024 -nodes -keyout ca-root.key -x509 -days 3650 -out ca-root.crt
```

### 2.2 CA Signing Key and Certificate
Generate signing key pair and certificate.
```
openssl genrsa -des3 -out ca-sign.key 1024 
openssl req -new -key ca-sign.key -out ca-sign.csr -subj "/CN=ca-sign@ca.com" 
```

Alternatively, generate decrypted key pair and CRS together.
```
openssl req -newkey rsa:1024 -nodes -keyout ca-sign.key -out ca-sign.csr -subj "/CN=ca-sign@ca.com"
```

Use CA root key and certificate to generate CA signing certificate.
```
openssl x509 -req -days 3650 -in ca-sign.csr -CA ca-root.crt -CAkey ca-root.key -set_serial 01 -out ca-sign.crt 
```

Now, CA signing key and certificate can be used to handle CSR and issue certificate for others.

## 3 Server Key and Certificate
### 3.1 Key Pair
Server generates encrypted private-public key pair, and decrypts it. When generate encrypted key pair, a pass phrase is required to encrypt it. The pass phrase is also required to decrypt key pair.
```
openssl genrsa -des3 -out server_encrypted.key 1024
openssl rsa -in server_encrypted.key -out server.key
```

Or generate a not encrypted key pair. This doesn't require the pass phrase.
```
openssl genrsa -out server.key 1024
```

### 3.2 Certificate Signing Request
Server creates certificate signing request with the public key (extracted from the key pair).
```
openssl req -new -key server.key -out server.csr -subj "/CN=server@acme.com"
```

### 3.3 Certificate
Server gives CSR to CA who uses signing certificate and signing key to generate server certificate (signed public key).
```
openssl x509 -req -days 3650 -in server.csr  -CA ca-sign.crt -CAkey ca-sign.key -set_serial 01 -out server.crt
```

## Appendix A.1
```
create_key()
{
    # Create CA root key and self-signed certificate.
    openssl req -newkey rsa:1024 -nodes -keyout ca-root.key -x509 \
        -days 3650 -out ca-root.crt
    # Create CA signing key and certificate signing request.
    openssl req -newkey rsa:1024 -nodes -keyout ca-sign.key \
        -out ca-sign.csr -subj "/CN=ca-sign@ca.com"
    # Create CA signing certificate.
    openssl x509 -req -days 3650 -in ca-sign.csr -CA ca-root.crt \
        -CAkey ca-root.key -set_serial 01 -out ca-sign.crt
    # Create server key and certificate signing request.
    openssl req -newkey rsa:1024 -nodes -keyout server.key \
        -out server.csr -subj "/CN=admin@acme.com"
    # Create server certificate.
    openssl x509 -req -days 3650 -in server.csr -CA ca-sign.crt \
        -CAkey ca-sign.key -set_serial 01 -out server.crt
}
```
