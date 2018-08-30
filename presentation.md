title: Docker basics hands-on
class: animation-fade
layout: true

<!-- This slide will serve as the base layout for all your slides -->
.bottom-bar[
  {{title}}
]

---

class: impact

# {{title}}

---

# Что такое Docker

<img src="https://cdn-images-1.medium.com/max/1400/1*9eqkIxHnho9ny2Tym5-I-w.jpeg" style="height:400px; float:right"/>

--

## Инфраструктура для

- упаковки

- хранения

- доставки

- запуска

- оркестрации

## приложений

---

# На чем это строится

--

- _Контейнеризация_ - виртуализация на уровне операционной системы Linux

- Контейнеры используют общее ядро операционной системы

- Изоляция с помощью [cgroups](https://en.wikipedia.org/wiki/Cgroups) и [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)


---

# Основные понятия

--

<img src="https://docs.docker.com/engine/images/architecture.svg" style="height:450px; float:right"/>

- Container

- Image

- Layer

- Docker host

- Docker registry

---

class: impact

# Практика

---

# Работа с контейнерами
--

Запустить ubuntu контейнер и зайти в bash в интерактивном режиме:
```shell
$ docker run -it ubuntu /bin/bash
```
--

То же, но примонтировав текущую хостовую дирректорию, причем, по одноименному пути:
```shell
$ docker run --rm -i -t -v "$PWD":"$PWD" --workdir "$PWD" ubuntu /bin/bash
```
--

Выполнить команду в новом процессе уже запущенного контейнера, например интерактивно зайти в новый bash шелл:
```shell
$ docker exec -it <container id | name> /bin/bash
```

---

# Работа с контейнерами

Посмотреть все запущенные контейнеры:

```shell
$ docker ps
```
--

Посмотреть все созданные контейнеры, включая незапущенные:
```shell
$ docker ps -a
```

Удалить контейнер (флаг `-f`: _forse_ удаление, даже запущенного контейнера):
```shell
$ docker rm <container id | name>
```
---

# Работа с образами

--

Посмотреть все образы на локальном docker host:
```shell
$ docker images
REPOSITORY    TAG       IMAGE ID        CREATED       SIZE
ubuntu        latest    16508e5c265d    7 days ago    84.1MB
```
--

Скачать новый образ:
```shell
$ docker pull hello-world:latest
```

```shell
$ docker images
REPOSITORY    TAG       IMAGE ID        CREATED       SIZE
ubuntu        latest    16508e5c265d    7 days ago    84.1MB
hello-world   latest    2cb0d9787c4d    7 weeks ago   1.85kB
```
--

Удалить докер образ (флаг `-f`: _forse_ удаление):
```shell
$ docker rmi
```
---

# Работа с образами

## Dockerfile. Cборка и тестрование приложения
--

В дирректории с проектом создаем _Dockerfile_

--
```dockerfile
# Compilaton and testing
FROM erlang:21.0 as build
```
--
```dockerfile
COPY ./ /build
WORKDIR /build
```
--
```dockerfile
RUN rebar3 compile
```
--
```dockerfile
RUN rebar3 do eunit, ct
```
---
# Работа с образами

## Билд образа

--
```shell
$ docker build ./
Sending build context to Docker daemon  1.859MB
Step 1/11 : FROM erlang:21.0 as build
 ---> 51aeeaecddeb

 ...
Removing intermediate container 595f1fdee99c
 ---> f5b5d971f2e0
Successfully built f5b5d971f2e0
```

- `./` - путь к дирректории с _Dockerfile_.
---

# Работа с образами

## Dockerfile. Сборка и упаковка приложения
--

В том же _Dockerfile_:
```dockerfile
RUN rm -rf _build/default/rel
RUN rebar3 release
```
--
```dockerfile
# Make an app image
FROM erlang:21.0
```
--
```dockerfile
WORKDIR /opt/dummy_docker
COPY --from=build /build/_build/default/rel/dummy_docker .
```
--
```dockerfile
CMD bin/dummy_docker foreground
```
---
# Работа с образами

## Билд образа. Финальный

--
```shell
$ docker build --squash -t dummy_docker:<tag> ./

Sending build context to Docker daemon  1.859MB
Step 1/11 : FROM erlang:21.0 as build
 ---> 51aeeaecddeb

...
Step 11/11 : CMD bin/dummy_docker foreground
 ---> Running in 94d00e80215c
Removing intermediate container 94d00e80215c
 ---> 87ad77b501bd
Successfully built 3d10516ba341
Successfully tagged dummy_docker:2018-08-30-19-15-15
```

- Таймстэмп в качестве тэга: `$(date +%Y-%m-%d-%H-%M-%S)`
---

# Работа с зависимостями

## docker-compose.yml

В дирректории с проектом создаем _docker-compose.yml_

--
```yaml
version: "3"

services:
  dummy:
    # build from Dockerfile
    build: ./
    depends_on:
      - db
  db:
    # pull and start from an image
    image: redis:4.0

```
---

# Работа с зависимостями

## Запуск

```shell
$ docker-compose up -d
```
--
```shell
$ docker-compose ps
        Name                     Command           State   Ports
---------------------------------------------------------------------
dummy_docker_db_1     docker-entrypoint.sh redis ...  Up     6379/tcp
dummy_docker_dummy_1  /bin/sh -c bin/dummy_docke ...  Up
```
---

# Работа с зависимостями

## Связность и остановка
Docker compose обеспечивает связь между контейнерами по `service` имени в _docker-compoe.yml_

```shell
$ docker exec -it dummy_docker_dummy_1 ping db
PING db (172.23.0.2) 56(84) bytes of data.
64 bytes from dummy_docker_db_1.dummy_docker_default (172.23.0.2): icmp_seq=1 ttl=64 time=0.093 ms
^C
--- db ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.093/0.093/0.093/0.000 ms
```

--
```shell
$ docker-compose down
```
---

class: impact

# Спасибо!
