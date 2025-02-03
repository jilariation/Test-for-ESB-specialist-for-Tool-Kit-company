# Order API на MuleSoft

Этот проект представляет собой REST API для обработки заказов, разработанное на платформе MuleSoft Anypoint Platform.
API позволяет выполнять операции CRUD с заказами, используя PostgreSQL для хранения данных и ActiveMQ для обработки сообщений.

## Возможности API

- **POST /orders** — создание нового заказа
- **GET /orders/{id}** — получение информации о заказе
- **PUT /orders/{id}/status** — обновление статуса заказа с отправкой сообщения в ActiveMQ
- **DELETE /orders/{id}** — удаление заказа

## Требования

- **MuleSoft Anypoint Studio 7.13+**
- **JDK 8+**
- **Maven**
- **Docker** (если запускается в контейнерах)
- **PostgreSQL 13+**
- **ActiveMQ**

## Поднятие проекта

### 1. Запуск в Docker

Используйте `docker-compose` для автоматического поднятия необходимых сервисов:

```sh
docker run -d \
  --name my-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=testdb \
  -p 5432:5432 \
  --restart unless-stopped \
  postgres:latest

docker run -d --name activemq -p 61616:61616 -p 8161:8161 \
  -e ACTIVEMQ_ADMIN_LOGIN=admin \
  -e ACTIVEMQ_ADMIN_PASSWORD=password \
  rmohr/activemq
```

После запуска контейнеров убедитесь, что PostgreSQL и ActiveMQ работают:
```sh
docker ps
```

### 2. Запуск локально (без Docker)

1. **Настроить базу данных**  
   Установите PostgreSQL и создайте базу:
   ```sh
   psql -U postgres
   CREATE DATABASE testdb;
   ```
2. **Настроить ActiveMQ**  
   Установите ActiveMQ и запустите его:
   ```sh
   ./activemq start
   ```
3. **Настроить Anypoint Studio**  
   - Откройте проект в Anypoint Studio.
   - Убедитесь, что конфигурация Database Connector указывает на PostgreSQL (`localhost:5432`).
   - Убедитесь, что конфигурация JMS Connector указывает на ActiveMQ (`tcp://localhost:61616`).
4. **Запустите Mule-приложение** через Anypoint Studio.

## Тестирование API

### 1. Создание заказа
```sh
curl -X POST "http://localhost:8081/orders" \
  -H "Content-Type: application/json" \
  -d '{"client": "John Doe", "amount": 150.5, "status": "pending"}'
```

### 2. Получение заказа
```sh
curl -X GET "http://localhost:8081/orders/1"
```

### 3. Обновление статуса заказа
```sh
curl -X PUT "http://localhost:8081/orders/1/status" \
  -H "Content-Type: application/json" \
  -d '{"status": "shipped"}'
```

### 4. Удаление заказа
```sh
curl -X DELETE "http://localhost:8081/orders/1"
```

## Логирование сообщений из ActiveMQ

После обновления статуса заказов в ActiveMQ отправляется сообщение.
Для проверки работы можно использовать ActiveMQ Web Console:

- Откройте в браузере `http://localhost:8161/admin` (логин: `admin`, пароль: `password`).
- Перейдите в раздел `Queues` и проверьте сообщения.

## Обработка ошибок

- **400 Bad Request** – неверный формат входных данных.
- **404 Not Found** – заказ не найден.
- **500 Internal Server Error** – внутренняя ошибка сервера.

## Деплой

Проект можно собрать и запустить через Maven:
```sh
mvn clean package -DskipTests
```

Затем развернуть `target/your-api.jar` в Mule Runtime.

## Заключение

Этот проект демонстрирует интеграцию MuleSoft с PostgreSQL и ActiveMQ, а также работу с Docker. Все ключевые компоненты настроены для удобного развертывания и тестирования.


