# Docker Munki SSL

A container that uses Using self-signed client certificate to serves static files at http://munki/repo using nginx.
Nginx expects the munki repo content to be located at /munki_repo. Use a data container and the --volumes-from option to add files.

*WARNING* This is in development.

## Versions

* 1.0, latest

# Usage
Creating Client Certs for Nginx:
---
## Using self-signed certificate.
### Create a Certificate Authority root (which represents this server)
Organization & Common Name: munkiclient

    openssl genrsa -des3 -out ca.key 4096
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt

### Create the Client Key and CSR
Organization & Common Name = munkiclient

    openssl genrsa -des3 -out client.key 4096
    openssl req -new -key client.key -out client.csr
    # self-signed
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt

#### Convert Client Key to PKCS
So that it may be installed in most browsers.

    openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

#### Convert Client Key to (combined) PEM
Combines `client.crt` and `client.key` into a single PEM file for programs using openssl.

    openssl pkcs12 -in client.p12 -out client.pem -clcerts

### Install Client Key on client device (OS or browser)
Use `client.p12`. Actual instructions vary.

## Create the Server Key and CRT
Organization & Common Name = munki.example.com

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt

## Creating a Data Container:
Create a data-only container to host the Munki repo.

    docker run -d --name munki-data --entrypoint /bin/echo ustwo/munki-ssl Data-only container for munki-ssl`

## Run the Munki container:
Run the container with --volumes-from and data container created and with port 443.

    docker run -d --name munki-ssl --volumes-from munki-data -p 443:443 -h munki-ssl-proxy ustwo/munki-ssl`

## Maintainers

* [Erik Aulin](mailto:erik@ustwo.com)

## Sources

* [afp548](https://www.afp548.com/2015/01/22/building-munki-with-docker)
* [mtigas](https://gist.github.com/mtigas/952344)
* [pravka](https://pravka.net/nginx-mutual-auth)
* [nginx](http://wiki.nginx.org/FullExample)
