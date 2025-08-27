# Lesson 4

Проект с Docker Compose

Этот проект демонстрирует, как развернуть несколько приложений (Python, Node.js, Go, Java) и проксировать их через Nginx с помощью Docker Compose. Все сервисы изолированы, используют свои тома для данных и логов, а также общий секрет.

## Структура проекта

```
.
├── data/                   # Общая директория для данных (не используется напрямую в docker-compose)
├── docker-compose.yml      # Главный файл конфигурации Docker Compose
├── go_app/                 # Go-приложение
│   ├── Dockerfile
│   └── main.go
├── java_app/               # Java-приложение
│   ├── DemoApplication.java
│   └── Dockerfile
├── nginx/                  # Конфигурация и Dockerfile для Nginx
│   ├── Dockerfile
│   ├── nginx.conf
│   └── ssl/
│       ├── selfsigned.crt  # Самоподписанный сертификат для HTTPS
│       └── selfsigned.key
├── node_app/               # Node.js-приложение
│   ├── Dockerfile
│   └── index.js
├── python_app/             # Python-приложение
│   ├── app.py
│   └── Dockerfile
└── secrets/
    └── app_secret.txt      # Секрет, используемый всеми приложениями
```


## Описание сервисов

- **python_app** — Приложение на Python (порт 8001)
- **node_app** — Приложение на Node.js (порт 8002)
- **go_app** — Приложение на Go (порт 8003)
- **java_app** — Приложение на Java (порт 8004)
- **nginx** — Веб-сервер, который проксирует запросы к каждому приложению и обслуживает HTTPS

Каждое приложение:

- Собирается из Dockerfile в своей папке
- Использует переменные окружения из `.env`
- Получает секрет из `secrets/app_secret.txt`
- Логи и данные хранятся в отдельных томах


## Требования

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)


## Запуск проекта

1. **Клонируйте репозиторий:**

```bash
git clone <URL-репозитория>
cd project-root
```

2. **Создайте файл секрета:**

```bash
echo "your_secret_value" > secrets/app_secret.txt
```

3. **Создайте необходимые файлы .env для каждого приложения**
Пример для Python:

```
# python_app/.env
DEBUG=True
PORT=8001
```

Аналогично для других приложений (`node_app/.env`, `go_app/.env`, `java_app/.env`).
4. **Запустите проект:**

```bash
docker-compose up --build
```

5. **Проверьте работу сервисов:**
    - Откройте [http://localhost](http://localhost) или [https://localhost](https://localhost) в браузере.
    - Nginx будет проксировать запросы к соответствующим приложениям.

## Переменные окружения

- **UID/GID** — идентификаторы пользователя/группы, чтобы контейнеры работали от вашего имени (для корректных прав на файлы).
- **.env** — для каждого приложения свои переменные окружения.


## Работа с секретами

Секреты хранятся в папке `secrets/` (например, `app_secret.txt`).
Docker Compose автоматически монтирует этот файл внутрь контейнеров.

## Полезные команды

- **Остановить проект:**

```bash
docker-compose down
```

- **Посмотреть логи:**

```bash
docker-compose logs -f
```

- **Пересобрать контейнеры:**

```bash
docker-compose up --build
```


## Примечания

- Для HTTPS используется самоподписанный сертификат (`nginx/ssl/selfsigned.crt`).
Браузер может ругаться на него — это нормально для тестовой среды.
- Все данные и логи приложений сохраняются в Docker-томах, чтобы не терялись при пересоздании контейнеров.
- Если вы запускаете на Linux, убедитесь, что ваш пользователь входит в группу docker.


