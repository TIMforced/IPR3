# ДЗ 3. Счётчик в Docker

[Условие задания](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

## Запуск с Redis

```sh
cp .env.example .env
docker compose up -d --build --wait
curl http://127.0.0.1:8081/
```

В `.env` можно поменять порт `APP_PORT` и текст `GREETING`. Приложение доступно на порту `8081`, а Redis доступен внутри сети Compose с именем `redis`. Redis пишет изменения на `redis-data`.

## Проверка Redis

Два запроса увеличивают счётчик. Запомни с равним число после второго запроса с числом после пересоздания контейнеров.

```sh
curl -X POST http://127.0.0.1:8081/increment
curl -X POST http://127.0.0.1:8081/increment
docker compose down
docker compose up -d --wait
curl http://127.0.0.1:8081/
```

Остановка Redis и проверка HTTP-статусов. `curl -i` показывает их в первой строке ответа.

```sh
docker compose stop redis
curl -i http://127.0.0.1:8081/healthz
curl -i http://127.0.0.1:8081/readyz
curl -i -X POST http://127.0.0.1:8081/increment
docker compose start redis
curl -fsS --retry 10 --retry-delay 1 http://127.0.0.1:8081/readyz
curl http://127.0.0.1:8081/
curl -X POST http://127.0.0.1:8081/increment
```

При остановленном Redis сервер отвечает на `/healthz` со статусом `200`. Проверка базы `/readyz` и попытка увеличить счётчик возвращают `503`. После запуска Redis счётчик доступен без перезапуска.

## Проверка SQLite

Один контейнер запускается на порту `8080`.

```sh
docker build -t doc .
docker run -d --name doc-counter -p 127.0.0.1:8080:8000 -e 'GREETING=Привет из Docker!' doc
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
docker logs doc-counter
```

Compose с SQLite можно проверить отдельно на порту `8082`.

```sh
APP_PORT=8082 docker compose -p ipr3-sqlite -f compose.sqlite.yaml up -d --build --wait
curl -X POST http://127.0.0.1:8082/increment
docker compose -p ipr3-sqlite -f compose.sqlite.yaml rm -sf app
APP_PORT=8082 docker compose -p ipr3-sqlite -f compose.sqlite.yaml up -d --wait
curl http://127.0.0.1:8082/
```

После пересоздания `hostname` меняется, а `count` сохраняется в томе.

## Результаты

- **4 балла.** Счётчик вырос с `0` до `1`. Два запроса вернули `200`. [Логи контейнера](docker-logs.txt).
- **6 баллов.** После пересоздания счётчик равен `1`. [Проверка SQLite](checks-6.txt).
- **8 баллов.** После пересоздания счётчик равен `2`. При отключении Redis получены статусы `200`, `503`, `503`. После запуска Redis счётчик`2` увеличился до `3`. [Проверка Redis](checks-8.txt).
