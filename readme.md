[![pipeline status](https://git.pik.ru/tech-control-services/issues/badges/master/pipeline.svg)](https://git.pik.ru/tech-control-services/issues/commits/master)
[![coverage report](https://git.pik.ru/tech-control-services/issues/badges/master/coverage.svg)](https://git.pik.ru/tech-control-services/issues/commits/master)

### Для разработчиков:
Нужно установить `docker-compose`. [Инструкция по установке на Linux Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/#prerequisites).

Если возникла ошибка 
```bash
ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.
```
то выполните `bash usermod -aG docker ${USER}` и перезагрузите компьютер.

##### Автоматизация запуска приложения
Необходимо установить утилиту `make`. 

[Список доступных инструкций makefile](docs/makefile.md)

##### Разворачивание проекта
Клонируем репозиторий и выполняем команду:
```bash
cp .env.example .env
```
В файле .env для миграции из MDS прописываем данные для авторизации:
```
MDS_USER=main\<login>
MDS_PASSWORD=<password>
```
Для переопределения окружения, например для включения xDebug, можно использовать пример override файла docker-compose.override.example.yml:
```bash
cp docker-compose.override.example.yml docker-compose.override.yml
```
Выполняем команды. Можно одной строкой:
```bash
docker-compose up -d --build && docker-compose exec app composer install && docker-compose exec app artisan migrate && docker-compose exec app artisan ide-helper:generate && docker-compose exec app php artisan jwt:secret && docker-compose restart
```
Или по порядку:
```bash
docker-compose up -d --build
docker-compose exec app composer install
docker-compose exec app artisan migrate
docker-compose exec app artisan ide-helper:generate #чтобы в ide не подсвечивало классы
docker-compose exec app php artisan jwt:secret #генерируем секретный ключ для jwt
docker-compose restart
curl localhost:8109 #проверка, что проект работает, должен вернуть json
```
Всё, проект развернут. Далее можно заполнить бд тестовыми данными:
```bash
make seed
```
Заполнить только справочники (справочники в разработке):
```bash
make seed-dict
```
Импорт объектов с заполнением rooms тестовыми данными у квартир
```bash
docker-compose exec app php -d memory_limit=-1 -d xdebug.remote_enable=0 artisan import:objects --testrooms
```
Запуск тестов:
```bash
docker-compose exec app phpunit
```
Рестарт с удалением контенеров, ребилд и вывод логов на консоль
```bash
docker-compose down && docker-compose up -d --build && docker-compose logs -f
```
### Swagger
```bash
# Свагер обновляется при пересборке контейнера
docker-compose up -d --build swagger
# или
make swagger
```
После чего свагер будет доступен по адресу:
[http://localhost:8111/](http://localhost:8111/)

Для работы с апи нужен токен. Если вы не сделали этого раньше, то генерируем секретный ключ для jwt:
```
docker-compose exec app php artisan jwt:secret
```
Сгенерировать годовой токен для админа:
```bash
docker-compose exec app artisan jwt:admin_token
```
После передаем токен в апи в заголовке "Authorization"

Полезные ссылки по сваггеру
* [https://github.com/zircote/swagger-php/blob/master/docs/Getting-started.md](https://github.com/zircote/swagger-php/blob/master/docs/Getting-started.md)
* [https://github.com/DarkaOnLine/L5-Swagger/blob/master/tests/storage/annotations/OpenApi/Anotations.php](https://github.com/DarkaOnLine/L5-Swagger/blob/master/tests/storage/annotations/OpenApi/Anotations.php)
* [https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md)

## Beanstalkd
Пример команды запуска воркера один раз для нескольких очередей import_object_rooms и default
`artisan queue:work --once --queue=import_object_rooms,default`
#### Список очередей для job
* import_object_rooms
* status_notifications
* issue_changes_to_crm
* interrogate_app
* export_encounter_status
* create_issue_in_crm
* update_issue_in_crm
* import_bds_elements
* put_company_roles_to_mds
* delete_company_roles_from_mds
* fill_prescription_google_doc
* masstransit_publish
* masstransit_consume
* notify_issue
* check_result
* export_issues_to_xls

## MassTransit и RabbitMQ 
Краткая документация по MassTransit и RabbitMQ [тут](docs/masstransit.md)

### Стенды
Информация по стендам [тут](docs/stands.md)

### Тесты - phpunit
Для ускорения тестов в CI/CD все feature тесты разделены на логические группы, тесты в CI/CD запускаются параллельно по 
каждой группе, плюс отдельно запускаются тесты из папки Unit.

Для того, чтобы поместить тест в группу, над названием класса теста указывается аннотация `@group groupName`.

Список групп: Issue, CheckList, User, Company, Object, OT, Other, default


## Настройка конфигураций в `.env`
#### Загрузки файлов
По умолчанию используется Amazon Web Services. Для его работы в `.env` надо заполнить
```dotenv
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=
AWS_BUCKET=
AWS_ENDPOINT=
```
#### Импорт данных со старого проекта Технадзора
```dotenv
DB_BACK_CONNECTION=
DB_BACK_HOST=
DB_BACK_PORT=
DB_BACK_DATABASE=
DB_BACK_USERNAME=
DB_BACK_PASSWORD=
```
#### Письма
```dotenv
PIK_DELIVERY_DOMAIN=
PIK_DELIVERY_TOKEN=
MAIL_FROM_ADDRESS=
MAIL_FROM_NAME=
MAIL_TO_TEST=
MAIL_PROJECT_NAME=
```
#### Для отправки и получения сообщений из RabbitMQ
```dotenv
RABBITMQ_HOST=
RABBITMQ_PORT=
RABBITMQ_USER=
RABBITMQ_PASSWORD=
RABBITMQ_VHOST=
```
#### Импорт данных из МДС
```dotenv
MDS_USER=
MDS_PASSWORD=
```
#### Пуш уведомления
```dotenv
GOOGLEAPIS_KEY=
```
#### Адрес фронт приложения
Используется для генерации ссылок из нашего приложения на фронт
```dotenv
APP_URL=
```

### Для прода
Создать БД для сервиса:
```bash
gcloud sql databases create objects --instance microservices-prod
```

### Настройка авторизации
В .env должны заполнены JWT_PUBLIC_KEY


JWT_PUBLIC_KEY генерится на сайте https://mkjwk.org, Key size - 2048, Key use - Signing, Algorithm - RS256, Key ID - 
берется из kid из ключа сервиса, в котором происходит авторизация, с этого сайта после генерации берется 
"Keypair Contains both public and private keys."

Для получения токена авторизации в нашей системе нужен code, его можно получить тут
`<хост внешнего сервиса авторизации>/connect/authorize?response_type=code&redirect_uri=https://tn.pik.ru/login&client_id=tn_service_spa&scope=openid+email+offline_access+tn_service_api`
Переходите по ссылке и берете из GET параметров code. Далее его используете в методе `GET /users/get_token?code=<код>`

### Первый запуск пайплайна

При первом запуске пайплайна необходимо закомментировать в `.gitlab-ci.yml` стороки вида:

```yaml
only:
    changes:
      - deployment/baseImages/*
```

Это нужно для создания базовых образов и образов с зависимостями

### Миграция/переиспользование пайплайна

Названия базовых образов захардкожены в Dockerfile для локальной сборки.
При миграции образов-артефактов в другой Docker Registry, необходимо поменять названия базовых образов в следующих файлах:

* Dockerfile
* vendors.Dockerfile
