# Kong 101

>Comandos docker y httpie para Meetup Kong 101 [Kong101](https://www.meetup.com/es-ES/Kong-SANTIAGO/events/254869963/)

## Antes de comenzar

Son necesarias algunas herramientas antes de comenzar con el Meetup de Kong.

* [Docker](https://docs.docker.com/install/)
* [HTTPie](https://httpie.org/doc)

Instalar httpie
---
La forma mas fácil según la documentación es con lo siguiente:

```shell
pip install --upgrade httpie
```

En el caso de no poder instalar Docker en tu maquina, puedes utilizar play-with-docker. Pero los commandos estan enfocados a docker en Linux, cualquier duda consultar.

* [Play with Docker](https://labs.play-with-docker.com/)
* Cuenta en [Docker Cloud](https://cloud.docker.com/) necesaria para el login de play-with-docker

Inicio DB Postgres
---

```shell
docker run -d --name kong-database \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  postgres:9.5
```

Inicio Migracion DB
---

```shell
docker run --rm --link kong-database:kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  kong kong migrations up
```

Inicio Kong
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

Probar Instalacion
---

```shell
http --pretty=all GET :8001
```

Iniciar Backend de Ejemplo
---
A simple HTTP Request & Response Service [Httpbin](https://httpbin.org/)

```shell
docker run -d -p 80:80 kennethreitz/httpbin
```

Crear services y routes para Ejemplo
---

```shell
http POST :8001/services name=ejemplo url=http://172.17.0.4
http POST :8001/services/ejemplo/routes paths:='["/httpbin"]'
http GET :8000/httpbin/get
```

Tambien se puede utilizar directamente desde el servicio en internet

```shell
http POST :8001/services name=ejemplo-dos url=https://httpbin.org/
http POST :8001/services/ejemplo-dos/routes paths:='["/httpbin-dos"]'
http GET :8000/httpbin-dos/get
```

Agregar plugins rate-limiting a services y routes
---
Los plugins se pueden asociar a los servicios, routes y consumers.

```shell

http POST :8001/services/ejemplo/plugins name=rate-limiting config.minute=5 
```
Para router es necesario pasar el id asociado. Se puede obtener desde /routes o /services
```shell

http POST :8001/routes 
http POST :8001/services/ejemplo/routes
http POST :8001/services/ejemplo-dos/routes
http POST :8001/routes/{id_route}/plugins name=rate-limiting config.minute=10
```

Agregar plugins API key
---

```shell
http POST :8001/services/ejemplo/plugins name=key-auth

http POST :8001/consumers username=meetup

http POST :8001/consumers/meetup/key-auth key=kong101

http GET :8000/httpbin/headers Apikey:123456

http GET :8000/httpbin/headers Apikey:kong101

```

Extras
---

Agregar nuevo nodo de kong en diferentes puertos

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
