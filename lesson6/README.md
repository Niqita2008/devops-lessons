# Lesson 6

# Автоматизация деплоя микросервисов с помощью Ansible

Этот проект предназначен для автоматизированного развертывания набора микросервисов (Python, Node.js, Go, Java) и Nginx-прокси на сервере с помощью Ansible.

## Структура проекта

```
.
├── group_vars/
│   └── all.yml               # Глобальные переменные для всех серверов
├── inventory.ini             # Инвентори-файл с описанием серверов
├── playbook.yml              # Главный Ansible playbook
├── roles/                    # Роли Ansible для каждого сервиса
│   ├── common/               # Базовая настройка (установка пакетов и т.д.)
│   ├── python_app/           # Flask-приложение
│   ├── node_app/             # Node.js-приложение
│   ├── go_app/               # Go-приложение
│   ├── java_app/             # Java-приложение
│   └── nginx_proxy/          # Nginx-прокси
└── templates/
    └── nginx/
        └── helloworld.j2     # Шаблон конфигурации Nginx
```


## Что разворачивается

- **common** — базовая настройка сервера, установка необходимых пакетов.
- **python_app** — деплой Flask-приложения, настройка systemd-сервиса.
- **node_app** — деплой Node.js-приложения, настройка systemd-сервиса.
- **go_app** — деплой Go-приложения, настройка systemd-сервиса.
- **java_app** — деплой Java-приложения (JAR), настройка systemd-сервиса.
- **nginx_proxy** — установка и настройка Nginx как обратного прокси для всех приложений.

Все сервисы запускаются как systemd-юниты, управляемые через Ansible.

## Требования

- **Ansible** (рекомендуется версия 2.9+)
- SSH-доступ к серверу (по ключу)
- Сервер с ОС Linux (Ubuntu, Debian и др.)


## Подготовка

1. **Настройте доступ по SSH**
Убедитесь, что ваш публичный ключ добавлен на сервер, и путь к приватному ключу указан в `inventory.ini`:

```
[staging_servers]
linux1 ansible_host=192.168.116.131 ansible_user=alina ansible_ssh_private_key_file=/home/alina2/.ssh/alina2
```

2. **Заполните переменные**
Отредактируйте `group_vars/all.yml` под свои нужды (например, порты, пути, переменные приложений).
3. **Проверьте роли**
В каждой роли есть файлы и шаблоны для systemd-юнитов, а также основные задачи (`tasks/main.yml`).

## Запуск playbook

1. **Проверьте подключение:**

```bash
ansible -i inventory.ini staging_servers -m ping
```

2. **Запустите деплой:**

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Ansible выполнит все роли по порядку: подготовит сервер, развернет приложения и настроит Nginx.

## Полезные команды

- **Перезапустить только Nginx:**

```bash
ansible -i inventory.ini staging_servers -m service -a "name=nginx state=restarted" --become
```

- **Перезапустить отдельное приложение:**

```bash
ansible-playbook -i inventory.ini playbook.yml --tags python_app
```

- **Посмотреть логи Ansible:**

```bash
tail -f /var/log/ansible.log
```


## Примечания

- Все сервисы управляются через systemd, шаблоны для юнитов находятся в папках `templates/systemd/` соответствующих ролей.
- Конфигурация Nginx генерируется из шаблона `templates/nginx/helloworld.j2`.
- Для продакшена рекомендуется использовать отдельные роли для безопасности и мониторинга.
- Если что-то не работает, проверьте логи Ansible и логи сервисов на сервере (`journalctl -u <service_name>`).

