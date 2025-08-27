# Lesson 12
**Мульти-контейнерный проект на Docker Compose:**
REST API на Go с хранением заметок в PostgreSQL и задач в MongoDB.

## Состав проекта

- **Go backend** — REST API для заметок и задач
- **PostgreSQL** — хранение заметок
- **MongoDB** — хранение задач


## Быстрый старт

### 1. Клонируйте репозиторий и перейдите в папку проекта

```bash
git clone <ваш-репозиторий>
cd <ваш-репозиторий>/nginx-configs/lesson12
```


### 2. Создайте необходимые файлы окружения

Файл `.env` уже включён в проект. Проверьте, что переменные корректны:

```env
# Postgres
POSTGRES_HOST=postgres
POSTGRES_DB=notes_db
POSTGRES_USER=notes_user
POSTGRES_PASSWORD=mystrongpassword

# MongoDB
MONGO_URI=mongodb://mongo_user:mongopass@mongo:27017/
MONGO_DB=tasks_db
```


### 3. Запустите сервисы через Docker Compose

```bash
docker-compose up --build
```

- Будут подняты три контейнера: backend (Go), PostgreSQL и MongoDB.
- Порты:
    - Go API: [http://localhost:5002](http://localhost:5002)
    - PostgreSQL: 5432
    - MongoDB: 27017


### 4. Инициализация БД

**PostgreSQL:**
При первом запуске вручную создайте таблицу `notes`, если она отсутствует:

```bash
docker exec -it <postgres-container-name> psql -U notes_user -d notes_db
```

Внутри psql:

```sql
CREATE TABLE notes (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
\q
```


### 5. Использование API

#### Заметки (PostgreSQL)

- Получить все заметки:

```
GET /notes
```

- Создать заметку:

```
POST /notes
Content-Type: application/json
{
  "title": "Заголовок",
  "content": "Текст заметки"
}
```


#### Задачи (MongoDB)

- Получить все задачи:

```
GET /tasks
```

- Создать задачу:

```
POST /tasks
Content-Type: application/json
{
  "title": "Задача",
  "description": "Описание"
}
```


### 6. Примеры запросов

```bash
curl -X POST http://localhost:5002/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Note","content":"Hello Postgres"}'

curl http://localhost:5002/notes

curl -X POST http://localhost:5002/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy milk","description":"From MongoDB"}'

curl http://localhost:5002/tasks
```


## Структура проекта

```
lesson12/
├── docker-compose.yml
├── .env
├── .gitignore
└── go/
    ├── main.go
    ├── go.mod
    └── go.sum
```


## Зависимости

- Go 1.22.2
- PostgreSQL 13
- MongoDB 6
- Docker, Docker Compose


## Примечания

- При первом запуске убедитесь, что таблица `notes` создана вручную.
- Для работы с задачами MongoDB дополнительная инициализация не требуется.
- Все переменные окружения настраиваются через `.env`.

