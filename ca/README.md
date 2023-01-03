## Setting up the CA

### The Root Certificate
The first task is to generate the root key pair:
```shell
openssl genrsa -aes256 -out private/ca.key.pem 4096

Enter pass phrase for ca.key.pem: experttls: xxxx
Verifying - Enter pass phrase for private/ca.key.pem: xxxx
```

Then create a root certificate, signed using the certificates own private key. 
All root certificates are signed this way, also note that a CSR is not required
for the root certificate

```shell
openssl req -config openssl.cfg -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem

Enter pass phrase for ca.key.pem: xxxx
```

### The Intermediate Certificate
We begin nby creating an intermediate key pair within ca/intermediate directory:

```shell
openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 2048

Enter pass phrase for intermediate.key.pem: xxxx
Verifying - Enter pass phrase for intermediate.key.pem: xxxx
```
Generate certificate signing request which will need to be signed by the root certificate

```shell
openssl req -config intermediate/openssl.cfg -new -sha256  \
 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
```
Sign the certificate signing request

```shell
openssl ca -config openssl.cfg -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
```

## Using the CA

### Generate the server key

```shell
openssl genrsa -aes256 -out intermediate/private/server.key.pem 2048

# Remove password from server key
openssl rsa -in intermediate/private/server.key.pem -out intermediate/private/server.key.pem

Enter pass phrase for server.key.pem: xxxx
```

### Generate Server CSR and Sign the Cert

```shell
openssl req -config intermediate/openssl.cfg -key intermediate/private/server.key.pem -new -sha256 -out intermediate/csr/server.csr.pem                               ~/dev/pythontls/ca  

Enter pass phrase for intermediate/private/server.key.pem:
```
If you were hosting a website, make sure CN matches domain name that you will be using.

Now we can sign the server certificate using the intermediate certificate (not root):

```shell
openssl ca -config intermediate/openssl.cfg -extensions server_cert -days 375 -notext -md sha256 -in intermediate/csr/server.csr.pem -out intermediate/certs/server.cert.pem
```

### Generate Client Key Client CSR and Sign the Cert

Generate Client Key

```shell
openssl genrsa -aes256 -out intermediate/private/client.key.pem 2048

# Remove password from client key
openssl rsa -in intermediate/private/client.key.pem -out intermediate/private/client.key.pem
Enter pass phrase for client.key.pem: xxxx
```

Create Client Certificate Signing Request

```shell
openssl req -config intermediate/openssl.cfg -key intermediate/private/client.key.pem -new -sha256 -out intermediate/csr/client.csr.pem                               ~/dev/pythontls/ca  

```

Sign the Client Certificate using the intermediate certificate
```shell
openssl ca -config intermediate/openssl.cfg -extensions usr_cert -days 375 -notext -md sha256 -in intermediate/csr/client.csr.pem -out intermediate/certs/client.cert.pem
```

