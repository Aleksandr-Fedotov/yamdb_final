![example workflow](https://github.com/Aleksandr-Fedotov/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

# Сервис YaMDb

Проект YaMDb собирает отзывы пользователей на произведения. 
Произведения делятся на категории: «Книги», «Фильмы», «Музыка». 
Список категорий может быть расширен администратором.



## Технологии

- Django rest_framework
- Django rest_framework_simplejwt
- Django django_filters
- Git
- Docker
- NGINX


## Начало работы

Для начала клонируйте репозиторий и перейдите в него выполнив команды в командной строке:

```sh
git clone https://github.com/Aleksandr-Fedotov/infra_sp2
cd infra_sp2/
```


## Шаблон наполнения .env-файла

Чтобы настроить переменные окружения для работы с базой данных пройдите из корневой папки проекта до соответствующего файла `/infra/.env`:

```sh
DB_NAME=<postgres>
# имя базы данных
POSTGRES_USER=<user>
# логин для подключения к базе данных
POSTGRES_PASSWORD=<password>
# пароль для подключения к БД
DB_HOST=<db>
# название сервиса (контейнера)
DB_PORT=<5432>
# порт для подключения к БД 
```


## Как запустить проект:

Перейти из корневой директории в `infra/` и запустить **docker-compose**:

```sh
cd infra/
docker-compose up -d --build
```

В контейнере **web** выполнить миграции и собрать статику:

```sh
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py collectstatic --no-input
```

Для создания суперпользователя:

```sh
[winpty] docker-compose exec web python manage.py createsuperuser
```

После запуска контейнеров можно ознакомится с документацией по проекту, в которой описана основная логика приложения, документация доступна по ссылке: http://localhost/redoc/


## Как загрузить данные в проект:

Выполнить команды в консоле внутри контейнера `web`:

```sh
docker exec <container_ID> python csv2json.py
docker exec <container_ID> python manage.py loaddata /reviews/fixtures/fixtures.json
```

## Описание проекта

### Пользовательские роли
 - Аноним — может просматривать описания произведений, читать отзывы и комментарии;
 - Аутентифицированный пользователь (user) — может читать всё, как и Аноним, может публиковать отзывы и ставить оценки произведениям (фильмам/книгам/песенкам), может комментировать отзывы; может редактировать и удалять свои отзывы и комментарии, редактировать свои оценки произведений. Эта роль присваивается по умолчанию каждому новому пользователю;
 - Модератор (moderator) — те же права, что и у Аутентифицированного пользователя, плюс право удалять и редактировать любые отзывы и комментарии;
 - Администратор (admin) — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям;
 - Суперюзер Django должен всегда обладать правами администратора, пользователя с правами admin. Даже если изменить пользовательскую роль суперюзера — это не лишит его прав администратора. Суперюзер — всегда администратор, но администратор — не обязательно суперюзер.

### Алгоритм регистрации пользователей
 - Пользователь отправляет POST-запрос с параметрами email и username на эндпоинт /api/v1/auth/signup/;
 - Сервис YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на указанный адрес email;
 - Пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт /api/v1/auth/token/, в ответе на запрос ему приходит token (JWT-токен);
 - В результате пользователь получает токен и может работать с API проекта, отправляя этот токен с каждым запросом;
 - После регистрации и получения токена пользователь может отправить PATCH-запрос на эндпоинт /api/v1/users/me/ и заполнить поля в своём профайле (описание полей — в документации по ссылке: http://localhost/redoc/).
 
### Создание пользователя администратором
 - Пользователя может создать администратор — через админ-зону сайта или через POST-запрос на специальный эндпоинт api/v1/users/ (описание полей запроса для этого случая — в документации);
 - В этот момент письмо с кодом подтверждения пользователю отправлять не нужно.
После этого пользователь должен самостоятельно отправить свой email и username на эндпоинт /api/v1/auth/signup/ , в ответ ему должно прийти письмо с кодом подтверждения;
 - Далее пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт /api/v1/auth/token/, в ответе на запрос ему приходит token (JWT-токен), как и при самостоятельной регистрации.


 ## Ресурсы сервиса API YaMDb
- auth: аутентификация;
- users: пользователи;
- titles: произведения, к которым пишут отзывы (определённый фильм, книга или песенка);
- categories: категории (типы) произведений («Фильмы», «Книги», «Музыка»);
- genres: жанры произведений. Одно произведение может быть привязано к нескольким жанрам;
- reviews: отзывы на произведения. Отзыв привязан к определённому произведению;
- comments: комментарии к отзывам. Комментарий привязан к определённому отзыву.


## Примеры запросов

### GET-запросы

***GET /api/v1/genres/***

***ответ***
```json
{
    "count": 4,
    "next": null,
    "previous": null,
    "results": [
        {
            "name": "Драма",
            "slug": "drama"
        },
        {
            "name": "Комедия",
            "slug": "comedy"
        },
        {
            "name": "Вестерн",
            "slug": "western"
        },
        {
            "name": "Фэнтези",
            "slug": "fantasy"
        }
    ]
}
```

***GET /api/v1/titles/1/reviews/***

***ответ***
```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "title": "Побег из Шоушенка",
            "text": "Ставлю десять звёзд!\n...Эти голоса были чище и светлее тех, о которых мечтали в этом сером, убогом месте. Как будто две птички влетели и своими голосами развеяли стены наших клеток, и на короткий миг каждый человек в Шоушенке почувствовал себя свободным.",
            "author": "bingobongo",
            "score": 10,
            "pub_date": "2019-09-24T21:08:21.567000Z"
        },
        {
            "id": 2,
            "title": "Побег из Шоушенка",
            "text": "Не привыкай\n«Эти стены имеют одно свойство: сначала ты их ненавидишь, потом привыкаешь, а потом не можешь без них жить»",
            "author": "capt_obvious",
            "score": 10,
            "pub_date": "2019-09-24T21:08:21.567000Z"
        }
    ]
}
```

### POST-запросы

***POST /api/v1/categories/***
```json
{
    "name": "Фотография",
    "slug": "photo"
}
```

***ответ***
```json
{
    "name": "Фотография",
    "slug": "photo"
}
```

### PATCH-запросы

***PATCH /api/v1/titles/2/***
```json
{
    "description": "Эпическая гангстерская драма режиссёра Фрэнсиса Форда Копполы. Экранизация одноимённого романа Марио Пьюзо, изданного в 1969 году. Слоган: «Предложение, от которого невозможно отказаться»."
}
```

***ответ***
```json
{
    "id": 2,
    "name": "Крестный отец",
    "year": 1972,
    "rating": 4,
    "description": "Эпическая гангстерская драма режиссёра Фрэнсиса Форда Копполы. Экранизация одноимённого романа Марио Пьюзо, изданного в 1969 году. Слоган: «Предложение, от которого невозможно отказаться».",
    "genre": [
        {
            "name": "Драма",
            "slug": "drama"
        }
    ],
    "category": {
        "name": "Фильм",
        "slug": "movie"
    }
}
```

## License

### Подготовлено командой разработчиков:

**Проект API YaMDb:**
- *evgenlit*
- *Aleksandr-Fedotov*
- *Margarita-pyth*

**Контейнеризация:**
- *Aleksandr-Fedotov*