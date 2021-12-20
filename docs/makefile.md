## Автоматизация процессов с makefile ##

**Запуск приложения**
```bash
make up
```
**Запуск приложения с xDebug**
```bash
make up-x
```
**Запуск приложения с тестовой БД и применением сидов**
```bash
make init
```
**Запуск тестов и генерация swagger.json**
```bash
make test
```
**Запуск полного цикла тестирования:**
* применение/откат миграций
* применение сидов
* запуск unit тестов
* генерация swagger.json
```bash
make test-full
```
**Удаление таблиц БД и применение миграций**
```bash
make fresh
```
**Запуск coverage**
```bash
make coverage
```
**Применение сидов на очищенную БД**
```bash
make seed
```
**Применение сидов на тестовую БД**
```bash
make seed-test
```
**Применение сидов на основную и тестовую БД**
```bash
make seed-all
```
**Генерация swagger.json**
```bash
make swagger
```
**Запуск tinker**
```bash
tinker
```
