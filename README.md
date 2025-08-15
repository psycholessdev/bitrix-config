# Контейнерное окружение Битрикс + отдельное PHP окружения для сайта

Этот репозиторий является быстрым способом развернуть битрикс и отдельное PHP окружения для вашего сайта.
Они будут работать на разных доменах, например `yourdomain.com` и `bitrix.yourdomain.com`

Структура следующая:
У нас есть основной `Nginx` сервер - он принимает ВСЕ запросы и перенаправляет их соответственно.
Для домена `yourdomain.com` он будет раздавать статику. Если же это PHP файл, Nginx отдаст его в `php-fpm` контейнер для выполнения, а затем возвращает результат.
Для домена `bitrix.yourdomain.com` Nginx перенаправляет все запросы на Btrix контейнер.

> [!Что такое Btrix контейнер]
> Вообще "Btrix" контейнера не существует.
> На самом деле он состоит из 12 контейнеров, а "Btrix контейнером" я называю второй Nginx который является точной взаимодействия с ним

Файлы Bitrix будут находится в `bitrix/www_data`

## Как запускать проект?

Создайте сеть `bitrix-expose`:
```bash
docker network create bitrix-expose
```


# Базовая проверка конфигурации окружения

После запуска сайта необходимо провести базовую проверку конфигурации веб-сервера. Она выполняется с помощью скрипта `bitrix_server_test.php`.

Используем способ, который работает одинаково на всех ОС.

Заходим в sh-консоль контейнера `php` из-под пользователя `bitrix`:
```bash
docker compose exec --user=bitrix php sh
```

Переходим в корневой каталог сайта и скачиваем скрипт `bitrix_server_test.php`:
```bash
cd /opt/www/
wget https://dev.1c-bitrix.ru/download/scripts/bitrix_server_test.php
```

В браузере переходим по ссылке вида:
```bash
https://bitrix.yourdomain.com/bitrix_server_test.php
```

> [!CAUTION]
> Внимание! После проверки конфигурации окружения скрипт `bitrix_server_test.php` нужно удалить.

Для Docker Engine на Linux расположение каталога сайта на хосте зависит от режима работы docker-а:
- rootfull:
```bash
/var/lib/docker/volumes/dev_www_data/_data/
```

- rootless:
```bash
/home/[USERNAME]/.local/share/docker/volumes/dev_www_data/_data
```

# BitrixSetup / Restore

Для установки продуктов компании 1С-Битрикс можно использовать скрипт `bitrixsetup.php`: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&LESSON_ID=4523&LESSON_PATH=10495.4495.4523

Для восстановления из резервной копии скрипт `restore.php`: https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=02014&LESSON_PATH=10495.4496.2014

Заходим в sh-консоль контейнера `php` из-под пользователя `bitrix`:
```bash
docker compose exec --user=bitrix php sh
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
http://bitrix.yourdomain.com/bitrixsetup.php
```

- для восстановления:
```bash
http://bitrix.yourdomain.com/restore.php
```

Устанавливаем продукт или восстанавливаем сайт, зависит от вашего выбора.

## Установка дистрибутивов

Используем скрипт `bitrixsetup.php`, скачиваем дистрибутив продукта компании 1С-Битрикс, демо или лицензионную версию.

Редакция и тип базы зависит от вашего выбора.

После переходим на сайт, используя URL:
```bash
http://bitrix.yourdomain.com/
```

Проходим мастер установки дистрибутива:
- `1С-Битрикс: Управление сайтом` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522
- `1С-Битрикс24` - https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=135&CHAPTER_ID=04522&LESSON_PATH=10495.4495.4522

На шаге создания базы вводим параметры подключения к ней:

- для `MySQL` версии:
  - Имя хоста (алиас) - `mysql`
  - Имя суперпользователя - `root`
  - Пароль суперпользователя - был создан вами в главе [Пароли к базам данных MySQL и PostgreSQL](#databasespasswords), хранится в файле `.env_sql`

- для `PostgreSQL` версии:
  - Имя хоста (алиас) - `postgres`
  - Имя суперпользователя - `postgres`
  - Пароль суперпользователя - был создан вами в главе [Пароли к базам данных MySQL и PostgreSQL](#databasespasswords), хранится в файле `.env_sql`
