# Lesson 9

# Notes Microservices Project

## Описание

Этот проект — пример микросервисной архитектуры для работы с заметками (*notes*), реализованный на двух языках программирования: **Go** и **Python (Flask)**.
Оба сервиса используют одну базу данных PostgreSQL и доступны через единый HTTPS reverse proxy на базе **nginx**.

Проект демонстрирует:

- организацию REST API для CRUD-операций с заметками,
- работу с PostgreSQL из Go и Python,
- проксирование запросов через nginx с поддержкой HTTPS (self-signed сертификат),
- контейнеризацию и удобный запуск с помощью Docker Compose.


## Структура проекта

```
my_project/
│
├── docker-compose.yml        # Docker Compose для запуска всех сервисов
├── go/                      # Go-сервис (REST API)
│   └── main.go
├── python/                  # Python-сервис (Flask REST API)
│   ├── app.py
│   └── requirements.txt
├── nginx/                   # Конфигурация nginx и SSL-сертификаты
│   ├── nginx.conf
│   └── ssl/
│       ├── selfsigned.crt
│       └── selfsigned.key
└── vnv/                     # Виртуальное окружение Python (не рекомендуется коммитить в репозиторий)
```


## Как это работает

- **nginx** принимает HTTPS-запросы на 443 порт, проксирует их на соответствующие микросервисы:
    - `/go/notes` → Go-сервис (порт 5002)
    - `/python/notes` → Python-сервис (порт 5001)
- Оба сервиса работают с одной таблицей `notes` в базе PostgreSQL.
- Реализованы методы:
    - `POST /notes` — создать заметку
    - `GET /notes` — получить список всех заметок


## Быстрый старт

### 1. Клонируйте репозиторий

```bash
git clone https://github.com/your_username/your_repo.git
cd your_repo
```

### 3. Примеры запросов

**Создать заметку (Go или Python сервис):**

```bash
curl -k -X POST https://notes.dobs/go/notes -H "Content-Type: application/json" -d '{"title":"Test","content":"Hello Go!"}'
curl -k -X POST https://notes.dobs/python/notes -H "Content-Type: application/json" -d '{"title":"Test","content":"Hello Python!"}'
```

**Получить все заметки:**

```bash
curl -k https://notes.dobs/go/notes
curl -k https://notes.dobs/python/notes
```

> `-k` используется для игнорирования self-signed SSL сертификата.

## Сервисы

### Go-сервис

- Находится в папке `go/`
- Использует стандартную библиотеку Go и драйвер `lib/pq` для PostgreSQL
- REST API:
    - `GET /notes`
    - `POST /notes`


### Python-сервис

- Находится в папке `python/`
- Использует Flask и psycopg2
- REST API:
    - `GET /notes`
    - `POST /notes`
- Для локальной разработки:

```bash
cd python
python3 -m venv vnv
source vnv/bin/activate
pip install -r requirements.txt
python app.py
```


### nginx

- Конфигурация в папке `nginx/`
- Использует self-signed сертификат для HTTPS
- Проксирует запросы к Go и Python сервисам


## Переменные окружения и настройки

- Все параметры подключения к БД заданы в коде (для продакшена рекомендуется использовать переменные окружения).
- База данных:
    - host: `localhost`
    - dbname: `notes_db`
    - user: `notes_user`
    - password: `mystrongpassword`


## Требования

- Docker и Docker Compose
или
- Python 3.10+, Go 1.20+, PostgreSQL 13+, nginx

