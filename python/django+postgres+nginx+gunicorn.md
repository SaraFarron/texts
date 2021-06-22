# Создание проекта с django, postgres и nginx (gunicorn)

## Установка пакетов

    sudo apt update
    sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

## PostgreSQL (долгий метод)

    sudo -u postgres psql

Попадаем в консоль постреса, там создаем базу данных, суперюзера, настройки и прочее

    CREATE DATABASE myproject;
    CREATE USER myprojectuser WITH PASSWORD 'password';
    ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
    ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
    ALTER ROLE myprojectuser SET timezone TO 'UTC';
    GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;

## Виртуальная среда проекта

    sudo -H pip3 install --upgrade pip
    sudo -H pip3 install virtualenv

    mkdir ~/myprojectdir
    cd ~/myprojectdir

    virtualenv myprojectenv
    source myprojectenv/bin/activate

    pip install django gunicorn psycopg2-binary

## Создание и настройка django

    django-admin.py startproject myproject ~/myprojectdir

Сейчас каталог проекта должен содержать следующее:

+ ~/myprojectdir/manage.py: скрипт управления проектами Django.
+ ~/myprojectdir/myproject/: здесь должны содержаться файлы __init__.py, settings.py, urls.py и wsgi.py.
+ ~/myprojectdir/myprojectenv/: виртуальный каталог.

Изменение настроек проекта

    nano ~/myprojectdir/myproject/settings.py

Добавить домены в ALLOWED_HOSTS:

    ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP', . . ., 'localhost']

Настроить DATABASES:

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'myproject',
            'USER': 'myprojectuser',
            'PASSWORD': 'password',
            'HOST': 'localhost',
            'PORT': '',
        }
    }

Добавить STATIC_ROOT

    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

Миграции, суперюзер, собрать статик

    ~/myprojectdir/manage.py makemigrations
    ~/myprojectdir/manage.py migrate
    ~/myprojectdir/manage.py createsuperuser
    ~/myprojectdir/manage.py collectstatic

Статичные файлы будут помещены в каталог static

Создать исключение для порта

    sudo ufw allow 8000

Можно протестировать

    ~/myprojectdir/manage.py runserver 0.0.0.0:8000

перейдя на http://server_domain_or_IP:8000 должна появиться страница джанги по умолчанию

## Gunicorn

    cd ~/myprojectdir
    gunicorn --bind 0.0.0.0:8000 myproject.wsgi

Можно запустить приложение и проверить работу, должна быть та же страница по тому же адресу. Если есть - значит gunicorn пашет

Выход из виртуальной среды

    deactivate

## Создание файлов сокета и служебных файлов systemd для Gunicorn

    sudo nano /etc/systemd/system/gunicorn.socket

Внутри файла:

    [Unit]
    Description=gunicorn socket

    [Socket]
    ListenStream=/run/gunicorn.sock

    [Install]
    WantedBy=sockets.target

Служебный файл для джуникорна

    sudo nano /etc/systemd/system/gunicorn.service

Внутри файла:

    [Unit]
    Description=gunicorn daemon
    Requires=gunicorn.socket
    After=network.target

    [Service]
    User=sammy
    Group=www-data
    WorkingDirectory=/home/sammy/myprojectdir
    ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
            --access-logfile - \
            --workers 3 \
            --bind unix:/run/gunicorn.sock \
            myproject.wsgi:application

    [Install]
    WantedBy=multi-user.target

Запуск и активация джуникорна

    sudo systemctl start gunicorn.socket
    sudo systemctl enable gunicorn.socket

Проверка состояния сокета

    sudo systemctl status gunicorn.socket

Проверка наличия сока "джуникорн":

    file /run/gunicorn.sock

Если команда systemctl status указывает на ошибку, или если в каталоге отсутствует файл gunicorn.sock, это означает, что сокет Gunicorn не удалось создать. Можно проверить журналы сокета Gunicorn с помощью следующей команды:

    sudo journalctl -u gunicorn.socket

## Тестирование активации сокета

Если запустить только gunicorn.socket, служба gunicorn.service не будет активна в связи с отсутствием подключений к сокету:

    sudo systemctl status gunicorn 

    Output
    ● gunicorn.service - gunicorn daemon
    Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
    Active: inactive (dead)

Протестировать механизм активации сокета:

    curl --unix-socket /run/gunicorn.sock localhost

Выводимые данные приложения должны отобразиться в терминале в формате HTML. Это показывает, что Gunicorn запущен и может обслуживать приложение Django. Если результат вывода curl или systemctl status указывают на наличие проблемы, причины лажи будут (наверное) в журнале:

    sudo journalctl -u gunicorn

Можно проверить файл /etc/systemd/system/gunicorn.service на наличие проблем. Если внесены изменения в файл /etc/systemd/system/gunicorn.service, нужен перезапуск демона, чтобы заново считать определение службы, и перезапустить процесс Gunicorn:

    sudo systemctl daemon-reload
    sudo systemctl restart gunicorn

## Настройка Nginx как прокси для Gunicorn

Нужно создать и открыть новый серверный блок в каталоге Nginx sites-available:

    sudo nano /etc/nginx/sites-available/myproject

Внутри файла:

    server {
        listen 80;
        server_name server_domain_or_IP;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /home/sammy/myprojectdir;
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
    }

    sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled

Нджинкcовский тест н наличие ошибок в конфиге

    sudo nginx -t

Если ошибок не будет найдено:

    sudo systemctl restart nginx

Нужна возможность открыть брандмауэр для обычного трафика через порт 80. Поскольку больше не потребуется доступ к серверу разработки, можно удалить правило и открыть порт 8000:

    sudo ufw delete allow 8000
    sudo ufw allow 'Nginx Full'

Теперь должна быть возможность перейти к домену или IP-адресу вашего сервера для просмотра вашего приложения.

<span style="color:yellow">
Примечание. После настройки Nginx необходимо защитить трафик на сервер с помощью SSL/TLS. Это важно, поскольку в противном случае вся информация, включая пароли, будет отправляться через сеть в простом текстовом формате.
Если имеется доменное имя, проще всего будет использовать Let’s Encrypt для получения сертификата SSL для защиты вашего трафика.

[Это](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04) руководство поможет настроить Let’s Encrypt с Nginx в Ubuntu 18.04.
Если у вас нет доменного имени, вы можете защитить свой сайт для тестирования и обучения с помощью сертификата SSL с собственной подписью.

<span style="color:white">

# Диагностика и устранение неисправностей Nginx и Gunicorn

## Nginx показывает страницу по умолчанию, а не приложение Django

Это обычно означает, что нужно изменить параметр server_name в файле /etc/nginx/sites-available/myproject, чтобы он указывал на IP-адрес или доменное имя сервера.

Nginx использует server_name, чтобы определять, какой серверный блок использовать для ответа на запросы. Если выводит страницу Nginx по умолчанию, это значит, что Nginx не может найти явное соответствие запросу в серверном блоке и выводит блок по умолчанию, заданный в /etc/nginx/sites-available/default.

## Nginx выводит ошибку 502 Bad Gateway вместо приложения Django

502 означает, что Nginx не может выступать в качестве прокси для запроса. Ошибка 502 может сигнализировать о разнообразных проблемах конфигурации.

В первую очередь идем в журналы ошибок Nginx с помощью следующей команды:

    sudo tail -F /var/log/nginx/error.log

### connect() to unix:/run/gunicorn.sock failed (2: No such file or directory)

Нужно сравнить расположение proxy_pass, определенное в файле etc/nginx/sites-available/myproject, с фактическим расположением файла gunicorn.sock, сгенерированным блоком systemd gunicorn.socket. Если файла gunicorn.sock в каталоге /run нету, то идем 
[сюда](Создание-файлов-сокета-и-служебных-файлов-systemd-для-Gunicorn) или 
[сюда](Тестирование-активации-сокета)

### connect() to unix:/run/gunicorn.sock failed (13: Permission denied)

Nginx не удалось подключиться к сокету Gunicorn из-за проблем с правами доступа. Это может произойти, если процедуру выполнять с привилегиями root, а не с привилегиями sudo. Хотя systemd может создать файл сокета Gunicorn, Nginx не может получить к нему доступ.

Это может произойти из-за ограничения прав доступа в любом месте между корневым каталогом (/) и файлом gunicorn.sock. Увидеть права доступа и владельцев файла сокета и всех его родительских каталогов:

    namei -l /run/gunicorn.sock

Если для любого из каталогов, ведущих к сокету, отсутствуют глобальные разрешения на чтение и исполнение, Nginx не сможет получить доступ к сокету.


## Django выводит ошибку: «could not connect to server: Connection refused»

    OperationalError at /admin/login/
    could not connect to server: Connection refused
        Is the server running on host "localhost" (127.0.0.1) and accepting
        TCP/IP connections on port 5432?

Django не может подключиться к базе данных Postgres. Убедиться в нормальной работе экземпляра Postgres:

    sudo systemctl status postgresql

Если он работает некорректно, возможный фикс:

    sudo systemctl start postgresql
    sudo systemctl enable postgresql

Если проблемы не исчезнут, правильность настроек базы данных (~/myprojectdir/myproject/settings.py) под вопросом. Настройка 
[тут](Создание-и-настройка-django)

## Дополнительная диагностика и устранение неисправностей

Следующие журналы могут быть полезными:
| Журнал | Команда |
|:------------- |:-------------|
| процессов Nginx |      sudo journalctl -u nginx |
| доступа Nginx |        sudo less /var/log/nginx/access.log |
| ошибок Nginx |         sudo less /var/log/nginx/error.log |
| приложения Gunicorn |  sudo journalctl -u gunicorn |
| сокета Gunicorn |      sudo journalctl -u gunicorn.socket |

При обновлении конфигурации или приложения может понадобиться перезапустить процессы для адаптации к изменениям.

Перезапустить процесс Gunicorn для адаптации к изменениям в django:

    sudo systemctl restart gunicorn

Если изменить файл сокета или служебные файлы Gunicorn, то требуется перезагрузка демона и  процесса:

    sudo systemctl daemon-reload
    sudo systemctl restart gunicorn.socket gunicorn.service

Если изменить конфигурацию серверного блока Nginx, следует протестирировать конфигурацию и Nginx с перезагрузкой последнего:

    sudo nginx -t && sudo systemctl restart nginx
