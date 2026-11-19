# Домашнее задание: Docker

Решение домашнего задания №3. Выполнил: студент группы БПМИ2610 Кучун Дмитрий Сергеевич. Далее описание работы Docker над приложением `app.py` и отчет о работе.

## Запуск

```sh
cp .env.example .env
docker compose up -d --build
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
docker compose logs app
```

В `.env` настраиваются `GREETING` и `APP_PORT` (порт на компьютере). Внутри
контейнера приложение слушает порт 8000. Только порт приложения опубликован;
Redis доступен приложению по `redis://redis:6379/0`.

`docker compose down` удаляет контейнеры, но оставляет данные в томе
`<compose-project>_redis_data`. Для полного сброса данных используйте
`docker compose down -v`.

## Отчёт о проверке

Проверено в PowerShell. Данные в именованном томе не удалялись.

```powershell
docker compose up -d --build --wait
curl.exe -sS http://127.0.0.1:8080/
curl.exe -sS -X POST http://127.0.0.1:8080/increment
curl.exe -sS -X POST http://127.0.0.1:8080/increment
curl.exe -sS -X POST http://127.0.0.1:8080/increment
curl.exe -sS http://127.0.0.1:8080/
```

Перед инкрементами сервис вернул `count: 0`, после трёх POST — `count: 3`.
Затем проверено сохранение после удаления и повторного создания контейнеров:

```powershell
docker compose down
docker compose up -d --wait
curl.exe -sS http://127.0.0.1:8080/
```

После пересоздания GET вернул `count: 3`.

Проверка поведения при остановке и восстановлении Redis:

```powershell
docker compose stop redis
curl.exe -i http://127.0.0.1:8080/healthz
curl.exe -i http://127.0.0.1:8080/readyz
curl.exe -i -X POST http://127.0.0.1:8080/increment
docker compose start redis
docker compose up -d --wait
curl.exe -sS http://127.0.0.1:8080/
curl.exe -sS -X POST http://127.0.0.1:8080/increment
```

Результаты при остановленном Redis: `/healthz` — HTTP 200 и
`{"status": "alive"}`; `/readyz` и `POST /increment` — HTTP 503 и
`{"error": "storage_unavailable"}`. После запуска Redis без перезапуска
приложения GET вернул `count: 3`, следующий POST — `count: 4`.W
