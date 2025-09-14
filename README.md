# NAS with restic rest-server

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
restic -r rest:http://<user>:<password>@<host>:<port>/<user> init
```

Delete repo

```shell
docker compose exec -it rest-server rm -rf /data/<user>
```