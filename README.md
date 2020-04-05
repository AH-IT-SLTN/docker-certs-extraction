# Docker certificates extraction based on Traefik

Generate certificates based on [Traefik](https://docs.traefik.io/) docker from json file to crt, key, pem, pfx and like acme.sh

A certs-extraction container is available. It includes the latest development HEAD version. You can use it to manage certificates.

Prerequisite:

* Generate certificates by [Traefik](https://docs.traefik.io/)
* Get a valid acme.json file

Steps:
1. Detect change every 3s on acme.json file based on Traefik
2. Extract crt, key, pem, pfx files under certs/
3. Copy certificates like acme.sh under acme/
4. Duplicate acme certificates under `ACME_COPY`

Example:

    /var/docker/traefik
      - acme.json
      certs/
        - ssl-cert.key
        - ssl-cert.crt
        - ssl-cert.pem
        - ssl-cert.pfx
      acme/
        - ca.cer
        - fullchain.cer
        - sub.example.com.key
        - sub.example.com.cer

The environmental variables are as follows:
* `DOMAIN`: The domain name that you are updating. ie. sub.example.com
* `ACME_COPY`: the mounted volume to copy acme folder content. Use | separator for multiples folders (need to be mounted as volume on Docker)

## Installation via Docker

Please follow the official documentation:

    https://docs.docker.com/install/

### Docker

Get the container:
```bash
$ docker pull joweisberg/certs-extraction
```

Run the container in *console mode* (notice the environment variable setting parameters for the startup command):
```bash
$ docker run -d --restart="unless-stopped" -e DOMAIN="sub.example.com" -v /var/docker/traefik:/mnt/data joweisberg/certs-extraction
```

### Docker Compose

```yml
version: "3.5"
services:
  certs-extraction:
    container_name: certs-extraction
    image: joweisberg/certs-extraction:latest
    restart: unless-stopped
    environment:
      - DOMAIN=sub.example.com
      - ACME_COPY=/mnt/certs-to-copy
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - /var/docker/traefik:/mnt/data
        - /mnt/certs-to-copy:/mnt/certs-to-copy
```
