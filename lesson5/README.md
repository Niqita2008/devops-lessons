# Lesson 5
```plaintext
Обратный прокси NGINX для 4 Hello World приложений с использованием Docker и Docker Compose
├── docker-compose.yaml
├── go_app/
│   ├── Dockerfile
│   └── main.go
├── java_app/
│   ├── DemoApplication.java
│   └── Dockerfile
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── ssl/
│       ├── selfsigned.crt
│       └── selfsigned.key
├── node_app/
│   ├── Dockerfile
│   └── index.js
└── python_app/
    ├── app.py
    └── Dockerfile
```
## Описание сервисов

- **python_app** — Приложение на Python (порт 8001)
- **node_app** — Приложение на Node.js (порт 8002)
- **go_app** — Приложение на Go (порт 8003)
- **java_app** — Приложение на Java (порт 8004)
- **nginx** — Прокси-сервер, который принимает внешние HTTPS-запросы и перенаправляет их к нужному приложению

Все приложения объединены в одну сеть `app_network`, чтобы они могли общаться друг с другом по внутренним именам контейнеров.

## Требования

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)


## Запуск проекта

1. **Клонируйте репозиторий:**

```bash
git clone <URL-репозитория>
cd lesson5
```

2. **Проверьте наличие всех файлов, особенно сертификатов в `nginx/ssl/`.**
Если сертификаты отсутствуют, создайте самоподписанные:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/selfsigned.key \
  -out nginx/ssl/selfsigned.crt \
  -subj "/CN=localhost"
```

3. **Запустите проект:**

```bash
docker compose up --build
```

4. **Откройте в браузере:**
    - [https://localhost](https://localhost)
(Браузер может предупредить о самоподписанном сертификате — это нормально для учебных целей.)

## Как это работает

- Каждый сервис собирается из своего Dockerfile и запускается в отдельном контейнере.
- Внутри сети `app_network` сервисы доступны друг другу по именам: `python_app`, `node_app`, `go_app`, `java_app`.
- Nginx слушает порт 443 (HTTPS) и проксирует запросы к соответствующим приложениям.
- Внешний доступ открыт только к Nginx.


## Полезные команды

- **Остановить все контейнеры:**

```bash
docker compose down
```

- **Посмотреть логи:**

```bash
docker compose logs -f
```

- **Пересобрать контейнеры:**

```bash
docker compose up --build
```


## Примечания

- Для работы HTTPS используется самоподписанный сертификат (`nginx/ssl/selfsigned.crt` и `selfsigned.key`).
Не используйте его в продакшене!
- Если вы хотите добавить переменные окружения — создайте файлы `.env` внутри папок приложений и подключите их в Dockerfile или docker-compose.yaml.
- Все внутренние порты приложений открыты только внутри сети Docker (`expose`), снаружи доступен только Nginx.

