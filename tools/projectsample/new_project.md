# Новый проект (django, docker, postgres, nginx)

Инструкция основана на [этой](https://www.pluralsight.com/guides/packaging-a-django-app-using-docker-nginx-and-gunicorn)
## Сделать джанго приложение (или копировать шаблон)

В директории лежит sampleDjangoApp, можно копировать его, но чтобы были красивые имена лучше создать свое.
## Упаковать приложение

Все файлы докера и настройки nginx уже лежат в шаблоне. Полностью копировать папку nginx, в конфиге поменять названия. Копировать файлы докера, .env, .gitignore, init.sql, entrypoint.sh, requirements.txt.

Поправить пути и названия в docker-compose.yml (строчки 9, 18, 21, 24 (название такое же как у проекта джанги))

В докерфайле (который не в nginx) добавить 4ой строчкой

    ENV APP_USER=root_user

Почему это так разберемся потом.

После первого запуска убрать 

    python manage.py collectstatic &&

из docker-compose.yml.

Переименовать tmp.gitignore в .gitignore
## Если не копировал приложение джанго а создал свое

settings.py:

    ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS').split(" ")

    DATABASES = {
        'default': {
            'ENGINE': os.environ.get('ENGINE'),
            'NAME': os.environ.get('DB_NAME'),
            'USER': os.environ.get('POSTGRES_USER'),
            'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
            'HOST': os.environ.get('DB_HOST'),
            'PORT': os.environ.get('DB_PORT'),
        }
    }

    STATIC_ROOT = os.path.join(BASE_DIR, "static")

Если нужно будет запустить без докера под виртуальной средой, то после import os:

    from dotenv import load_dotenv


    load_dotenv()

## После первого запуска

Убрать строчку

    python manage.py collectstatic &&

в docker-compose.yml. Пока что это не создавало проблем, но должно

## Если не выполняются команды внутри контейнера джанги

    docker exec -it <container_name> <command>

Это скорее всего для python команд.

## Возможно когда-нибудь я сделаю скрипт который принимает параметры и сам создает все это, но это маловероятно