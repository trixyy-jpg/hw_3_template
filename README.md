# ДЗ 3. Docker

В папке `app` находится приложение со счётчиком. Код приложения не менялся.

`Dockerfile` собирает образ приложения. В `compose.yaml` запускаются два контейнера: `app` и `redis`. Значение счётчика хранится в Redis и сохраняется в volume `redis_data`.

## Запуск

Нужен установленный Docker. Команды выполняются из папки проекта.

```bash
cp .env.example .env
docker compose up -d --build
```

Приложение открывается по адресу http://localhost:8080/.

В файле `.env` можно изменить `HOST_PORT` (порт) и `GREETING` (приветствие). Пример настроек находится в `.env.example`.

Остановка:

```bash
docker compose down
```

## Проверка одного контейнера

```bash
docker build -t hw3-app .
docker run -d --name hw3-single -p 8081:8000 -e GREETING="Hello!" hw3-app
curl http://localhost:8081/
curl -X POST http://localhost:8081/increment
docker logs hw3-single
docker rm -f hw3-single
```

## Проверка Compose с SQLite

Для хранения данных в volume используется файл `compose.sqlite.yaml`.

```bash
docker compose -f compose.sqlite.yaml up -d --build
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
docker compose -f compose.sqlite.yaml down
docker compose -f compose.sqlite.yaml up -d
curl http://localhost:8080/
docker compose -f compose.sqlite.yaml down
```

Счётчик после повторного запуска должен остаться прежним.

## Проверка Compose с Redis

```bash
docker compose up -d --build
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
docker compose down
docker compose up -d
curl http://localhost:8080/
```

После повторного запуска значение счётчика не должно измениться. Команду `down -v` здесь использовать нельзя, иначе удалится volume с данными.

Проверка отключения Redis:

```bash
docker compose stop redis
curl -i http://localhost:8080/healthz
curl -i http://localhost:8080/readyz
curl -i -X POST http://localhost:8080/increment
docker compose start redis
curl -i http://localhost:8080/readyz
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
```

При остановленном Redis `/healthz` возвращает 200, а `/readyz` и `/increment` — 503. После запуска Redis нужно дождаться, пока `/readyz` снова вернёт 200. Счётчик должен сохраниться.

Результаты проверок находятся в `reports/verification_8.txt`.
