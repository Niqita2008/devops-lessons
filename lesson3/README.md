# Lesson 3
Проект: Мульти-языковой HelloWorld с Nginx

## Описание

Этот проект демонстрирует работу нескольких простых веб-приложений, написанных на разных языках программирования (Python, Node.js, Go, Java Spring Boot), за прокси-сервером Nginx.
Каждое приложение отвечает на HTTP-запрос простым сообщением, а Nginx маршрутизирует запросы к соответствующему backend-сервису по разным URL-префиксам.

## Цель проекта

- **Показать, как можно объединить несколько сервисов на разных языках в одном проекте.**
- **Продемонстрировать настройку Nginx в качестве обратного прокси для различных backend-приложений.**
- **Обеспечить защищённый (SSL) доступ к сервисам через единый домен и разные пути.**


## Структура проекта

```
.
├── application.properties      # Конфиг для Spring Boot (Java)
├── DemoApplication.java        # Java Spring Boot приложение
├── go_app.go                  # Go-приложение
├── helloworld.dobs            # Конфиг Nginx для сайта
├── index.js                   # Node.js Express приложение
└── python_app.py              # Python Flask приложение
```


## Подробности реализации

### 1. Backend-приложения

#### Python (Flask)

- Файл: `python_app.py`
- Запускается на порту **8001**
- Возвращает: `Hello from Python Flask!`

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Python Flask!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8001)
```


#### Node.js (Express)

- Файл: `index.js`
- Запускается на порту **8002**
- Возвращает: `Hello from Node.js Express!`

```javascript
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from Node.js Express!'));
app.listen(8002);
```


#### Go

- Файл: `go_app.go`
- Запускается на порту **8003**
- Возвращает: `Hello from Go!`

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from Go!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8003", nil)
}
```


#### Java (Spring Boot)

- Файлы: `DemoApplication.java`, `application.properties`
- Запускается на порту **8004**
- Возвращает: `Hello from Java Spring Boot!`

**application.properties:**

```
spring.application.name=demo
server.port=8004
```

**DemoApplication.java:**

```java
package com.example.demo;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/")
    public String home() {
        return "Hello from Java Spring Boot!";
    }
}
```


### 2. Nginx-конфигурация

- Файл: `helloworld.dobs`
- Слушает порт **443** (SSL)
- Использует самоподписанный сертификат (`/etc/nginx/ssl/selfsigned.crt`)
- Включён gzip для ускорения передачи данных
- Проксирует запросы по разным путям к соответствующим backend-сервисам:

| URL-префикс | Проксируется на | Backend порт |
| :-- | :-- | :-- |
| `/python/` | http://127.0.0.1:8001/ | 8001 |
| `/node/` | http://127.0.0.1:8002/ | 8002 |
| `/go/` | http://127.0.0.1:8003/ | 8003 |
| `/java/` | http://127.0.0.1:8004/ | 8004 |

**Пример фрагмента конфига:**

```nginx
server {
    listen 443 ssl;
    server_name helloworld.dobs;
    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    access_log /var/log/nginx/helloworld_access.log;
    error_log /var/log/nginx/helloworld_error.log;

    location /python/ {
        proxy_pass http://127.0.0.1:8001/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /node/ {
        proxy_pass http://127.0.0.1:8002/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /go/ {
        proxy_pass http://127.0.0.1:8003/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /java/ {
        proxy_pass http://127.0.0.1:8004/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
## Как запустить

1. **Установите зависимости** для каждого backend-языка (Python, Node.js, Go, Java).
2. **Запустите все backend-приложения** на соответствующих портах.
3. **Настройте Nginx** с данным конфигом и убедитесь, что SSL-сертификаты присутствуют.
4. **Перейдите в браузере** по адресу:
    - `https://helloworld.dobs/python/` — Python Flask
    - `https://helloworld.dobs/node/` — Node.js Express
    - `https://helloworld.dobs/go/` — Go
    - `https://helloworld.dobs/java/` — Java Spring Boot

## Примечания

- Для работы SSL потребуется добавить самоподписанный сертификат или использовать свой.
- Все backend-приложения должны быть запущены на одной машине (localhost).



