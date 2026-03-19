## Запуск

### Быстрый старт

```bash
git clone <repo-url>
cd TestTaskForDevOps
cp .env.example .env
# отредактировать .env и задать POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB
docker compose up -d
```

После старта фронтенд будет доступен на `http://localhost/`, API на `http://localhost/api/time`.

Первый запуск занимает время (сначала поднимается PostgreSQL, потом бэкенд, потом nginx), так настроены healthcheck-зависимости.

### Переменные окружения

Все переменные задаются в файле `.env` (создать из `.env.example`):

| Переменная | Описание |
|---|---|
| `POSTGRES_USER` | Имя пользователя PostgreSQL |
| `POSTGRES_PASSWORD` | Пароль |
| `POSTGRES_DB` | Название базы данных |

Остальные параметры прописаны в `docker-compose.yml`. PostgreSQL доступен только через бэкенд.

### Мониторинг

Отдельный стек: Prometheus + Grafana + cadvisor + Loki + Promtail. Запускается поверх основного:

```bash
docker compose -f docker-compose.yml -f docker-compose.monitoring.yml up -d
```

Grafana доступна на `http://localhost:3001`. Логин и пароль берутся из `.env` (переменные `GF_ADMIN_USER` и `GF_ADMIN_PASSWORD`, по умолчанию `admin/admin`). Дашборд "Container Overview" появляется автоматически (CPU и память контейнеров плюс логи).

Prometheus scrape-метрики: `http://localhost:9090`. cadvisor собирает метрики контейнеров, Promtail читает логи из Docker и отправляет в Loki.

### Continuous Deployment

В `docker-compose.yml` поднимается [Watchtower](https://containrrr.dev/watchtower/). Он раз в минуту проверяет ghcr.io и перезапускает контейнеры при появлении нового образа. Чтобы заработало, нужно задать `IMAGE_PREFIX` в `.env` (например `ghcr.io/youruser/testtaskfordevops`).

Второй вариант тоже реализован, job `deploy` в CI workflow. Если выставить переменную `DEPLOY_HOST` в Settings > Variables, после успешной сборки он подключится по SSH к серверу и выполнит `docker compose pull && docker compose up -d`. Нужные настройки: переменные `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_PATH` и секрет `DEPLOY_KEY` с приватным ключом.

---

# Тестовое задание: Простая DevOps-инфраструктура с CI/CD

[![MIT License](https://img.shields.io/github/license/serega404/TestTaskForDevOps)](https://github.com/serega404/TestTaskForDevOps/blob/main/LICENSE)

## Цель

Разработать и развернуть `Docker Compose`-стек из трёх контейнеров, а также настроить CI/CD с использованием **GitHub Actions** (или **Gitea Actions**). А так же написать документацию по развертыванию и использованию.

**Здесь ожидается не просто запуск контейнеров, а их правильная настройка и взаимодействие друг с другом. Включая healtcheck, изоляцию сетей и ограничение ресурсов.** `Покажите свой максимум!`

### Доп задание: **Инструменты мониторинга**

Нужно написать Compose файл для запуска мониторингового стека, а также конфиги для него.
Задача - собирать метрики и логи с сервисов и отображать их в Grafana. Сделать простенький дашборд с графиками или взять готовые и настроить provisoring.

Предлагаемый стек:

* Grafana
* Prometheus совместимая бд
* prometheus exporter/telegraf коллекторы
* Alloy или другой коллектор логов

## Требования к проекту

* Публичный репозиторий
* Фронтенд и бэкенд части в одном репозитории:
    * Фронтенд — в папке /front.
    * Бэкенд — в папке /api.

## Архитектура Docker Compose

Стек должен состоять из следующих сервисов:

1. **Nginx (frontend)**
   * Последняя версия.
   * Отдаёт одну HTML-страницу (`/front/html/index.html`) по корневому URL.
   * Проксирует запросы на `/api` к бэкенду.

2. **Node.js (backend)**
   * Готовое приложение расположено в папке [`/api`](/api).
   * Подключается к базе данных PostgreSQL.

   Инструкция по запуску приложения лежит в файле [`/api/README.md`](/api/README.md).

3. **PostgreSQL (database)**
   * Последняя версия.

## CI (Continuous Integration)

* Использовать **GitHub Actions** или **Gitea Actions**.
* При каждом коммите:
  * Собрать Docker-образы для фронтенда и бэкенда (необходима поддержка архитектур `amd64` и `arm64`).
  * Опубликовать их в **[ghcr.io](https://ghcr.io)**.
  * Отправить уведомление в **Telegram/Discord** о результате сборки.

## CD (Continuous Deployment)

Реализовывать необязательно, но будет плюсом.
Если не реализовано - описать как это должно работать.

## Где выполнять?

* В виртуальной машине.
* На личном сервере или локальной машине.
* С помощью **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**.

## Использованные технологии

* **Docker** и **Docker Compose** для контейнеризации.
* **GitHub Actions** или **Gitea Actions** для CI/CD.
* **PostgreSQL** для хранения данных.
* **Nginx** для проксирования запросов и раздачи статических файлов.
* **Node.js+Express.js** для бэкенда.
* **GNU/Linux** для виртуальной машины или сервера.

## Взаимодействие с ревьювером

Рекомендуется форкнуть репозиторий и после выполнения задания отправить ссылку на форк. В случае приватного репозитория пригласить ревьювера в качестве контрибьютора в репозиторий.

### Чек-лист для проверки (ANSWERS.md.enc)

Здесь хранится зашифрованный чек-лист для проверки задания ревьювером.
Для расшифровки используйте консольную утилиту OpenSSL:

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -iter 2000000 -in ANSWERS.md.enc -out ANSWERS.md
```

Пароль можно запросить у ревьювера после прохождения ревью.

## Лицензия

Распространяется под лицензией MIT. Дополнительные сведения смотрите в файле [`LICENSE`](./LICENSE).