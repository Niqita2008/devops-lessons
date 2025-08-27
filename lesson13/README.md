# Lesson 13

# Notes \& Tasks Microservices Project

## Описание

Этот проект реализует два микросервиса для работы с заметками и задачами:

- **/notes** — хранение и получение заметок в PostgreSQL.
- **/tasks** — хранение и получение задач в MongoDB.

Вся работа ведётся через HTTPS-прокси nginx с self-signed сертификатом. Каждый микросервис реализован на отдельном языке (Python/Flask и Go) и запускается в Docker-контейнере.

## Структура проекта

```
my-project/
├── docker-compose.yml
├── go/
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   └── main.go
├── nginx/
│   ├── nginx.conf
│   └── ssl/
│       ├── selfsigned.crt
│       └── selfsigned.key
└── python/
    ├── app.py
    ├── Dockerfile
    └── requirements.txt
```


## Быстрый старт

### 1. Клонируйте репозиторий и перейдите в папку проекта

```bash
git clone <repo_url>
cd my-project
```


### 2. Создайте файл `.env`

Пример содержимого:

```
# Postgres
POSTGRES_HOST=postgres
POSTGRES_DB=notes_db
POSTGRES_USER=notes_user
POSTGRES_PASSWORD=mystrongpassword

# MongoDB
MONGO_URI=mongodb://mongo_user:mongopass@mongo:27017/
MONGO_DB=tasks_db
```

### 3. Соберите и запустите все сервисы

```bash
docker-compose up --build
```

Дождитесь, когда все сервисы будут здоровы (`healthy`) и nginx начнет слушать порт 443.

## Эндпоинты

### Python (Flask) микросервис

- **POST /python/notes**
Создать заметку (PostgreSQL)

```bash
curl -k -X POST https://localhost/python/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Note","content":"Hello Postgres"}'
```

- **GET /python/notes**
Получить все заметки

```bash
curl -k https://localhost/python/notes
```

- **POST /python/tasks**
Создать задачу (MongoDB)

```bash
curl -k -X POST https://localhost/python/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy milk","description":"From MongoDB"}'
```

- **GET /python/tasks**
Получить все задачи

```bash
curl -k https://localhost/python/tasks
```


### Go микросервис

- **POST /go/notes**
Создать заметку (PostgreSQL)

```bash
curl -k -X POST https://localhost/go/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"Go Note","content":"Go Postgres"}'
```

- **GET /go/notes**
Получить все заметки

```bash
curl -k https://localhost/go/notes
```

- **POST /go/tasks**
Создать задачу (MongoDB)

```bash
curl -k -X POST https://localhost/go/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Go Task","description":"Go Mongo"}'
```

- **GET /go/tasks**
Получить все задачи

```bash
curl -k https://localhost/go/tasks
```


> Используйте опцию `-k` для curl, так как используется self-signed сертификат.

## Схемы данных

### PostgreSQL (notes)

```sql
CREATE TABLE notes (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


### MongoDB (tasks)

```json
{
  "_id": ObjectId,
  "title": "string",
  "description": "string",
  "created_at": ISODate
}
```


## Конфигурация nginx

- Все запросы на `/python/` проксируются на Flask (порт 5001).
- Все запросы на `/go/` проксируются на Go (порт 5002).
- Сертификаты хранятся в `nginx/ssl/`.


## Переменные окружения

- Указываются в `.env` (см. пример выше).
- Не коммитятся в репозиторий.


## Проверка работы и отладка

- Логи сервисов:

```bash
docker-compose logs flask
docker-compose logs go
```

- Проверить состояние контейнеров:

```bash
docker ps
```

- Остановить сервисы:

```bash
docker-compose down
```


## Примечания

- Для работы с БД внутри контейнеров используйте команды:
    - PostgreSQL:

```bash
docker exec -it <postgres_container> psql -U notes_user -d notes_db
```

    - MongoDB:

```bash
docker exec -it <mongo_container> mongosh -u mongo_user -p mongopass --authenticationDatabase admin
use tasks_db
db.tasks.find().pretty()
```


## Требования

- Docker и docker-compose должны быть установлены.
- Порты 443, 5001, 5002, 5432, 27017 должны быть свободны.



