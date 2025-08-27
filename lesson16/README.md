# Lesson 16

# Notes \& Tasks Microservices Project

Проект демонстрирует микросервисную архитектуру с использованием RabbitMQ как брокера сообщений, MongoDB как хранилища данных, двумя producer-сервисами на Flask (Python) и Go, и соответствующими воркерами-обработчиками сообщений. Запросы от клиентов проксируются через Nginx.

## Структура проекта

```
my_project/
├── app/                  # Python сервис и воркер
│   ├── app.py            # Flask API: producer задач
│   ├── worker.py         # Python воркер: consumer
│   ├── Dockerfile
│   └── requirements.txt
├── goapp/                # Go сервис и воркер
│   ├── main.go           # Go API: producer задач
│   ├── worker.go         # Go воркер: consumer
│   ├── Dockerfile
│   ├── go.mod
│   └── go.sum
├── nginx/                # Nginx прокси
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml    # Compose конфигурация для всех сервисов
├── .env                  # Переменные окружения
└── README.md             # Этот файл
```


## Используемые технологии

- **Flask** — producer на Python
- **Go (Gin)** — producer на Go
- **RabbitMQ** — брокер сообщений, очередь задач
- **MongoDB** — хранение задач после обработки
- **Nginx** — прокси для маршрутизации на разные сервисы
- **Docker \& Docker Compose** — управление контейнерами


## Переменные окружения `.env`

```ini
# Общие параметры
RABBIT_HOST=rabbitmq
RABBIT_PORT=5672
RABBIT_QUEUE=task_queue
MONGO_URI=mongodb://mongo:27017/
DB_NAME=mydb

# Порты для сервисов
FLASK_PORT=5001
GO_PORT=5002
```


## Запуск проекта

1. Клонируйте репозиторий и перейдите в корень проекта.
2. Убедитесь, что Docker и Docker Compose установлены и работают.
3. Соберите и запустите стек:
```bash
docker-compose build --no-cache
docker-compose up -d
```

4. Проверьте, что запущены все контейнеры:
```bash
docker-compose ps
```

Должны быть активны сервисы: `mongo`, `rabbitmq`, `flask`, `pyworker`, `goapp`, `goworker`, `nginx`.

## Работа с API

### Flask API

- Добавить задачу (producer):

```bash
curl -X POST http://localhost/flask/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Test task from Flask"}'
```

- Получить список обработанных задач:

```bash
curl http://localhost/flask/tasks
```


### Go API

- Добавить задачу:

```bash
curl -X POST http://localhost/go/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Test task from Go"}'
```

- Получить список задач:

```bash
curl http://localhost/go/tasks
```


## RabbitMQ Management UI

Доступен по адресу:

```
http://localhost:15672/
```

- Логин/пароль: `guest` / `guest`
- Позволяет просматривать очереди, сообщения и состояние брокера.


## Архитектура и взаимодействие компонентов

- **Producer (Flask/Go)** при POST /tasks публикует задачу в RabbitMQ очередь `task_queue`.
- **Worker (python/go)** слушает очередь, принимает сообщения, обрабатывает (добавляет мета-данные), сохраняет задачу в MongoDB.
- **Nginx** маршрутизирует запросы:
    - `/flask/` → Flask на порт 5001
    - `/go/` → Go-сервис на порт 5002
    - `/rabbitmq/` → RabbitMQ Management UI на порт 15672


## Ключевые файлы

- `app/app.py` — Flask producer API
- `app/worker.py` — Python воркер
- `goapp/main.go` — Go producer API (Gin)
- `goapp/worker.go` — Go воркер
- `nginx/nginx.conf` — прокси конфигурация Nginx
- `docker-compose.yml` — описание всех сервисов


## Полезные команды

- Посмотреть логи:

```bash
docker-compose logs flask
docker-compose logs pyworker
docker-compose logs goapp
docker-compose logs goworker
docker-compose logs nginx
```

- Остановить и удалить контейнеры:

```bash
docker-compose down
```

- Остановить с удалением орфанных контейнеров:

```bash
docker-compose down --remove-orphans
```

