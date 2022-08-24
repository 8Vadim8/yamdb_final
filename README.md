![](https://github.com/8Vadim8/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)
# Проект: CI и CD для «API для YaMDb»
 Реализация Continuous Integration и Continuous Deployment для проекта API для YaMDb.
 ## Что сделано:
 - автоматический запуск тестов
 - обновление образа на Docker Hub
 - автоматический деплой на боевой сервер при пуше в главную ветку на Githube
 - отправка сообщения в Telegram через бота об успешном деплое

## Стэк
  - Python 3.7
  - Django 2.2
  - DRF
  - Simple-JWT
  - PostgreSQL
  - Docker
  - Nginx
  - Gunicorn
  - GitHub Actions
---

## Запуск приложения в Docker-контейнерах

### Сначала нужно клонировать репозиторий , а затем перейти в папку infra и создать в ней файл .env с переменными окружения, необходимыми для работы приложения:
```
git clone git@github.com:8Vadim8/yamdb_final.git
cd yamdb_final/infra
nano .env
```
Или

```
echo "SECRET_KEY=YourSecretKey 
DB_ENGINE=django.db.backends.postgresql 
DB_NAME=postgres 
POSTGRES_USER=postgres 
POSTGRES_PASSWORD=postgres 
DB_HOST=db DB_PORT=5432" > .env
```
Пример содержимого файла .env:
```
SECRET_KEY=key                          # секретный ключ Джанго-приложения
DB_ENGINE=django.db.backends.postgresql # тип базы данных
DB_NAME=postgres                        # название БД(свое)
POSTGRES_USER=postgres                  # логин для подключения к БД(свой)
POSTGRES_PASSWORD=postgres              # пароль от БД(свой)
DB_HOST=db                              # название сервиса(контейнера)
DB_PORT=5432                            # порт для подключения

```
Секретный ключ Джанго можно сгенерировать [здесь](https://djecrety.ir), либо:
```
from django.core.management.utils import get_random_secret_key
get_random_secret_key()
```
Далее следует запустить docker-compose:
```
docker-compose up -d --build
```
Будут созданы и запущены в фоновом режиме необходимые для работы приложения контейнеры (db, web, nginx).

Затем нужно внутри контейнера web выполнить миграции, создать суперпользователя и собрать статику:
```
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-input 
```
После этого проект должен быть доступен по адресу http://localhost/

### Заполнение базы данных
Нужно зайти на на http://localhost/admin/, авторизоваться и внести записи в базу данных через админку.

Резервную копию базы данных можно создать командой
```
docker-compose exec web python manage.py dumpdata > fixtures.json 
```
Загрузка из копии базы данных
```
docker-compose exec web python manage.py loaddata fixtures.json 
```
### Остановка контейнеров
Остановка контейнеров
```
docker-compose stop 
```
Также можно запустить контейнеры без их создания заново командой
```
docker-compose start 
```
Остановка контейнеров с последующим их удалением
```
docker-compose down -v
```
### Документация в формате Redoc:
Чтобы посмотреть документацию API в формате Redoc, нужно запустить проект и перейти на страницу http://localhost/redoc/

***
### Над проектом работал:

- _[Вадим Евлентьев](https://github.com/8Vadim8)_
