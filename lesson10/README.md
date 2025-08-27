# Lesson 10

# Notes App with Go, Python, PostgreSQL, Nginx и Docker Compose

Это проект многосервисного приложения для создания и просмотра заметок (notes), написанного на Go и Python. Сервис использует базу данных PostgreSQL и проксируется через Nginx с поддержкой HTTPS. Все запускается с помощью `docker-compose`.

## Структура проекта

```
.
├── docker-compose.yml          # Конфигурация Docker Compose
├── go                         # Приложение на Go
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   └── main.go                # Основной код Go API
├── nginx                      # Nginx и SSL конфигурация
│   ├── Dockerfile
│   └── nginx.conf
├── python                     # Приложение на Python (Flask)
│   ├── Dockerfile
│   ├── notes_app.py
│   └── requirements.txt
```


## Как это работает

- **postgres** — контейнер с базой данных PostgreSQL;
- **python-app** — Flask API на Python, работающий на порту 5001;
- **go-app** — API на Go, работающий на порту 5002;
- **nginx** — обратный прокси, который маршрутизирует запросы и выполняет SSL termination.

Nginx проксирует запросы:

- `/python/notes` → Python API (`python-app:5001/notes`)
- `/go/notes` → Go API (`go-app:5002/notes`)


## Требования

- Docker
- Docker Compose


## Запуск проекта

1. Склонируйте или скачайте проект к себе.
2. Перейдите в папку с проектом, где лежит `docker-compose.yml`.
3. Запустите команду для сборки и старта контейнеров:
```bash
docker-compose up --build
```

4. Подождите, пока контейнеры запустятся. Логи покажут сообщения вроде `Server started at :5001` и `Server started at :5002`.

## Тестирование API

### Через Python API (порт 5001):

- Создать заметку:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"title":"Test","content":"Hello from Python"}' http://localhost/python/notes
```

- Получить список заметок:

```bash
curl http://localhost/python/notes
```


### Через Go API (порт 5002):

- Создать заметку:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"title":"Test","content":"Hello from Go"}' http://localhost/go/notes
```

- Получить список заметок:

```bash
curl http://localhost/go/notes
```


## HTTPS и SSL

Nginx настроен слушать 443 порт с самоподписанным сертификатом.

- Файлы сертификата лежат в `nginx/ssl/` (скопированы в контейнер).
- Сертификат используется для HTTPS.

Если хотите протестировать HTTPS, добавьте в `/etc/hosts` запись:

```
127.0.0.1 notes.dobs
```

И откройте в браузере:

```
https://notes.dobs/python/notes
https://notes.dobs/go/notes
```


## Остановить проект

```bash
docker-compose down
```



