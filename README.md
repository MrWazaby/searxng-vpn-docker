# searxng-vpn-docker

Create a new private/auth protected SearXNG instance with a VPN for better privicy in five minutes using Docker

## What is included ?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [Caddy](https://github.com/caddyserver/caddy)    | Reverse proxy (create a LetsEncrypt certificate automatically) | [docker.io/library/caddy:2-alpine](https://hub.docker.com/_/caddy)               | [Dockerfile](https://github.com/caddyserver/caddy-docker/blob/master/Dockerfile.tmpl) |
| [SearXNG](https://github.com/searxng/searxng)    | SearXNG by itself                                              | [docker.io/searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng)     | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile)               |
| [Gluetun](https://github.com/qdm12/gluetun)      | VPN client                                                     | [docker.io/qmcgaw/gluetun:latest](https://hub.docker.com/r/qmcgaw/gluetun)       | [Dockerfile](https://github.com/qdm12/gluetun/blob/master/Dockerfile)                 |
| [Authelia](https://github.com/authelia/authelia) | Auth system to protect your private instance                   | [docker.io/authelia/authelia:latest](https://hub.docker.com/r/authelia/authelia) | [Dockerfile](https://github.com/authelia/authelia/blob/master/Dockerfile)             |

## How to use it

- Set up a A record on your DNS pointing to your public ip
- Set up a CNAME record on auth.your_domain.tld pointing to the previous A record
- [Install docker](https://docs.docker.com/install/)
- Get searxng-vpn-docker
  ```sh
  cd /usr/local
  git clone https://github.com/mrwazaby/searxng-vpn-docker.git
  cd searxng-vpn-docker
  ```
- Generate three secrets keys `openssl rand -hex 32` for `JWT_SECRET`, `ENCRYPTION_KEY` and `SESSION_SECRET`
- Create the `.env` file (`cp .env.example .env`) and edit it to set the variables
- To configure the VPN section refer to the [gluetun documentation](https://github.com/qdm12/gluetun-wiki/blob/main/setup/readme.md#setup)
- Edit the [searxng/settings.yml](https://github.com/mrwazaby/searxng-vpn-docker/blob/master/searxng/settings.yml) file according to your need
- Generate passwords for yout Authelia users `docker run -it authelia/authelia:latest authelia crypto hash generate argon2`
- Copy the user config [example file](https://github.com/mrwazaby/searxng-vpn-docker/blob/master/users_database.yml.example) into authelia/config and edit it according to your needs
- Check everything is working: `docker compose up`
- Run SearXNG in the background: `docker compose up -d`

> [!WARNING]  
> If you use an older version of docker desktop (`< 3.6.0`), you may have to install Docker Compose v1.
> Accordingly, you should modify the commands in this documentation to suit Docker Compose v1. For instance, change 'docker compose up' to 'docker-compose up'.
>
> [Install the docker-compose plugin](https://docs.docker.com/compose/install/#scenario-two-install-the-compose-plugin) (be sure that docker-compose version is at least 1.9.0)

> [!NOTE]  
> Windows users can use the following powershell script to generate the secret key:
> ```powershell
> $randomBytes = New-Object byte[] 32
> (New-Object Security.Cryptography.RNGCryptoServiceProvider).GetBytes($randomBytes)
> $secretKey = -join ($randomBytes | ForEach-Object { "{0:x2}" -f $_ })
> (Get-Content searxng/settings.yml) -replace 'ultrasecretkey', $secretKey | Set-Content searxng/settings.yml
> ```

## How to access the logs

To access the logs from all the containers use: `docker compose logs -f`.

To access the logs of one specific container:

- Caddy: `docker compose logs -f caddy`
- SearXNG: `docker compose logs -f searxng`
- Gluetun: `docker compose logs -f gluetun`
- Authelia : `docker compose logs -f authelia` 

### Start SearXNG with systemd

You can skip this step if you don't use systemd.

- ```cp searxng-vpn-docker.service.template searxng-vpn-docker.service```
- edit the content of ```WorkingDirectory``` in the ```searxng-vpn-docker.service``` file (only if the installation path is different from /usr/local/searxng-vpn-docker)
- Install the systemd unit:
  ```sh
  systemctl enable $(pwd)/searxng-vpn-docker.service
  systemctl start searxng-vpn-docker.service
  ```

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allow the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users want to disable the image proxy, you have to modify [./Caddyfile](https://github.com/mrwazaby/searxng-vpn-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:

- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
git pull
docker compose pull
docker compose up -d
```

Or the old way (with the old docker-compose version):

```sh
git pull
docker-compose pull
docker-compose up -d
```

## Inspirations

List of inspirations for this project: 
- [searxng-docker](https://github.com/searxng/searxng-docker)
- [Installing SearXNG with a VPN for private searching](https://sa.mtate.me.uk/posts/2024/installing-searxng/)
- [Authelia Docker](https://www.authelia.com/integration/deployment/docker/)
