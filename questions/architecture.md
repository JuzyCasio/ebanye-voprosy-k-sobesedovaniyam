# Архитектура и проектирование

[← На главную](../README.md)

Проектирование тестовых решений: слои фреймворка, зависимости, паттерны, тестовые данные, стабильность и масштабирование.

[Полный конспект](../materials/test-framework-architecture.md) · [Быстрая шпаргалка](../cheatsheets/test-framework-architecture.md)

## Содержание

### Основы

- что такое архитектура тестового фреймворка;
- чем фреймворк отличается от набора тестов;
- какие задачи решает архитектура;
- как выбирать структуру под контекст проекта.

### Слои

- tests — сценарии и бизнес-проверки;
- domain/steps — повторяющиеся бизнес-операции;
- adapters — API, UI, DB, broker и внешние системы;
- infrastructure — transport, configuration, logging и reporting;
- fixtures, test data, assertions и cleanup как сквозные механизмы.

### Устройство Python-проекта

- структура директорий;
- разделение `conftest.py`;
- fixtures и scope;
- предметные clients вместо универсального god object;
- Page Object и Component Object;
- repositories и broker adapters;
- models, factories, builders и assertion helpers.

### Паттерны

- composition over inheritance;
- dependency injection;
- Factory и Builder;
- Adapter;
- Facade;
- Strategy;
- Repository;
- Page Object.

### Эксплуатация

- конфигурация окружений и secrets;
- уникальные данные и parallel run;
- timeout, polling и retry;
- диагностика flaky-тестов;
- CI/CD и артефакты;
- unit-тесты вспомогательного кода;
- масштабирование фреймворка на несколько команд.

### Собеседование

В [полном конспекте](../materials/test-framework-architecture.md#проектирование-на-собеседовании) есть:

- порядок проектирования фреймворка с нуля;
- готовый развёрнутый ответ;
- вопросы, которые нужно задать до проектирования;
- архитектурные trade-offs;
- антипаттерны;
- чек-лист ревью;
- короткие ответы на частые вопросы.
