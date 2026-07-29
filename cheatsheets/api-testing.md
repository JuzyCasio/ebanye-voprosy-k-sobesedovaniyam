# API-автоматизация на Python — короткая шпаргалка

[← Все шпаргалки](README.md) · [Полный материал](../materials/api-testing.md)

Короткий повтор перед собеседованием AQA/SDET Python. Подробные объяснения и практические сценарии находятся в полном материале.

## 1. Что проверяет API-тест

API-тест отправляет запрос программному интерфейсу и проверяет:

- status code;
- headers;
- body;
- schema;
- бизнес-правила;
- авторизацию;
- изменение состояния;
- ошибки и побочные эффекты.

API-тесты обычно быстрее и стабильнее UI-тестов, но не проверяют пользовательский интерфейс.

## 2. HTTP request и response

Request:

- method;
- URL;
- headers;
- path/query parameters;
- cookies;
- body;
- auth.

Response:

- status code;
- headers;
- body;
- cookies;
- URL и redirect history;
- время выполнения.

## 3. HTTP-методы

| Метод | Назначение |
|---|---|
| `GET` | Получить данные |
| `POST` | Создать ресурс или выполнить операцию |
| `PUT` | Полностью заменить ресурс |
| `PATCH` | Частично изменить |
| `DELETE` | Удалить |
| `HEAD` | Получить только headers |
| `OPTIONS` | Узнать доступные возможности |

Фактическое поведение определяется контрактом API.

## 4. Safe и idempotent

Safe-метод не должен менять бизнес-состояние: `GET`, `HEAD`, `OPTIONS`.

Идемпотентный запрос при повторении оставляет такое же итоговое состояние. Обычно идемпотентны `GET`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`.

`POST` обычно не идемпотентен. Повтор может быть безопасен при поддержке `Idempotency-Key`.

Идемпотентность не означает одинаковый status code каждого ответа.

## 5. Status codes

| Код | Значение |
|---|---|
| `200` | Успех |
| `201` | Создано |
| `202` | Принято в асинхронную обработку |
| `204` | Успех без body |
| `400` | Некорректный запрос |
| `401` | Нет корректной аутентификации |
| `403` | Нет права на операцию |
| `404` | Ресурс не найден |
| `409` | Конфликт состояния |
| `422` | Ошибка валидации данных |
| `429` | Превышен лимит |
| `500` | Ошибка сервера |
| `502` | Плохой ответ от upstream |
| `503` | Сервис недоступен |
| `504` | Gateway не дождался upstream |

Проверяй код из контракта, а не просто «примерно подходящий».

## 6. Headers, параметры и JSON

Частые headers:

- `Authorization`;
- `Content-Type`;
- `Accept`;
- `Idempotency-Key`;
- correlation ID.

`Content-Type` описывает формат текущего body. `Accept` — формат, который клиент хочет получить.

```text
GET /users/42
```

`42` — path parameter.

```text
GET /users?role=admin&page=2
```

`role` и `page` — query parameters.

JSON и Python:

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` или `float` |
| boolean | `bool` |
| null | `None` |

JSON — текстовый формат, а `dict` — объект Python.

## 7. Базовый запрос через `requests`

```python
import requests


response = requests.get(
    "https://api.example.com/users/42",
    timeout=5,
)

assert response.status_code == 200
```

У `requests` нет timeout по умолчанию, поэтому его нужно указывать явно.

## 8. `params`, `json` и `data`

Query parameters:

```python
response = requests.get(
    f"{base_url}/users",
    params={"role": "admin", "page": 2},
    timeout=5,
)
```

JSON:

```python
response = requests.post(
    f"{base_url}/users",
    json={"name": "Alex"},
    timeout=5,
)
```

Form data:

```python
response = requests.post(
    f"{base_url}/login",
    data={"username": "alex", "password": "secret"},
    timeout=5,
)
```

`json=` сериализует объект и устанавливает подходящий `Content-Type`. Словарь в `data=` обычно отправляется как form data.

## 9. Объект `Response`

```python
response.status_code
response.headers
response.text
response.content
response.json()
response.cookies
response.url
response.history
response.elapsed
response.request
```

- `text` — строка;
- `content` — bytes;
- `json()` — разбор JSON.

Успешный `json()` не доказывает правильность данных.

## 10. `raise_for_status()` и исключения

```python
response.raise_for_status()
```

Метод выбрасывает `HTTPError` для неуспешного HTTP-статуса. В негативном тесте часто удобнее явно проверить ожидаемый `400`, `404` или другой код.

```python
try:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
except requests.Timeout:
    ...
except requests.ConnectionError:
    ...
except requests.HTTPError:
    ...
except requests.RequestException:
    ...
```

Не лови исключение только ради того, чтобы скрыть полезный stack trace.

## 11. Timeout

```python
requests.get(url, timeout=5)
```

Отдельные значения:

```python
requests.get(
    url,
    timeout=(3.05, 10),
)
```

- `3.05` — connect timeout;
- `10` — read timeout.

Это не обязательно предел полного времени всего сценария.

## 12. `Session`

```python
with requests.Session() as session:
    session.headers.update(
        {"Accept": "application/json"}
    )
    response = session.get(
        f"{base_url}/users",
        timeout=5,
    )
```

Session:

- хранит cookies;
- содержит общие headers/auth;
- переиспользует соединения;
- уменьшает дублирование.

## 13. Session как fixture

```python
import pytest
import requests


@pytest.fixture
def api_session():
    with requests.Session() as session:
        session.headers.update(
            {"Accept": "application/json"}
        )
        yield session
```

Общая session-scoped fixture может привести к утечке cookies или headers между тестами.

## 14. API client layer

```python
class UsersClient:
    def __init__(self, base_url, session, timeout=5):
        self.base_url = base_url.rstrip("/")
        self.session = session
        self.timeout = timeout

    def get(self, user_id):
        return self.session.get(
            f"{self.base_url}/users/{user_id}",
            timeout=self.timeout,
        )

    def create(self, payload):
        return self.session.post(
            f"{self.base_url}/users",
            json=payload,
            timeout=self.timeout,
        )
```

Клиент отвечает за HTTP-взаимодействие. Тест — за сценарий и проверки.

## 15. Что проверять в ответе

Обычно:

1. status code;
2. `Content-Type`;
3. структура;
4. обязательные поля;
5. типы;
6. бизнес-значения;
7. сохранённое состояние;
8. побочные эффекты.

Одного status code недостаточно: `200` может содержать неправильные данные.

## 16. Negative tests

Проверяй:

- отсутствующее поле;
- неправильный тип;
- граничные значения;
- слишком длинную строку;
- неправильный формат;
- неизвестное поле;
- несуществующий ресурс;
- отсутствие токена;
- неправильную роль;
- конфликт уникальности;
- неправильный `Content-Type`.

Ошибка должна иметь ожидаемый код и понятное стабильное тело.

## 17. Тестовые данные и cleanup

```python
from uuid import uuid4


def build_user_payload(**overrides):
    payload = {
        "name": "Test User",
        "email": f"user-{uuid4()}@example.com",
    }
    payload.update(overrides)
    return payload
```

Каждый тест получает новые данные.

```python
@pytest.fixture
def created_user(users_client):
    response = users_client.create(build_user_payload())
    assert response.status_code == 201

    user = response.json()
    yield user

    users_client.delete(user["id"])
```

Cleanup должен выполняться и после падения теста.

## 18. Проверка CRUD

Create:

1. отправить `POST`;
2. проверить `201`;
3. проверить body;
4. получить через `GET`;
5. очистить.

Update:

- проверить изменённые поля;
- убедиться, что остальные не потерялись;
- проверить последующим `GET`.

Delete:

- проверить код;
- выполнить `GET`;
- проверить повторный `DELETE`;
- учитывать soft/hard delete.

## 19. Авторизация и права

Basic Auth:

```python
requests.get(
    url,
    auth=("username", "password"),
    timeout=5,
)
```

Bearer Token:

```python
requests.get(
    url,
    headers={"Authorization": f"Bearer {token}"},
    timeout=5,
)
```

- `401` — клиент не аутентифицирован;
- `403` — пользователь распознан, но действие ему запрещено.

Проверяй отсутствующий, неправильный, истёкший токен, другую роль и доступ к чужому ресурсу.

## 20. JSON Schema

```python
from jsonschema import validate


schema = {
    "type": "object",
    "required": ["id", "name"],
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
    },
}

validate(
    instance=response.json(),
    schema=schema,
)
```

Схема проверяет структуру и типы, но не заменяет бизнес-assertions.

## 21. Pydantic

```python
from pydantic import BaseModel, ConfigDict


class UserResponse(BaseModel):
    model_config = ConfigDict(extra="forbid")

    id: int
    name: str


user = UserResponse.model_validate(response.json())
```

Pydantic по умолчанию может преобразовывать значения. Для точной проверки типов используй strict mode.

| JSON Schema | Pydantic |
|---|---|
| Языконезависимый контракт | Python-модель |
| Удобна с OpenAPI | Удобен для типизированного кода |
| Проверяет структуру | Валидирует и может преобразовывать |

## 22. Контрактное тестирование

Контракт включает:

- endpoint и method;
- parameters;
- request/response body;
- headers;
- status codes;
- schema;
- правила совместимости.

Одна JSON Schema — только часть контракта.

Потенциально ломающие изменения:

- удаление endpoint или поля;
- переименование;
- изменение типа;
- новое обязательное request-поле;
- новая семантика status code;
- сужение допустимых значений.

## 23. Пагинация, фильтры и сортировка

Пагинация:

- первая и последняя страницы;
- размер;
- пустой результат;
- отрицательные и слишком большие значения;
- отсутствие дублей;
- стабильный порядок;
- next cursor.

Фильтры и сортировка:

- один и несколько фильтров;
- отсутствие результатов;
- регистр и спецсимволы;
- `null`;
- неизвестное поле;
- оба направления сортировки;
- сочетание с пагинацией.

## 24. Файлы и redirect

Upload:

```python
with open("report.pdf", "rb") as file:
    response = requests.post(
        f"{base_url}/files",
        files={"file": ("report.pdf", file, "application/pdf")},
        timeout=10,
    )
```

Проверяй имя, MIME type, размер, checksum и содержимое.

Redirect:

```python
response = requests.get(
    url,
    allow_redirects=False,
    timeout=5,
)
assert response.status_code == 302
```

Если нужно проверить первый ответ, автоматический redirect отключают.

## 25. Retry и rate limit

```python
from requests.adapters import HTTPAdapter
from urllib3.util import Retry


retry = Retry(
    total=3,
    backoff_factor=0.5,
    status_forcelist=[429, 502, 503, 504],
    allowed_methods={"GET", "HEAD", "OPTIONS"},
)

session.mount(
    "https://",
    HTTPAdapter(max_retries=retry),
)
```

Не повторяй любой `POST`: операция могла выполниться, а ответ потеряться.

Rate limit часто возвращает:

- `429 Too Many Requests`;
- `Retry-After`;
- headers с лимитом.

## 26. Polling и eventual consistency

Для `202 Accepted`:

1. получить ID операции;
2. периодически проверять статус;
3. завершить при success;
4. упасть при failed;
5. иметь общий deadline.

Фиксированный `sleep(30)` не учитывает фактическое состояние.

При eventual consistency жди конкретное конечное состояние. Не объясняй любой дефект фразой «это eventual consistency».

## 27. Моки и внешние системы

Mock/stub полезен для:

- редких ошибок;
- timeout;
- нестабильных зависимостей;
- проверки отправленного запроса.

Mock не заменяет настоящий integration test.

Мок Python-функции не проверяет URL, headers и сериализацию. HTTP stub server проверяет больше транспортных деталей.

## 28. Параллельность, конфигурация и логи

Для параллельности нужны:

- уникальные пользователи и данные;
- независимый cleanup;
- отсутствие зависимости от порядка;
- отсутствие общих файлов;
- учёт rate limit.

Конфигурация:

```python
import os


BASE_URL = os.environ["API_BASE_URL"]
API_TOKEN = os.environ["API_TOKEN"]
```

Не храни секреты в репозитории.

Логируй method, URL, status, duration и correlation ID. Не логируй пароли, tokens, cookies и персональные данные.

Для внутреннего TLS-сертификата передай доверенный CA bundle через `verify="company-ca.pem"`. Не превращай `verify=False` в постоянное решение: оно отключает проверку подлинности сервера.

## 29. Requests и HTTPX

| Requests | HTTPX |
|---|---|
| Sync API | Sync и async API |
| `Session` | `Client`/`AsyncClient` |
| Нет timeout по умолчанию | Есть timeout по умолчанию |
| Очень распространён | Поддерживает HTTP/2 |

Для обычных синхронных pytest-тестов `requests` часто достаточно.

У GraphQL HTTP status может быть `200`, а ошибка находиться в поле `errors`, поэтому нужно проверять и `data`, и `errors`.

## 30. Flaky-тесты и диагностика

Причины flaky:

- общие данные;
- отсутствие timeout;
- неправильный polling;
- нестабильная зависимость;
- неконтролируемый retry;
- плохой cleanup;
- одинаковые данные в workers;
- текущее время;
- rate limit.

Диагностика:

1. Проверить method, URL и окружение.
2. Посмотреть status code и body.
3. Посмотреть request без секретов.
4. Найти correlation ID.
5. Проверить логи сервиса.
6. Проверить БД, брокер и downstream.
7. При необходимости повторить через `curl`.

## 31. Частые ответы

**Зачем timeout?**  
Чтобы запрос не ждал неограниченно долго. У `requests` его нет по умолчанию.

**Зачем Session?**  
Для cookies, общих настроек и переиспользования соединений.

**Почему status code недостаточно?**  
`200` может содержать неправильные данные.

**Почему опасен retry POST?**  
Операция могла выполниться, а ответ потеряться. Повтор создаст дубль.

**Что проверяет схема?**  
Структуру и типы. Бизнес-значения проверяются отдельно.

**Чем `401` отличается от `403`?**  
`401` — нет корректной аутентификации. `403` — нет права.

**Что такое контрактный тест?**  
Проверка согласованного интерфейса между consumer и provider.

**Что такое polling?**  
Повторная проверка статуса асинхронной операции до результата или deadline.

## 32. Сильный короткий ответ

> Я выношу HTTP-вызовы в API-клиенты, использую Session, обязательные timeout и безопасное логирование. Фикстуры управляют авторизацией, данными и cleanup. Тесты проверяют status code, структуру и бизнес-результат, используют уникальные данные и не зависят от порядка. Retry применяю только к безопасным или явно идемпотентным операциям, а асинхронные процессы проверяю polling с deadline.

## 33. Что повторить перед собеседованием

1. HTTP methods и status codes.
2. Идемпотентность.
3. `params`, `json`, `data`, headers.
4. `Response`, timeout и исключения.
5. `Session`.
6. API client layer.
7. Negative testing.
8. Авторизация и `401`/`403`.
9. JSON Schema и Pydantic.
10. Retry и `Idempotency-Key`.
11. Polling и eventual consistency.
12. Данные, cleanup и параллельность.
