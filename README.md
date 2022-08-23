![example workflow status](https://github.com/github/docs/actions/workflows/main.yml/badge.svg)
## Техническое описание проекта API YaMDb.
Проект YaMDb собирает отзывы (Review) пользователей на произведения (Titles).

Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список категорий (Category) может быть расширен администратором.

Произведению может быть присвоен жанр (Genre) из списка предустановленных (например, «Сказка», «Рок» или «Артхаус»). Новые жанры может создавать только администратор.

Благодарные или возмущённые пользователи оставляют к произведениям текстовые отзывы (Review) и ставят произведению оценку в диапазоне от одного до десяти (целое число); из пользовательских оценок формируется усреднённая оценка произведения — рейтинг (целое число). На одно произведение пользователь может оставить только один отзыв.

Отзыв может быть прокомментирован (Сomment) пользователями.

- ### Пользовательские роли

  - Аноним — может просматривать описания произведений, читать отзывы и комментарии.
  - Аутентифицированный пользователь (user) — может читать всё, как и Аноним, может публиковать отзывы и ставить оценки произведениям (фильмам/книгам/песенкам), может комментировать отзывы; может редактировать и удалять свои отзывы и комментарии, редактировать свои оценки произведений. Эта роль присваивается по умолчанию каждому новому пользователю.
  - Модератор (moderator) — те же права, что и у Аутентифицированного пользователя, плюс право удалять и редактировать любые отзывы и комментарии.
  - Администратор (admin) — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.
  - Суперюзер Django должен всегда обладать правами администратора, пользователя с правами admin. Даже если изменить пользовательскую роль суперюзера — это не лишит его прав администратора. Суперюзер — всегда администратор, но администратор — не обязательно суперюзер.

- ### Самостоятельная регистрация новых пользователей

  - Пользователь отправляет POST-запрос с параметрами email и username на эндпоинт /api/v1/auth/signup/.
  - Сервис YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на указанный адрес email.
  - Пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт /api/v1/auth/token/, в ответе на запрос ему приходит token (JWT-токен).
  - В результате пользователь получает токен и может работать с API проекта, отправляя этот токен с каждым запросом.
  - После регистрации и получения токена пользователь может отправить PATCH-запрос на эндпоинт /api/v1/users/me/ и заполнить поля в своём профайле (описание полей — в документации).

- ### Создание пользователя администратором

  - Пользователя может создать администратор — через админ-зону сайта или через POST-запрос на специальный эндпоинт api/v1/users/ (описание полей запроса для этого случая — в документации). В этот момент письмо с кодом подтверждения пользователю отправлять не нужно.
  - После этого пользователь должен самостоятельно отправить свой email и username на эндпоинт /api/v1/auth/signup/ , в ответ ему должно прийти письмо с кодом подтверждения.
  - Далее пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт /api/v1/auth/token/, в ответе на запрос ему приходит token (JWT-токен), как и при самостоятельной регистрации.

- ### Ресурсы API YaMDb

  - Ресурс auth: аутентификация.
  - Ресурс users: пользователи.
  - Ресурс titles: произведения, к которым пишут отзывы (определённый фильм, книга или песенка).
  - Ресурс categories: категории (типы) произведений («Фильмы», «Книги», «Музыка»).
  - Ресурс genres: жанры произведений. Одно произведение может быть привязано к нескольким жанрам.
  - Ресурс reviews: отзывы на произведения. Отзыв привязан к определённому произведению.
  - Ресурс comments: комментарии к отзывам. Комментарий привязан к определённому отзыву.
- ### Стэк
  -Python 3.7, Django, DRF, Simple-JWT, PostgreSQL, Docker, nginx, gunicorn.
---

# Запуск приложения в Docker-контейнерах

### Сначала нужно клонировать репозиторий и перейти в корневую папку:
```
git clone git@github.com:8Vadim8/infra_sp2.git
cd infra_sp2
```
Затем нужно перейти в папку infra и создать в ней файл .env с переменными окружения, необходимыми для работы приложения.

```
cd infra/
nano .env
```
Пример содержимого файла .env:
```
SECRET_KEY=key
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432

```
Далее следует запустить docker-compose:
```
docker-compose up -d
```
Будут созданы и запущены в фоновом режиме необходимые для работы приложения контейнеры (db, web, nginx).

Затем нужно внутри контейнера web выполнить миграции, создать суперпользователя и собрать статику:
```
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
Для остановки работы приложения можно набрать в терминале команду Ctrl+C либо открыть второй терминал и воспользоваться командой
```
docker-compose stop 
```
Также можно запустить контейнеры без их создания заново командой
```
docker-compose start 
```
### Документация в формате Redoc:
Чтобы посмотреть документацию API в формате Redoc, нужно запустить проект и перейти на страницу http://localhost:8000/redoc/

***
### Над проектом работал:

- _[Вадим Евлентьев](https://github.com/8Vadim8)_
