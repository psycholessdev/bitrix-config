# dev сайт - контейнерное окружение для Битрикс

Проект представляет собой dev сайт, он же девелоперский сайт, предназначенный для тестирования, разработки, примера использования окружения на базе технологий Docker.

Программы, необходимые для работы технологий Битрикс, запускаются в контейнерах Docker. По шаблону из docker образов. Данные хранят в docker томах. Связаны между собой посредством docker сети. И управляются (оркестируются) использую Docker Compose.

> [!CAUTION]
> Внимание! dev сайт не предназначен для использования в продакшене. (___!!!ToDo_дополнить_и_расписать___)

# Содержимое

* [Docker и Docker Compose](#docker)
* [Сборка или скачавание Docker образов](#dockerimages)
  * [Базовые образы](#basicimages)
  * [Битрикс образы (bx-*)](#bitriximages)
* [Управление](#management)
* [Адресация](#iporurls)
* [Порты](#ports)
* [Доступ к сайту](#siteaccess)
* [Базовая проверка конфигурации веб-сервера](#bitrixservertestphp)
* [BitrixSetup / Restore](#bitrixsetupphprestorephp)
* [Установка дистрибутивов](#installdistro)
* [Восстановление из резервной копии](#restorebackup)

<a id="docker"></a>
# Docker и Docker Compose

Для запуска проекта понадобится Docker. Для оркестировки и управления Docker Compose.

Способ развертывания зависит от вашей операционной системы, используемой на хосте.

Возможны варианты:
- рабочая станция, персональный компьютер, ноутбук и т.д. с рабочим столом и графической средой, он же desktop
- сервер, без графической среды, но с консолью или удаленным доступом, он же server

Для взаимодействия с docker в графическом режиме будем использовать продукт [Docker Desktop](https://docs.docker.com/desktop/), который возможно запустить на ОС Windows, Linux, MacOS.

Ознакомтесь с документацией и выполните развертывание docker, в зависимости от используемой вами ОС:
- Docker Desktop on Windows: https://docs.docker.com/desktop/setup/install/windows-install/
- Docker Desktop on Linux: https://docs.docker.com/desktop/setup/install/linux/
- Docker Desktop on Mac: https://docs.docker.com/desktop/setup/install/mac-install/

Для взаимодействия с docker в режиме командной строки (без графического режима) будем использовать продукт [Docker Engine](https://docs.docker.com/engine/), который возможно запустить на ОС Linux.

Ознакомтесь с документацией и выполните развертывание docker, в зависимости от используемой вами ОС Linux:
- Docker Engine: https://docs.docker.com/engine/install/

В современных версиях продуктов Docker обычно в их состав уже включен Docker Compose.

Ознакомтесь с документацией и выполните развертывание Docker Compose, если это требуется отдельно:
- Docker Compose: https://docs.docker.com/compose/

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

<a id="iporurls"></a>
# Адресация

Обращение к запущенному сайту может быть:
1) через `localhost` или `127.0.0.1`
2) по IP вашей локальной сети вида `10.X.X.X`, `192.X.X.X`, `172.X.X.X` и т.д.
3) по имени локального домена в вашей локальной сети вида `dev.bx`
4) по IP глобальной сети интернет вида `85.86.87.88`
5) по имени настоящего домена вида `devexample.com`

Выбор зависит от вашей конфигурации.

Так же стоит учесть: при использовании https нужен ssl сертификат, который можно сгенерировать или получить на доменное имя, но не на IP.

<a id="ports"></a>
# Порты

Запущенный сайт для своей работы по умолчанию использует порты:
- `8858` для http
- `8859` для https

В случае если вы используете файрволл необходимо открыть порты. Пример команд для разных ОС:

- Debian/Ubuntu и т.д.:
```bash
ufw allow 8588 && ufw allow 8589
```

- RHEL/CentOS/AlmaLinux/RockyLinux/OracleLinux/ArchLinux и т.д.:
```bash
firewall-cmd --add-port=8858/tcp --permanent && firewall-cmd --add-port=8859/tcp --permanent && firewall-cmd --reload
```

Для других ОС смотрите документацию по работе с файрволлом.

<a id="siteaccess"></a>
# Доступ к сайту

Итак, согласно разделам Адресация и Порты выше, к сайту можно обратиться по http/https следующим образом:

- через локалхост:
  - http://127.0.0.1:8858/
  - https://127.0.0.1:8859/

- по локальному ip:
  - http://10.0.1.119:8858/
  - https://10.0.1.119:8859/

- используя домен:
  - http://dev.bx:8858/
  - https://dev.bx:8859/

> [!IMPORTANT]
> <b>НЕ</b> используйте `127.0.0.1` или `localhost` при работе с сайтом на локальной машине. Используйте ip или домен, пример `10.0.1.119` или `dev.bx`.

В примерах ниже будет использоваться локальный ip `10.0.1.119`.

<a id="bitrixservertestphp"></a>
# Базовая проверка конфигурации веб-сервера

После запуска сайта необходимо провести базовую проверку конфигурации веб-сервера. Она выполняется с помощью скрипта `bitrix_server_test.php`.

Используем способ, который работает одинаково на всех ОС.

Заходим в sh-консоль контейнера `php` из под `root`:
```bash
docker compose exec --user=root php sh
```

Переходим в корневой каталог сайта и скачиваем скрипт `bitrix_server_test.php`:
```bash
cd /opt/www/
wget https://dev.1c-bitrix.ru/download/scripts/bitrix_server_test.php
```

В браузере переходим по ссылке вида:
```bash
http://10.0.1.119:8588/bitrix_server_test.php
```

После проверки скрипт нужно удалить.

PS: для Docker Engine на Linux расположение каталога сайта на хосте зависит от режима работы docker-а:
- rootfull:
```bash
/var/lib/docker/volumes/dev_www_data/_data/
```

- rootless:
```bash
/home/[USERNAME]/.local/share/docker/volumes/dev_www_data/_data
```

<a id="bitrixsetupphprestorephp"></a>
# BitrixSetup / Restore

Для установки продуктов Битрикс можно использовать скрипт `bitrixsetup.php`: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&LESSON_ID=4523&LESSON_PATH=10495.4495.4523

Для восстановления из резервной копии скрипт `restore.php`: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=02014&LESSON_PATH=10495.4496.2014

Заходим в sh-консоль контейнера `php` из под `root`:
```bash
docker compose exec --user=root php sh
```

Переходим в корневой каталог сайта и скачиваем скрипт(ы):
```bash
cd /opt/www/
wget https://www.1c-bitrix.ru/download/scripts/bitrixsetup.php
wget https://www.1c-bitrix.ru/download/scripts/restore.php
```

В браузере переходим по ссылке вида:
- для установки:
```bash
http://10.0.1.119:8588/bitrixsetup.php
```

- для восстановления:
```bash
http://10.0.1.119:8588/restore.php
```

Устанавливаем продукт или восстанавливаем сайт, зависит от вашего выбора.

<a id="installdistro"></a>
# Установка дистрибутивов

Используем скрипт `bitrixsetup.php`, скачиваем дистрибутив Битрикс, будь это демо версия или лицензионная версия. Редакция и тип базы зависит от вашего выбора.
После переходим на сайт используя урл:
```bash
http://10.0.1.119:8588/
```

Проходим мастер установки дистрибутива:
- `1С-Битрикс: Управление сайтом` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522
- `1С-Битрикс24` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522

На шаге создания базы вводим параметры подключения к ней:

1) для `mysql` версии:
Имя хоста (алиас) - `mysql`
Имя суперпользователя - `root`
Пароль суперпользователя - `BiTRiX@#2025`

2) для `pgsql` версии:
Имя хоста (алиас) - `postgres`
Имя суперпользователя - `postgres`
Пароль суперпользователя - `BiTRiX@#2025`

Пароль суперпользователя для обеих баз хранится в файле `.env_sql` и по умолчанию его значение равно `BiTRiX@#2025`.

<a id="restorebackup"></a>
# Восстановление из резервной копии

Используем скрипт `restore.php`.............

(___ToDo___)

......ToDo......
