# dev сайт - контейнерное окружение для Битрикс

Проект представляет собой dev сайт, он же девелоперский сайт, предназначенный для тестирования, разработки, примера использования окружения на базе технологий Docker.

Программы, необходимые для работы технологий Битрикс, запускаются в контейнерах Docker. По шаблону из docker образов. Данные хранят в docker томах. Связаны между собой посредством docker сети. И управляются (оркестируются) использую Docker Compose.

> [!CAUTION]
> Внимание! dev сайт не предназначен для использования в продакшене. (___!!!ToDo_дополнить_и_расписать___)

# Содержимое

* [Сборка или скачавание Docker образов](#dockerimages)
  * [Базовые образы](#basicimages)
  * [Битрикс образы (bx-*)](#bitriximages)
* [Управление](#management)

<a id="dockerimages"></a>
# Сборка или скачавание Docker образов

Наша цель - подготовить или собрать максимально совместимые и готовые для Bitrix образы, запускающие набор ПО в контейнерах.

<a id="basicimages"></a>
## Базовые образы

Там где возможно будем использовать официальные Docker образы ПО, теги для которых будем брать с DockerHub-а:
- Percona Server: https://hub.docker.com/_/percona
- PostgreSQL: https://hub.docker.com/_/postgres
- Redis: https://hub.docker.com/_/redis
- Memcached: https://hub.docker.com/_/memcached

В этот список попадают (формат `название`:`полный_тег_с_указанием_версии_и_ос`):
- `percona/percona-server:8.0.40`
- `postgres:16.8-bookworm`
- `redis:7.2.7-alpine`
- `memcached:1.6.37-alpine`

Можно предварительно скачать список выше, используя команды:
```bash
docker pull percona/percona-server:8.0.40
docker pull postgres:16.8-bookworm
docker pull redis:7.2.7-alpine
docker pull memcached:1.6.37-alpine
```

<a id="bitriximages"></a>
## Битрикс образы (bx-*)

Так же нам понадобятся:
- веб сервер: используем стабильный `nginx:1.26.3-alpine-slim`, добавим модули слоем сверху, соберем `bx-nginx:1.26.3-alpine`
- интерпритарор php кода: готового совместимого образа php увы нет, берем по умолчанию `php:8.2.28-fpm-alpine` и добавляем то, что нам надо через пару слоев сверху, соберем `bx-php:8.2.28-fpm-alpine`
- поиск: готового образа sphinx нет, но есть собранный пакет sphinx на базе alpine linux в официальной репе, соберем `bx-sphinx:2.2.11-alpine` установив пакет
- push сервер: готового образа нет, используем NodeJS 20-ой версии, соберем `bx-push:3.0-alpine`, используя его исходники `push-server-0.4.0`

Список официальных Docker образов, которые будем брать с DockerHub-а:
- Nginx: https://hub.docker.com/_/nginx
- PHP: https://hub.docker.com/_/php
- NodeJS: https://hub.docker.com/_/node

Для сборки нам понадобятся следующие образы (их можно предварительно скачать используя команды):
```bash
docker pull nginx:1.26.3-alpine-slim
docker pull php:8.2.28-fpm-alpine
docker pull alpine:3.21
docker pull node:20
docker pull node:20-alpine
```

Собираем образы, в названии используем bx-:

- bx-sphinx:
```bash
cd dev/sources/bxsphinx2211/
docker build -f Dockerfile -t bx-sphinx:2.2.11-alpine --no-cache .
```

- bx-push:
```bash
cd dev/sources/bxpush30/
docker build -f Dockerfile -t bx-push:3.0-alpine --no-cache .
```

- bx-php:
```bash
cd dev/sources/bxphp8228/
docker build -f Dockerfile -t bx-php:8.2.28-fpm-alpine --no-cache .
```

- bx-nginx:
```bash
cd dev/sources/bxnginx1263/
docker build -f Dockerfile -t bx-nginx:1.26.3-alpine --no-cache .
```

<a id="management"></a>
# Управление

Управление (или оркестровка) набором контейнеров или одним из контейнеров осуществляется через Docker Compose.

Переходим в каталог проекта `cd dev` и выполняем команды ниже:

Запустить все контейнеры и оставить их работать в фоне:
```bash
docker compose up -d
```

Отобразить список контейнеров и их статус:
```bash
docker compose ps
```

Показать логи сразу всех контейнеров:
```bash
docker compose logs
```

Показать лог определенного сервиса-контейнера:
```bash
docker compose logs redis
```

Перезапустить определенный контейнер:
```bash
docker compose restart nginx
```

Перезапустить все контейнеры:
```bash
docker compose restart
```

Остановить все контейнеры:
```bash
docker compose stop
```

Остановить все контейнеры, удалить их:
```bash
docker compose down
```

Остановить все контейнеры, удалить их и удалить все тома этих контейнеров:
```bash
docker compose down -v
```

Зайти в sh консоль определенного контейнера, например nginx:
```bash
docker compose exec nginx sh
```

Подробней в документации docker compose: https://docs.docker.com/reference/cli/docker/compose/

......ToDo......
