# Lesson 7

# Java Web Server: Инструкция по запуску как systemd-службы

## Описание

**Этот проект демонстрирует создание простого HTTP-сервера на Java** с использованием стандартной библиотеки (com.sun.net.httpserver.HttpServer), а также настройку его запуска как systemd-сервиса на Ubuntu.
Поддерживаются оба варианта сборки: Maven и Gradle[^1][^2].

## Шаги установки и запуска

### 1. Установка зависимостей

```bash
sudo apt update
sudo apt install default-jdk maven gradle -y
```


### 2. Создание проекта

**Maven:**

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=java-server -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
cd java-server
```

**Gradle:**

```bash
gradle init --type java-application
cd <имя-проекта>
```


### 3. Добавление кода веб-сервера

Замените содержимое файла:

- Maven: `src/main/java/com/example/App.java`
- Gradle: `src/main/java/App.java`

на следующий код:

```java
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;
import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;

public class App {
    public static void main(String[] args) throws Exception {
        int port = 8000;
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        server.createContext("/", new HttpHandler() {
            @Override
            public void handle(HttpExchange exchange) throws IOException {
                String response = "Hello from Java!";
                exchange.sendResponseHeaders(200, response.length());
                try (OutputStream os = exchange.getResponseBody()) {
                    os.write(response.getBytes());
                }
            }
        });

        server.start();
        System.out.println("Server running on port " + port);
    }
}
```


### 4. Сборка проекта

**Maven:**

```bash
mvn clean package
```

**Gradle:**

```bash
gradle build
```


### 5. Создание systemd-сервиса

Создайте файл `/etc/systemd/system/java-web.service`:

```ini
[Unit]
Description=Java Web Server

[Service]
# Для Maven
WorkingDirectory=/home/твой_пользователь/java-server
ExecStart=/usr/bin/java -jar target/java-server-1.0-SNAPSHOT.jar
# Для Gradle
#WorkingDirectory=/home/твой_пользователь/проект-gradle
#ExecStart=/usr/bin/java -jar build/libs/проект-gradle.jar
Restart=always
RestartSec=5
User=твой_пользователь

[Install]
WantedBy=multi-user.target
```

**Измените пути и имя пользователя под свою систему!**

### 6. Активация и запуск службы

```bash
sudo systemctl daemon-reload
sudo systemctl start java-web
sudo systemctl enable java-web
```


### 7. Проверка работы

```bash
curl http://localhost:8000
# Ожидаемый ответ: "Hello from Java!"
sudo systemctl status java-web
journalctl -u java-web -f
```


## Отладка и советы

- Проверить логи:
`journalctl -u java-web -e`
- Проверить статус сервиса:
`sudo systemctl status java-web`
- Проверить, что порт 8000 свободен:
`sudo lsof -i :8000`
- Проверить существование jar-файла:
`ls -l путь/к/jar-файлу`
- Проверить доступность сервера:
`curl -I http://localhost:8000`
- Открыть порт в firewall (если нужно):
`sudo ufw allow 8000`


## Чек-лист

- [ ] Проверить установку:
`java -version`, `mvn -v`, `gradle -v`
- [ ] Проверить сборку:
Maven: файл `target/*.jar`
Gradle: файл `build/libs/*.jar`
- [ ] Проверить работу службы:
`sudo systemctl is-active java-web`
- [ ] Проверить доступность порта:
`curl http://localhost:8000`



