# dev сайт - контейнерное окружение для Битрикс

Проект представляет собой dev сайт, он же девелоперский сайт, предназначенный для тестирования, разработки, примера использования окружения на базе технологий [Docker](https://www.docker.com/).

Программы, необходимые для работы технологий Битрикс, запускаются в контейнерах `Docker`. По шаблону из `Docker образов`. Данные хранят в `Docker томах`. Связаны между собой посредством `Docker сети`. И управляются (оркестируются) использую `Docker Compose`.

> [!CAUTION]
> Внимание! dev сайт не предназначен для использования в продакшене. (___!!!ToDo_дополнить_и_расписать___)

# Содержимое

* [Docker и Docker Compose](#docker)
* [Сборка или скачивание Docker образов](#dockerimages)
  * [Базовые образы](#basicimages)
  * [Битрикс образы (bx-*)](#bitriximages)
* [Управление](#management)
* [Часовой пояс (timezone)](#timezone)
* [Адресация](#iporurls)
* [Порты](#ports)
* [Доступ к сайту](#siteaccess)
* [Базовая проверка конфигурации веб-сервера](#bitrixservertestphp)
* [BitrixSetup / Restore](#bitrixsetupphprestorephp)
* [Установка дистрибутивов](#installdistro)
* [Восстановление из резервной копии](#restorebackup)
* [Настройки модулей](#modulessettings)
* [Проверка системы](#sitechecker)
* [Тест производительности](#perfmonpanel)
* [Cron](#cron)
  * [Выполнение агентов на cron](#agentsoncron)
  * [Кастомные cron задания](#owncrontasks)
* [Хранение временных файлов вне корневой директории сайта](#bxtemporaryfilesdirectory)
* [Push & pull](#pushpull)
  * [Push сервер](#pushserver)
* [Sphinx](#sphinx)
  * [Поиск с помощью Sphinx](#sphinxsearch)

<a id="docker"></a>
# Docker и Docker Compose

Для запуска проекта понадобится `Docker`. Для оркестировки и управления `Docker Compose`.

Способ развертывания зависит от вашей операционной системы, используемой на хосте.

Возможны варианты:
- рабочая станция, персональный компьютер, ноутбук и т.д. с рабочим столом и графической средой, он же desktop
- сервер, без графической среды, но с консолью или удаленным доступом, он же server

Для взаимодействия с `Docker` в графическом режиме будем использовать продукт [Docker Desktop](https://docs.docker.com/desktop/), который возможно запустить на ОС Windows, Linux, MacOS.

Ознакомтесь с документацией и выполните развертывание `Docker`, в зависимости от используемой вами ОС:
- `Docker Desktop on Windows`: https://docs.docker.com/desktop/setup/install/windows-install/
- `Docker Desktop on Linux`: https://docs.docker.com/desktop/setup/install/linux/
- `Docker Desktop on Mac`: https://docs.docker.com/desktop/setup/install/mac-install/

Для взаимодействия с `Docker` в режиме командной строки (без графического режима) будем использовать продукт [Docker Engine](https://docs.docker.com/engine/), который возможно запустить на ОС Linux.

Ознакомтесь с документацией и выполните развертывание `Docker Engine`, в зависимости от используемой вами ОС Linux:
- `Docker Engine`: https://docs.docker.com/engine/install/

В современных версиях продуктов `Docker` обычно в их состав уже включен `Docker Compose`.

Ознакомтесь с документацией и выполните развертывание `Docker Compose`, если это требуется отдельно:
- `Docker Compose`: https://docs.docker.com/compose/

<a id="dockerimages"></a>
# Сборка или скачивание Docker образов

Наша цель - подготовить или собрать максимально совместимые и готовые для Bitrix образы, запускающие набор ПО в контейнерах.

<a id="basicimages"></a>
## Базовые образы

Там где возможно будем использовать официальные Docker образы ПО, теги для которых будем брать с [DockerHub](https://hub.docker.com/):
- `Percona Server`: https://hub.docker.com/_/percona
- `PostgreSQL`: https://hub.docker.com/_/postgres
- `Redis`: https://hub.docker.com/_/redis
- `Memcached`: https://hub.docker.com/_/memcached

В этот список попадают (формат `название`:`полный_тег_с_указанием_версии_и_ос`):
- `percona/percona-server:8.0.40`
- `postgres:16.8-bookworm`
- `redis:7.2.7-alpine`
- `memcached:1.6.38-alpine`

Можно предварительно скачать список выше, используя команды:
```bash
docker pull percona/percona-server:8.0.40
docker pull postgres:16.8-bookworm
docker pull redis:7.2.7-alpine
docker pull memcached:1.6.38-alpine
```

<a id="bitriximages"></a>
## Битрикс образы (bx-*)

Так же нам понадобятся:
- веб сервер: используем стабильный `nginx:1.26.3-alpine-slim`, добавим модули слоем сверху, соберем `bx-nginx:1.26.3-alpine`
- интерпритарор php кода: готового совместимого образа php увы нет, берем по умолчанию `php:8.2.28-fpm-alpine` и добавляем то, что нам надо через пару слоев сверху, соберем `bx-php:8.2.28-fpm-alpine`
- поиск: готового образа sphinx нет, но есть собранный пакет sphinx на базе alpine linux в официальной репе, соберем `bx-sphinx:2.2.11-alpine` установив пакет
- push сервер: готового образа нет, используем NodeJS 20-ой версии, соберем `bx-push:3.0-alpine`, используя его исходники `push-server-0.4.0`

Список официальных Docker образов, которые будем брать с [DockerHub](https://hub.docker.com/):
- `Nginx`: https://hub.docker.com/_/nginx
- `PHP`: https://hub.docker.com/_/php
- `NodeJS`: https://hub.docker.com/_/node
- `Alpine`: https://hub.docker.com/_/alpine

Для сборки нам понадобятся следующие образы (их можно предварительно скачать используя команды):
```bash
docker pull nginx:1.26.3-alpine-slim
docker pull php:8.2.28-fpm-alpine
docker pull node:20
docker pull node:20-alpine
docker pull alpine:3.21
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

<a id="timezone"></a>
# Часовой пояс (timezone)

Значение таймзоны, используемой контейнерами, хранится в файле `.env`.
Значение по умолчание задано как:
```bash
TZ=Europe/Kaliningrad
```

Исключение составляет контейнер с `php`, значение тамйзоны для которого продублировано в отдельном файле `confs/php/etc/php/conf.d/timezone.ini` как:
```bash
date.timezone = Europe/Kaliningrad
```

При необходимости смените значение в обоих файлах. Перезапустите все контейнеры:
```bash
docker compose restart
```

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

Итак, согласно разделам `Адресация` и `Порты` выше, к сайту можно обратиться по http/https следующим образом:

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

В примерах ниже будет использоваться локальный ip `10.0.1.119` или локальный домен `dev.bx`.

<a id="bitrixservertestphp"></a>
# Базовая проверка конфигурации веб-сервера

После запуска сайта необходимо провести базовую проверку конфигурации веб-сервера. Она выполняется с помощью скрипта `bitrix_server_test.php`.

Используем способ, который работает одинаково на всех ОС.

Заходим в sh-консоль контейнера `php` из под пользователя `root`:
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

Заходим в sh-консоль контейнера `php` из под пользователя `root`:
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

Используем скрипт `bitrixsetup.php`, скачиваем дистрибутив Битрикс, будь это демо версия или лицензионная версия.

Редакция и тип базы зависит от вашего выбора.

После переходим на сайт используя урл:
```bash
http://10.0.1.119:8588/
```

Проходим мастер установки дистрибутива:
- `1С-Битрикс: Управление сайтом` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522
- `1С-Битрикс24` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522

На шаге создания базы вводим параметры подключения к ней:

- для `mysql` версии:
  - Имя хоста (алиас) - `mysql`
  - Имя суперпользователя - `root`
  - Пароль суперпользователя - `BiTRiX@#2025`

- для `pgsql` версии:
  - Имя хоста (алиас) - `postgres`
  - Имя суперпользователя - `postgres`
  - Пароль суперпользователя - `BiTRiX@#2025`

Пароль суперпользователя для обеих баз хранится в файле `.env_sql` и по умолчанию его значение равно `BiTRiX@#2025`.

<a id="restorebackup"></a>
# Восстановление из резервной копии

Используем скрипт `restore.php`.............

(___ToDo___)

<a id="modulessettings"></a>
# Настройки модулей

После установки дистрибутива или восстановления сайта из бекапа необходимо подстроить сайт.

Например, настроить `Главный модуль (main)` (`/bitrix/admin/settings.php?lang=ru&mid=main`).

(___ToDo_описать_важные_опции_и_ссылка_на_доку___)

<a id="sitechecker"></a>
# Проверка системы

Для всесторонней проверки соответствия параметров системы, на которой осуществляется функционирование сайта, минимальным и рекомендуемым техническим требованиям продукта используйте страницу `Проверка системы` (`/bitrix/admin/site_checker.php?lang=ru`).

Документация:
- https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=35&LESSON_ID=14020&LESSON_PATH=3906.4493.4506.2024.14020
- https://dev.1c-bitrix.ru/user_help/settings/utilities/site_checker.php

<a id="perfmonpanel"></a>
# Тест производительности

Для оценки производительности перейдите на страницу `Панель производительности` (`/bitrix/admin/perfmon_panel.php?lang=ru`).

Выполните тест конфигурации. Результаты подскажут "узкие" места системы или её конфигурации.

Документация: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=35&CHAPTER_ID=03376&LESSON_PATH=3906.6663.4904.3376

<a id="cron"></a>
# Cron

<a id="agentsoncron"></a>
## Выполнение агентов на cron

Для работы cron заданий используется отдельный контейнер `cron` на базе образа `php`.

По умолчанию контейнер сконфигурирован на выполнение заданий модуля `Главный модуль (main)` раз в минуту:
```bash
php -f /opt/www/bitrix/modules/main/tools/cron_events.php
```

Это задание будет выполнятся только если дистрибутив установлен.

Для включения выполнения агентов на кроне нужно отредактировать файл `/bitrix/php_interface/dbconn.php`, добавить строку:
```bash
define('BX_CRONTAB_SUPPORT', true);
```

И сохранить.

Проверить настройку можно на странице `Проверка системы` (`/bitrix/admin/site_checker.php?lang=ru`) и на странице `Список агентов` (`/bitrix/admin/agent_list.php?lang=ru`).

<a id="owncrontasks"></a>
## Кастомные cron задания

Cron задания хранятся в папке `/etc/periodic/` контейнера `cron`.

Внутри каталога размещены подкаталоги, указывающие на периодичность запуска заданий:
```bash
1min
15min
hourly
daily
weekly
monthly
```

Для примера создадим задание, которое выполняется раз в сутки - создание резервной копии сайта с базой MySQL - бекап.

Заходим в sh-консоль контейнера `cron` из под пользователя `root`:
```bash
docker compose exec --user=root cron sh
```

В каталоге `/etc/periodic/daily/` создадим файл `backup` без указания расширения файла.
```bash
apk add mc
mcedit /etc/periodic/daily/backup
```

Заполним содержимое файла:
```bash
#!/bin/sh
#
php -f /opt/www/bitrix/modules/main/tools/backup.php; > /dev/null 2>&1
#
```

Сохраняем файл.
Делаем его исполняемым:
```bash
chmod -R a+x /etc/periodic/daily
```

Создаем файл `/etc/crontabs/cron.update`, чтобы `crond` демон перезапустил задания крона:
```bash
touch /etc/crontabs/cron.update
```

Итог: раз в день (в 2ч ночи) контейнер запустит `backup` задание и выполнит резервное копирование.

Документация: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=35&LESSON_ID=4464&LESSON_PATH=3906.4833.4464

<a id="bxtemporaryfilesdirectory"></a>
# Хранение временных файлов вне корневой директории сайта

По умолчанию конфигурация `nginx` подготовлена таким образом, чтобы временные файлы хранились вне пределах корневой директории проекта.

Чтобы окончательно заработало надо отредактировать файл `/bitrix/php_interface/dbconn.php`, добавить строку:
```bash
define("BX_TEMPORARY_FILES_DIRECTORY", "/opt/.bx_temp");
```

Cохранить.

Проверить настройку можно на странице `Сканер безопасности` (`/bitrix/admin/security_scanner.php?lang=ru`) модуля `Проактивной защиты (security)`.

<a id="pushpull"></a>
# Push & pull

<a id="pushserver"></a>
## Push сервер

Для работы push сервера будет запущено два контейнера: один для режима `pub`, второй для режима `sub`. Оба используют один образ `bx-push`.

Параметры переменных сред контейнеров хранятся в файлах `.env_push_pub` и `.env_push_sub`.

Секретный ключ сгенерирован предварительно и хранится в файле `.env_push`. По умолчанию его значение равно:
```bash
oqq2gaHWkogJHDfYY8CRzBaR9d26ZuCmTHIZa2egZ2Kk3IN3QKWDRB2Ixlt7usICbi1Qlvla3MylfqRrl1Cxhy9Af0nKDVH4GYWtE7FXbTZO8Kb9TUpysiDvPvMvTUy2
```

Возвращаемся к главам `Адресация` и `Порты` этого документа. Определяемся по какой схеме работает сайт. Например:
- используется локальный IP вида `10.0.1.119`
- порт для http `8588`
- порт для https `8589`

Итого получается:
- для http: `10.0.1.119:8588`
- для https: `10.0.1.119:8589`

Запоминаем эти значения.

В административной части продукта:
- редактируем файл `/bitrix/.settings.php`
- добавляем блок настроек, но меняем значения выше для http и https по примеру:
```bash
  'pull_s1' => 'BEGIN GENERATED PUSH SETTINGS. DON\'T DELETE COMMENT!!!!',
  'pull' => array(
    'value' => array(
      'path_to_listener' => 'http://10.0.1.119:8588/bitrix/sub/',
      'path_to_listener_secure' => 'https://10.0.1.119:8589/bitrix/sub/',
      'path_to_modern_listener' => 'http://10.0.1.119:8588/bitrix/sub/',
      'path_to_modern_listener_secure' => 'https://10.0.1.119:8589/bitrix/sub/',
      'path_to_mobile_listener' => 'http://#DOMAIN#:8893/bitrix/sub/',
      'path_to_mobile_listener_secure' => 'https://#DOMAIN#:8894/bitrix/sub/',
      'path_to_websocket' => 'ws://10.0.1.119:8588/bitrix/subws/',
      'path_to_websocket_secure' => 'wss://10.0.1.119:8589/bitrix/subws/',
      'path_to_publish' => 'http://10.0.1.119:8588/bitrix/pub/',
      'path_to_publish_web' => 'http://10.0.1.119:8588/bitrix/rest/',
      'path_to_publish_web_secure' => 'https://10.0.1.119:8589/bitrix/rest/',
      'path_to_json_rpc' => 'http://10.0.1.119:8588/bitrix/api/',
      'nginx_version' => '4',
      'nginx_command_per_hit' => '100',
      'nginx' => 'Y',
      'nginx_headers' => 'N',
      'push' => 'Y',
      'websocket' => 'Y',
      'signature_key' => 'oqq2gaHWkogJHDfYY8CRzBaR9d26ZuCmTHIZa2egZ2Kk3IN3QKWDRB2Ixlt7usICbi1Qlvla3MylfqRrl1Cxhy9Af0nKDVH4GYWtE7FXbTZO8Kb9TUpysiDvPvMvTUy2',
      'signature_algo' => 'sha1',
      'guest' => 'N',
    ),
  ),
  'pull_e1' => 'END GENERATED PUSH SETTINGS. DON\'T DELETE COMMENT!!!!',
```

Сохраняем файл.

Для продукта Битрикс24 проверить настройку push сервера можно на странице `Проверка системы` (`/bitrix/admin/site_checker.php?lang=ru`), используя таб `Работа портала`.

<a id="sphinx"></a>
# Sphinx

<a id="sphinxsearch"></a>
## Поиск с помощью Sphinx

Для работы полнотекстового поиска, используя `Sphinx`, будет запущен один контейнер на базе образа `bx-sphinx`.

Для запуска и настройки:
- переходим в настройки модуля `Поиск` (`/bitrix/admin/settings.php?lang=ru&mid=search`)
- на табе `Морфология`:
  - выбираем `Полнотекстовый поиск с помощью` - `Sphinx`
  - заполняем строку `Строка подключения для управления индексом (протокол MySql)` - `sphinx:3306`
  - поле `Идентификатор индекса` не меняем - `bitrix`

Сохраняем настройки модуля.

На странице `Переиндексация сайта` (`/bitrix/admin/search_reindex.php?lang=ru`) снимаем галку `Переиндексировать только измененные` и выполняем полную переиндексацию сайта.

Настройка завершена.

Для продукта Битрикс24 проверить работу можно в разделе `/search/` сайта.

Документация: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=35&CHAPTER_ID=04507&LESSON_PATH=3906.4507

Примечение: для продуктов, использующих базу данных PostgreSQL, нужно обновление модуля `search` версии `25.0.0`.

......ToDo......
