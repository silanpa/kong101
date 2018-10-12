#Kong 101

>Comandos docker y http para Meetup Kong 101 *[Kong101](https://www.meetup.com/es-ES/Kong-SANTIAGO/events/254869963/)

##Antes de comenzar

Son necesarias algunas herramientas antes de poder hacer el run de Kong con Docker, y las peticiones con HTTPie.

* [Docker](https://docs.docker.com/install/)
* [HTTPie](https://httpie.org/doc)

En el caso de no poder instalar Docker en tu maquina, puedes utilizar Play-With-Docker

* [PlayWithDocker](https://labs.play-with-docker.com/)
* Cuenta en [Docker Cloud](https://cloud.docker.com/)

Inicio Postgres
---

```shell
docker run -d --name kong-database \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  postgres:9.5
```

Inicio Migracion
---

```shell
docker run --rm --link kong-database:kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  kong kong migrations up
```

Init Kong
---

```shell
docker run -d --name kong --link kong-database:kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_LOG_LEVEL=debug" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  kong
```

Instalar httpie
---

```shell
pip install --upgrade httpie
```

Probar Instalacion
---

```shell
http --pretty=all GET :8001 | less
```

Iniciar Backend de Ejemplo
---

```shell
docker run -d -p 80:80 kennethreitz/httpbin
```

Crear service y route para Ejemplo
---

```shell
http POST :8001/services name=ejemplo url=http://172.17.0.4

http POST :8001/services/ejemplo/routes paths:='["/httpbin"]'

http GET :8000/httpbin/get
```

Agregar plugins rate-limiting a los consumers
---

```shell
http GET :8001/consumers/meetup

http POST :8001/consumers username=kong

http POST :8001/consumers/kong/key-auth key=kong

http POST :8001/services/ejemplo/plugins name=rate-limiting config.minute=5 consumer_id=
http POST :8001/services/ejemplo/plugins name=rate-limiting config.minute=100 consumer_id=

http GET :8000/httpbin/headers Apikey:kong101
http GET :8000/httpbin/headers Apikey:kong
```

Agregar plugins API key
---

```shell
http POST :8001/services/ejemplo/plugins name=key-auth

http POST :8001/consumers username=meetup

http POST :8001/consumers/meetup/key-auth key=kong101

http GET :8000/httpbin/headers Apikey:123456

```

Upstream y target
---

```shell
http POST :8001/upstreams name=cl.httpbin.service

http POST :8001/consumers username=meetup

http POST :8001/consumers/meetup/key-auth key=kong101

http GET :8000/httpbin/headers Apikey:kong101

```

Extras
---

```shell
docker run -d --name kong-2 --link kong-database:kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_LOG_LEVEL=debug" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
  -p 8100:8000 \
  -p 8543:8443 \
  -p 8101:8001 \
  -p 8544:8444 \
  kong

```
