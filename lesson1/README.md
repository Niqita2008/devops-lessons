# Lesson 1
## Описание

В этом проекте реализована **настройка NGINX как обратного прокси** для двух сайтов с раздельной маршрутизацией запросов, использованием блоков `upstream`, раздельными логами и обслуживанием статики.
Сайты:

- **site1.dops** — проксирует все запросы на один backend и отдаёт статику по `/static/`
- **site2.dops** — проксирует запросы на другой backend по пути `/dops/`


## Структура проекта

```
.
├── index.html         # Пример статического файла
├── site1.dops         # Конфиг для первого сайта
├── site2.dops         # Конфиг для второго сайта
└── upstreams.conf     # Конфиги upstream-блоков
```


## Как это работает

- **site1.dops**
    - Все запросы на `/` проксируются на backend по адресу 127.0.0.1:8001
    - Запросы к `/static/` обслуживаются из директории `/var/www/`
    - Логи: `/var/log/nginx/dops_access.log`, `/var/log/nginx/dops_error.log`
- **site2.dops**
    - Запросы на `/dops/` проксируются на backend по адресу 127.0.0.1:8002
    - Логи: те же, что и для первого сайта
- **upstream-блоки**
    - Описывают адреса backend-серверов (см. важное замечание ниже)


## Быстрый старт

1. **Установите NGINX** (если не установлен):

```bash
sudo apt update
sudo apt install nginx
```

2. **Запустите backend-серверы:**

```bash
python3 -m http.server 8001 --bind 127.0.0.1
mkdir -p ~/backend_dops/dops
echo "Hello world!" > ~/backend_dops/dops/index.html
cd ~/backend_dops && python3 -m http.server 8002 --bind 127.0.0.1

```
3. **Разместите конфиги** в нужной директории (например, `/etc/nginx/sites-available/`).
4. **Активируйте сайты:**

```bash
sudo ln -s /etc/nginx/sites-available/site1.dops /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site2.dops /etc/nginx/sites-enabled/
```

5. **Создайте директорию для статики и пример файла:**

```bash
sudo mkdir -p /var/www/static
echo "Hello world!" | sudo tee /var/www/static/index.html
```

6. **Пропишите хосты** в `/etc/hosts`:

```
127.0.0.1 site1.dops
127.0.0.1 site2.dops
```

7. **Проверьте конфиги и перезапустите NGINX:**

```bash
sudo nginx -t && sudo systemctl reload nginx
```


## Пример конфигураций

**site1.dops**

```nginx
server {
    listen 80;
    server_name site1.dops;
    access_log /var/log/nginx/dops_access.log;
    error_log /var/log/nginx/dops_error.log;

    location / {
        proxy_pass http://backend_site1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        root /var/www/;
    }
}
```

**site2.dops**

```nginx
server {
    listen 80;
    server_name site2.dops;
    access_log /var/log/nginx/dops_access.log;
    error_log /var/log/nginx/dops_error.log;

    location /dops/ {
        proxy_pass http://backend_site2/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**upstreams.conf**

```nginx
upstream backend_site1 {
    server 127.0.0.1:8001;
}

upstream backend_site2 {
    server 127.0.0.1:8002;
}
```


