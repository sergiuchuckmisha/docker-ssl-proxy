# docker-ssl-websocket-proxy

Builds a basic nginx server that proxies incoming SSL calls to a target host
(usually another Docker container).
Included support of secure websocket connection (wss)

## Environment variables

The following environment variables configure nginx:

- ``DOMAIN``: domain in the SSL certificate (default value: ``www.example.com``)
- ``TARGET_PORT``: target port for the reverse proxy (default value: ``80``)
- ``TARGET_HOST``: target host for the reverse proxy (default value: ``proxyapp``)
- ``TARGET_HOST_HEADER``: value to be used as the Host header when sending
  requests to the target host (defaults to the value of ``$TARGET_HOST``)
- ``CLIENT_MAX_BODY_SIZE``: maximum size of client uploads (default value: ``20M``)
- ``SSL_PORT``: port ngnix SSL proxy listens on

## Certificates and CA location

The SSL certificate is generated using a own-ROOT-ca that is available in the
directory ``/etc/nginx/ca``, you may use Docker volumes to share the CAs with
other containers, so they can trust the installed certificate.

## Using own Certificate

You can use existing SSL certificates for your ``DOMAIN``
by connecting an volume onto ``/etc/nginx/certs`` with following files inside:

- ``key.pem``: private key file
- ``cert.pem``: certificate file

The certificate generator will check on existing ``key.pem`` and abort.

### example command to create docker volume with certs:
```bash
docker run -d --rm --name dummy -v certs_volume:/etc/nginx/certs alpine tail -f /dev/null
docker cp key.pem  dummy:/etc/nginx/certs/key.pem
docker cp cert.pem dummy:/etc/nginx/certs/cert.pem
docker stop dummy
```

## example command to start proxy via docker 

```bash
docker run -d --restart=on-failure:5 \
  --name docker-ssl-proxy \
  --link 'dockerhost' \
  -p 8443:8443 \
  -e 'TARGET_PORT=8000' \
  -e 'TARGET_HOST=dockerhost' \
  -e 'SSL_PORT=8443' \
  -v certs_volume:/etc/nginx/certs \
  sergiuchuckmisha/docker-ssl-sergiuchuckmisha-proxy:latest
```

## Docker Hub Image

You can get the publicly available docker image at the following location:
[docker-ssl-websocket-proxy](https://hub.docker.com/r/sergiuchuckmisha/docker-ssl-websocket-proxy).
