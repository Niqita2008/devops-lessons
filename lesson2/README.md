# Lesson 2
## Описание

В этом проекте реализованы простые веб-приложения на Python (Flask), Java (Spring Boot), Node.js (Express) и Go. Также представлены два конфигурационных файла NGINX для проксирования запросов к этим приложениям.

## Структура проекта

```
.
├── app.py                # Python Flask приложение
├── DemoApplication.java  # Java Spring Boot приложение
├── index.js              # Node.js Express приложение
├── main.go               # Go HTTP сервер
├── site3.dops            # NGINX конфиг для site3.dops
├── site4.dops            # NGINX конфиг для site4.dops
```


## Запуск приложений

### Python (Flask)

```bash
pip install flask
python3 app.py
```

Доступно на: http://localhost:8001

### Java (Spring Boot)

Соберите и запустите приложение:

```bash
# Сборка (если используете Maven)
mvn clean package

# Запуск
java -jar target/demo-*.jar
```

Доступно на: http://localhost:8080

### Node.js (Express)

```bash
npm install express
node index.js
```

Доступно на: http://localhost:8002

### Go

```bash
go run main.go
```

Доступно на: http://localhost:8003

## Конфигурация NGINX

### site3.dops

- Проксирует все запросы к `/` на backend (например, Flask, Express, Go).
- Обрабатывает статику из `/var/www/` по пути `/static/`.
- Использует SSL (self-signed сертификаты).


### site4.dops

- Проксирует запросы к `/dops/` на другой backend.
- Также использует SSL.

**Пример запуска NGINX:**

```bash
sudo nginx -c /path/to/site3.dops
sudo nginx -c /path/to/site4.dops
```
## Исходный код приложений

### Python (Flask) — `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Python Flask!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8001)
```


### Java (Spring Boot) — `DemoApplication.java`

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


### Node.js (Express) — `index.js`

```javascript
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from Node.js Express!'));
app.listen(8002);
```


### Go — `main.go`

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


## Конфигурация NGINX

### site3.dops

```nginx
server {
    listen 443 ssl;
    server_name site3.dops;
    ssl_certificate     /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    gzip on;
    gzip_types text/plain application/json application/javascript text/css;
    access_log /var/log/nginx/dops_access.log;
    error_log  /var/log/nginx/dops_error.log;

    location / {
        proxy_pass http://backend_dops1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        root /var/www/;
        expires 1h;
        add_header Cache-Control "public";
    }
}
```

### site4.dops

```nginx
server {
    listen 443 ssl;
    server_name site4.dops;
    ssl_certificate     /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    gzip on;
    gzip_types text/plain application/json application/javascript text/css;
    access_log /var/log/nginx/dops_access.log;
    error_log  /var/log/nginx/dops_error.log;

    location /dops/ {
        proxy_pass http://backend_dops2;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Запуск приложений

### Python (Flask)

```bash
pip install flask
python3 app.py
```

Доступно на: http://localhost:8001

### Java (Spring Boot)

```bash
# Сборка (если используете Maven)
mvn clean package

# Запуск
java -jar target/demo-*.jar
```

Доступно на: http://localhost:8080

### Node.js (Express)

```bash
npm install express
node index.js
```

Доступно на: http://localhost:8002

### Go

```bash
go run main.go
```

Доступно на: http://localhost:8003

## Примеры ответов

- **Flask:** `Hello from Python Flask!`
- **Spring Boot:** `Hello from Java Spring Boot!`
- **Express:** `Hello from Node.js Express!`
- **Go:** `Hello from Go!`



