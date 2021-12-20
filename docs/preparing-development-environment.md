## Установка проекта ##

```bash
cp .env.example .env
docker-compose up -d --build
docker-compose exec app composer install
docker-compose exec app artisan migrate
docker-compose exec app artisan ide-helper:generate #чтобы в ide не подсвечивало классы
docker-compose exec app php artisan db:seed #создаст пользователя
docker-compose exec app php artisan jwt:secret #генерируем секретный ключ
curl localhost:8090 # проверка, отдаст json
```
Запуск тестов:
```bash
docker-compose exec app phpunit
```
Рестарт с удалением контейнеров, ребилд и вывод логов на консоль:
```bash
docker-compose down && docker-compose up -d --build && docker-compose logs -f
```
### Swagger
```bash
# Генерация swagger.json
docker run -v "$PWD":/app -it gudik/swagger-php:3.0.1 -e vendor -o deployment/swagger/swagger.json app
```
После чего свагер будет доступен по адресу:
[http://localhost:8093/](http://localhost:8093/)

Для работы с апи нужен токен. Если вы не сделали этого раньше, то сначало генерируем секретный ключ:
```
docker-compose exec app php artisan jwt:secret
```
Сгенерировать годовой токен для админа:
```bash
docker-compose exec app artisan jwt:admin_token
```
После передаем токен в апи в заголовке "Authorization"

### GIT
Если делали изменения в докер образах, то необходимо добавить комментарий к коммиту: `[ci: buildBaseImages]` в этом случае ci перебилдит докер образ.

Если вносились изменения в файл composer.* (устанавливали пакет, обновляли) то добавляем к коммиту коментарий `[ci:buildvendors]` тут ci пересоберет вендоров.
