# NAS with restic rest-server

Currently, the solution is to rely on [zero-conf networking](https://en.wikipedia.org/wiki/Zero-configuration_networking).
This way the NAS server can be accessed by using its **hostname** as url.
**WARNING:** Subdomains are not supported with this approach.

## Initialize

```shell
mkdir certs logs
```

## Management

Create user

```shell
docker compose exec -it rest-server create_user <user> <password>
```

Delete user

```shell
docker compose exec -it rest-server delete_user <user>
```

Create repo

```shell
restic -r rest:https://<user>:<password>@<host>/rest-server/<user> init --insecure-tls
```

Delete repo

```shell
docker compose exec -it rest-server rm -rf /data/<user>
```

List snapshots

```shell
restic -r rest:https://<user>:<password>@<host>/rest-server/<user> snapshots --insecure-tls
```

Create user credentials for traefik dashboard

```shell
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

## SSL self-signed certificates

**WARNING:** self-signed certs require the flag `--insecure-tls` when using restic.

```shell
HOSTNAME=$(hostname)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout certs/local.key \
    -out certs/local.crt \
    -subj "/CN=${HOSTNAME}" \
    -addext "subjectAltName = DNS:${HOSTNAME}"
```

## Photoprism

```shell
mkdir -p photoprism/database/flo
mkdir -p photoprism/database/maria
mkdir -p photoprism/storage/flo
mkdir -p photoprism/storage/maria
```

## Dynamic DNS

This is useful to make a local server accessible from the internet.
E.g. to get a valid Let's Encrypt SSL certificate (which won't work for local access...).

Using [dynv6](https://dynv6.com/) and [Telekom Speedport Smart 4 - Dynamic DNS](http://speedport.ip/html/content/internet/dyn_dns.html)

Instructions overview:

1. Create a [zone](https://dynv6.com/zones)
1. Use the zone to activate the [Dynamic DNS](http://speedport.ip/html/content/internet/dyn_dns.html) -> see settings section below
1. Secure the server as much as possible with auto-updates, ufw, etc.
1. Secure traefik as much as possible, e.g. with `exposedbydefault=false` and whitelisting
1. Activate [Portforwarding](http://speedport.ip/html/content/internet/portforwarding.html) to the ports traefik is using on the target server (80, 443)

### Settings for dynamic DNS

Provider: Other provider  
Host name: \<your-dynv6-domain\>  
User name: none  
Password: \<[your-http-token](https://dynv6.com/keys#token)\>  
Updateserver address: dynv6.com  
Protocol: HTTPS  
Port: 443