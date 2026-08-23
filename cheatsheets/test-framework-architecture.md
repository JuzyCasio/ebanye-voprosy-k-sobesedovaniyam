# Архитектура тестового фреймворка — быстрый повтор

[← Все шпаргалки](README.md) · [Полный материал](../materials/test-framework-architecture.md)

## Сильный ответ на собеседовании

> Я оставляю в тестах сценарий и бизнес-проверки. API, UI, БД и брокер изолирую в предметных адаптерах, повторяющиеся workflows — в steps layer, данные — в factories/builders. Fixtures управляют зависимостями и cleanup, settings приходят из CLI и environment variables. На уровне transport находятся timeout, безопасные логи и attachments. Для CI учитываю markers, parallel run, уникальные данные и диагностические артефакты.

## Базовые слои

```text
Tests
  ↓
Domain / Steps
  ↓
API clients / Pages / DB repositories / Broker adapters
  ↓
Transport / Config / Logging / Reporting
  ↓
Тестируемая система
```

| Слой | Ответственность |
|---|---|
| Tests | сценарий, expected result, бизнес-проверки |
| Domain / Steps | повторяющиеся бизнес-операции |
| Adapters | API, UI, DB, broker и внешние сервисы |
| Infrastructure | transport, settings, соединения, logging |
| Cross-cutting | fixtures, data, assertions, reporting, cleanup |

Главное правило: нижние слои не импортируют тесты, а тесты не дублируют детали транспорта.

## Структура проекта

```text
tests/
├── conftest.py
├── api/
├── ui/
└── integration/

framework/
├── config/
├── clients/
├── pages/
├── db/
├── broker/
├── models/
├── data/
├── steps/
├── assertions/
├── reporting/
└── utils/
```

Создавать отдельную папку стоит только при наличии отдельной ответственности.

## Tests

В тесте остаются:

1. arrange — исходное состояние;
2. act — действие;
3. assert — ожидаемый результат.

```python
def test_user_can_cancel_new_order(order_steps, new_order) -> None:
    cancelled = order_steps.cancel(new_order.id)

    assert cancelled.status == "CANCELLED"
```

В тесте не должны повторяться настройка HTTP-сессии, длинный SQL или десятки локаторов.

## Clients и adapters

- общий transport: timeout, headers, logging, retry технических ошибок;
- предметный client: endpoints и модели одного сервиса;
- Page Object: локаторы и действия со страницей;
- Repository: запросы к БД;
- Broker adapter: publish, consume, correlation ID и ожидание события.

Не создавать один `BaseClient`, который знает все сервисы, asserts и cleanup.

## Steps layer

Нужен для повторяющихся бизнес-операций:

- создать оплаченный заказ;
- зарегистрировать пользователя;
- дождаться обработки события;
- подготовить корзину.

Client знает технический контракт одного сервиса. Step может объединять несколько clients.

Если повторяющихся workflows нет, отдельный steps layer не нужен.

## Fixtures

Fixture управляет зависимостью или ресурсом:

- создаёт client/browser/connection;
- готовит исходное состояние;
- выполняет teardown.

```python
@pytest.fixture
def created_user(user_client):
    user = user_client.create(build_user())
    yield user
    user_client.delete(user.id)
```

Запомнить:

- изменяемые данные — обычно `function` scope;
- session scope подходит для settings и стабильных read-only ресурсов;
- scope не гарантирует безопасность параллельного доступа;
- важное бизнес-действие не стоит скрывать внутри autouse fixture;
- огромный `conftest.py` нужно делить по областям ответственности.

## Test data

- Factory создаёт валидный объект с defaults.
- Builder собирает сложный объект пошагово.
- Overrides позволяют менять только нужное поле.
- UUID/worker ID предотвращает конфликты.
- Cleanup должен удалять только данные текущего теста.
- Случайные данные должны быть воспроизводимыми при необходимости.

## Configuration

Приоритет настроек:

1. CLI CI;
2. environment variables;
3. локальный `.env`;
4. безопасные defaults.

Секреты не хранятся в Git и маскируются в логах.

## Assertions

- простая проверка — прямо в тесте;
- повторяющийся контракт — helper/matcher;
- название helper должно объяснять ожидаемое состояние;
- client и Page Object обычно не содержат asserts конкретного теста;
- негативные API-тесты должны иметь доступ к raw response.

## Паттерны

| Паттерн | Где полезен |
|---|---|
| Composition | сборка steps из clients без глубокой иерархии |
| Dependency Injection | явная передача clients, settings и repositories |
| Factory | создание готовых тестовых данных |
| Builder | пошаговая сборка сложного payload |
| Adapter | единый интерфейс к внешней системе |
| Facade | повторяющийся workflow из нескольких операций |
| Strategy | разные auth, cleanup или transports |
| Repository | изоляция SQL и транзакций |
| Page/Component Object | изоляция UI-локаторов и действий |

## Параллельность

Основные риски:

- одинаковые email/external ID;
- один пользователь для всех тестов;
- общий consumer group;
- один файл или порт;
- cleanup данных другого worker;
- изменяемая session fixture.

Решения:

- UUID и worker ID;
- отдельные namespaces/accounts/schemas;
- адресный cleanup;
- отсутствие глобального изменяемого состояния;
- отдельные consumer groups;
- marker для непараллельных тестов.

## Timeout, polling и retry

- timeout нужен каждой внешней операции;
- polling ждёт конкретное состояние до deadline;
- retry допустим для временной инфраструктурной ошибки или безопасной операции;
- retry не должен скрывать баг;
- фиксированный `sleep` медленный и нестабильный.

## Диагностика

При падении сохранять:

- окружение и название теста;
- method, URL и status code;
- безопасные request/response;
- correlation ID / trace ID;
- последнее состояние polling;
- screenshot, console и trace для UI;
- событие и headers для broker;
- логи без секретов.

## CI/CD

- lint и unit-тесты helpers;
- smoke на merge request;
- regression по расписанию;
- markers и параметры окружения;
- parallel run;
- корректный exit code;
- Allure, logs, screenshots и traces как artifacts;
- cleanup временных ресурсов.

## Антипаттерны

- огромный `conftest.py`;
- god object / универсальный `BaseClient`;
- глубокая иерархия `BaseTest`;
- asserts внутри client/Page Object;
- половина сценария внутри fixture;
- общий изменяемый пользователь для всех тестов;
- папка `utils` для несвязанных функций;
- глобальные settings и singleton со скрытым состоянием;
- `sleep` вместо ожидания условия;
- retry всех падений;
- секреты в репозитории и логах;
- абстракции без реального упрощения.

## Что спросить перед проектированием

1. Какие типы тестов нужны?
2. Какие сервисы и внешние системы участвуют?
3. Как создавать и очищать данные?
4. Нужна ли работа с БД и брокером?
5. Сколько окружений?
6. Нужен ли parallel run?
7. Какие ограничения и rate limits?
8. Какие артефакты нужны при падении?
9. Какие тесты запускаются в CI?
10. Кто будет поддерживать framework?

## Короткие ответы

**Зачем clients?** Изолировать URL, headers, serialization, timeout и logging.

**Зачем steps?** Выразить повторяющуюся бизнес-операцию из нескольких client-вызовов.

**Где assertions?** Простые — в тесте, повторяющиеся контракты — в matchers/helpers.

**Что в `conftest.py`?** Fixtures и hooks подходящей области видимости, но не весь framework.

**Почему composition?** Зависимости видны, части можно заменять независимо.

**Нужен ли `BaseTest`?** В pytest обычно нет: fixtures и composition проще.

**Как бороться с flaky?** Найти причину: ожидания, данные, порядок, locator, время, внешняя система. Rerun — временная мера.

**Как поддержать parallel run?** Уникальные данные, отдельные namespaces, адресный cleanup и отсутствие shared mutable state.

## Финальный чек-лист

- По тесту понятен сценарий.
- Зависимости направлены сверху вниз.
- API/UI/DB/broker изолированы в adapters.
- Fixtures имеют понятные scope и cleanup.
- Config приходит извне, secrets маскируются.
- Данные уникальны для workers.
- У внешних операций есть timeout.
- Для async используется polling.
- Падение оставляет полезные artifacts.
- Framework helpers имеют unit-тесты.
- Новая функциональность добавляется без массового изменения существующих тестов.
