# Диплом ExploreWithMe — этап 1 (аудит 08.11.2026)

## Что подтверждено по диагностическому ZIP

- Командный репозиторий: `UserOwl-dotNew/explore-with-me`, итоговая `main` коммит `58ee53d`.
- Pull request #15 из `feature` объединён в командную `main` 10.08.2026. Это **не подтверждает** учебную приёмку ревьюером.
- Личная папка `C:\Users\Anna\IdeaProjects\explore-with-me-diploma` содержит **ровно те же 157 исходных файлов**, что команда. Но это ещё **не коммит и не push**: личный `main` GitHub остаётся `8fcf3e5` (шаблон).
- Только групповая версия ранее прошла 184 Maven-теста на Java 21. Новая Spring Cloud версия в этой среде Maven и Docker не запускалась.
- Этот пакет содержит полную проектную версию с инфраструктурой этапа 1 и защитой при применении. Автоматическая проверка структуры выполнена. **Не отправлять PR до проверки Maven и реального запуска.**

## Реализация

- Maven: `infra/config-server`, `infra/discovery-server`, `infra/gateway-server`, `core/main-service`, прежние `statistics-service/*`, `common`.
- Eureka на `8761`; Config Server на случайном порту через Eureka; `main-service` и `stats-server` на случайных портах.
- Конфигурация `main-service`, `stats-server`, `gateway-server` в `infra/config-server/src/main/resources/config/`.
- Gateway `8080` маршрутизирует `admin/users/events/categories/compilations` через `lb://main-service`.
- Клиент статистики получает адрес из `DiscoveryClient`, с краткими повторами при отсутствии регистрации.
- Сохранены все исходные Java-классы бизнес-логики, исключая адресное изменение `StatsClient`/`RestTemplateConfig` и добавленные классы инфраструктуры.

## Применение без изменения командного репозитория

1. Распакуй архив в отдельную папку.
2. Запусти `RUN_APPLY.cmd`. Скрипт проверяет личный репозиторий, его текущую `main`, доступность GitHub, SHA256 всех исходных файлов, **отсутствие неожиданных изменений** и отсутствие ветки `spring-cloud`. При несоответствии — `STOP`.
3. Только если проверки прошли, он создаёт **локальный** коммит групповой основы в личной `main`, создаёт локальную ветку `spring-cloud` и применяет полный этап 1. На GitHub сам не отправляет. Код группового репозитория не изменяет.
4. Открой личный дипломный проект в IntelliJ IDEA: Project SDK и Maven Runner — JDK **21** (не JDK 25).
5. В корневой панели Maven: `Lifecycle → clean`, затем `test`, затем `package`. Важно получить BUILD SUCCESS после изменений, не только предыдущие 184 теста на групповой версии.
6. Запусти Docker Desktop. Из терминала **личной дипломной папки**, где `docker-compose.yml`:
   ```powershell
   docker compose up --build -d
   powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\smoke-stage1.ps1
   ```
   Smoke ожидает реальную Eureka-регистрацию четырёх сервисов, фактические порты, конфигурацию Config Server каждого приложения и HTTP через Gateway.
7. Только при успешном `clean test package` + полном `PASS` smoke проверь Git diff, затем отправь **сначала** личную `main`, **затем** `spring-cloud`:
   ```powershell
   git push -u origin main
   git push -u origin spring-cloud
   ```
   Создай PR `spring-cloud → main`, приложи результаты проверок. Не объединяй до одобрения.

Никакие исходные данные группы и GitHub не изменяются при распаковке архива. **Не использовать** `docker compose down -v`: оно удаляет локальные тома БД.

## Особенности

- Старый `application-docker.properties` не используется. Реальная конфигурация импортируется централизованно; локальные тестовые `application.properties` отключают внешнюю инфраструктуру.
- При старте клиентов Eureka должен быть доступен. Сервер Config Server обнаруживается через Eureka по имени `config-server`.
- `test` проверяет код; smoke — сетевые сценарии. Ни один из них не доказывает 100% отсутствие дефектов, но оба обязательны перед ревью.
- Если GitHub личного репозитория теперь содержит новые коммиты/PR, этот пакет **остановится**, не перезаписывая их.
