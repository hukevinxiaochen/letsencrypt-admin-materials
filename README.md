# Obtaining or Renewing an SSL Certificate from Lets Encrypt

## Materials Needed

* user account key pair (private and public)
  - `./user.key`
  - `./user.pub`
* openssl
* python
* ACME client script
  - `./sign_csr.py`

## Concepts

* [How SSL certificates work]()
* [How Let's Encrypt works]()
* [What is TLS?]()

## Protocol

_You already have a user account key. Skip this step._

### 1 Generate a user account key pair for Let's Encrypt

First, you need to generate an user account key for Let's Encrypt. This is the key that you use to register with Let's Encrypt. If you already have user account key with Let's Encrypt, you can skip this step.

```
openssl genrsa 4096 > user.key
openssl rsa -in user.key -pubout > user.pub
```

_Your user account keys are in:_

`./user.key`

`./user.pub`

### 2 To apply for a new certificate or to renew, generate a new domain key (domain.key) and new signing request (domain.csr)

```
openssl genrsa 4096 > domain.key
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:ericakevin.com,DNS:www.ericakevin.com")) > domain.csr
```

### 3 Run the signing script/ACME protocol with the file-based option: 

`python sign_csr.py --public-key user.pub --file-based domain.csr > signed.crt`

The file-based option is for when you already have a server listening on port 80.

### 4 Complete the challenges

Your challenges have always been serving a file in the `.well-known` directory of the python web app.

### 5 Generate a certificate chain and concatenate to your signed cert

[What's my chain cert?](https://whatsmychaincert.com/)

For nginx, you serve the signed and chain certification as a single concatenated file.

`cat signed.crt chained.crt > signed-chained.crt`

### 6 Serve the domain.key and the signed.crt

Serve in the directory `/var/www/ssl`

### 7 Restart NGINX

`sudo service nginx restart`

