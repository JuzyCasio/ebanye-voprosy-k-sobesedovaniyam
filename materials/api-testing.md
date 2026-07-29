# API-автоматизация на Python — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/api-testing.md)

Материал для подготовки к собеседованию AQA/SDET Python. Здесь HTTP рассматривается с практической стороны: как строить API-тесты, пользоваться `requests`, проверять контракты, проектировать клиент и диагностировать падения.

Сетевые основы подробно разобраны в [отдельном конспекте](networks.md), а возможности pytest — в [конспекте по Pytest](pytest.md).

## 1. Что такое API

API — интерфейс, через который одна программа взаимодействует с другой.

В веб-приложениях клиент обычно отправляет HTTP-запрос, а сервер возвращает HTTP-ответ.

Пример:

```text
GET /api/users/42
```

Ответ:

```json
{
  "id": 42,
  "name": "Alex"
}
```

### Короткий ответ

API задаёт правила взаимодействия между системами: какие операции доступны, какие данные нужно передать и какой ответ можно получить.

## 2. Что такое API-тестирование

API-тест проверяет систему через программный интерфейс без взаимодействия с UI.

Обычно проверяются:

- статус ответа;
- тело;
- headers;
- схема данных;
- бизнес-правила;
- авторизация;
- изменение состояния;
- ошибки;
- время ответа;
- взаимодействие с другими сервисами.

### Короткий ответ

API-тестирование — проверка запросов, ответов и бизнес-поведения сервиса через его API.

## 3. Почему API-тесты важны для AQA

По сравнению с UI-тестами они обычно:

- быстрее;
- стабильнее;
- точнее локализуют проблему;
- проще запускаются параллельно;
- позволяют проверить больше негативных сценариев;
- не зависят от браузерной вёрстки.

Но API-тест не проверяет пользовательский интерфейс. Поэтому API- и UI-тесты дополняют друг друга.

## 4. Из чего состоит HTTP-запрос

В запрос входят:

- метод;
- URL;
- headers;
- query parameters;
- path parameters;
- cookies;
- body, если оно предусмотрено;
- данные авторизации.

Пример:

```http
POST /api/users?notify=true HTTP/1.1
Host: example.com
Authorization: Bearer token
Content-Type: application/json

{"name": "Alex"}
```

## 5. Из чего состоит HTTP-ответ

В ответ входят:

- status code;
- headers;
- body;
- cookies;
- фактический URL;
- история redirect;
- время выполнения.

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/42

{"id": 42, "name": "Alex"}
```

## 6. Структура URL

```text
https://api.example.com:443/v1/users/42?details=true#section
```

- `https` — схема;
- `api.example.com` — host;
- `443` — port;
- `/v1/users/42` — path;
- `details=true` — query string;
- `section` — fragment.

Fragment после `#` браузер обычно не отправляет серверу в HTTP-запросе.

## 7. Path parameters и query parameters

Path parameter идентифицирует ресурс:

```text
GET /users/42
```

Query parameter меняет выборку или представление:

```text
GET /users?role=admin&page=2
```

Это не абсолютное правило проектирования, но хороший ориентир.

## 8. Основные HTTP-методы

- `GET` — получить данные;
- `POST` — создать ресурс или выполнить операцию;
- `PUT` — полностью заменить ресурс;
- `PATCH` — частично изменить;
- `DELETE` — удалить;
- `HEAD` — получить только headers;
- `OPTIONS` — узнать доступные возможности.

Реальное поведение определяется контрактом конкретного API.

## 9. Safe и idempotent методы

Safe-метод не должен менять бизнес-состояние. К safe относятся `GET`, `HEAD` и `OPTIONS`.

Идемпотентный запрос можно повторить несколько раз, и итоговое состояние останется таким же, как после одного запроса.

Обычно идемпотентны:

- `GET`;
- `PUT`;
- `DELETE`;
- `HEAD`;
- `OPTIONS`.

`POST` обычно не идемпотентен: повтор может создать второй ресурс. Но API может поддерживать `Idempotency-Key`.

### Важное уточнение

Идемпотентность говорит про итоговое состояние сервера, а не про одинаковый status code. Первый `DELETE` может вернуть `204`, а повторный — `404`, хотя ресурс всё равно удалён.

## 10. Основные группы status codes

- `1xx` — информационные;
- `2xx` — успешное выполнение;
- `3xx` — перенаправление;
- `4xx` — запрос клиента нельзя выполнить;
- `5xx` — сервер не смог обработать корректный запрос.

Частые коды:

| Код | Значение |
|---|---|
| `200` | Успешный ответ |
| `201` | Ресурс создан |
| `202` | Запрос принят в асинхронную обработку |
| `204` | Успех без тела ответа |
| `400` | Некорректный запрос |
| `401` | Нет корректной аутентификации |
| `403` | Пользователь известен, но доступ запрещён |
| `404` | Ресурс не найден |
| `409` | Конфликт текущего состояния |
| `422` | Данные синтаксически понятны, но не проходят валидацию |
| `429` | Превышен лимит запросов |
| `500` | Внутренняя ошибка сервера |
| `502` | Gateway получил плохой ответ от upstream |
| `503` | Сервис временно недоступен |
| `504` | Gateway не дождался upstream |

Нужно проверять код из контракта, а не выбирать «примерно подходящий».

## 11. Headers

Часто используемые request headers:

- `Authorization`;
- `Content-Type`;
- `Accept`;
- `User-Agent`;
- `Idempotency-Key`;
- `X-Request-ID` или `X-Correlation-ID`.

Часто проверяемые response headers:

- `Content-Type`;
- `Location`;
- `Retry-After`;
- `Cache-Control`;
- `Set-Cookie`;
- correlation ID.

Названия HTTP headers регистронезависимы.

## 12. `Content-Type` и `Accept`

`Content-Type` описывает формат отправленного или полученного body:

```http
Content-Type: application/json
```

`Accept` сообщает серверу, в каком формате клиент хочет получить ответ:

```http
Accept: application/json
```

### Короткий ответ

`Content-Type` — что находится в текущем body. `Accept` — какой формат ответа ожидает клиент.

## 13. JSON и сериализация

JSON поддерживает:

- object;
- array;
- string;
- number;
- boolean;
- `null`.

Соответствие Python:

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` или `float` |
| boolean | `bool` |
| null | `None` |

JSON — текстовый формат. Python-словарь и JSON — не одно и то же.

## 14. Что такое REST

REST — архитектурный стиль построения распределённых систем.

Для REST API характерны:

- ресурсы с адресами;
- стандартные HTTP-методы;
- stateless-взаимодействие;
- представление ресурса, часто JSON;
- использование HTTP-семантики и status codes.

Не любое HTTP API автоматически является REST API.

## 15. Что такое OpenAPI и Swagger

OpenAPI — спецификация описания HTTP API в JSON или YAML.

Она может описывать:

- endpoints;
- методы;
- параметры;
- request body;
- responses;
- schemas;
- authentication.

Swagger — набор инструментов вокруг OpenAPI, например Swagger UI и Swagger Editor.

### Что проверять

Наличие красивой Swagger-страницы не гарантирует, что реализация соответствует контракту. Нужно сравнивать реальные запросы и ответы со спецификацией.

## 16. Установка библиотек

```bash
pip install requests pytest jsonschema "pydantic[email]"
```

Дополнение `[email]` устанавливает библиотеку, необходимую для типа `EmailStr`. Если проверка email не нужна, достаточно обычного `pydantic`.

Минимальная структура:

```text
project/
  clients/
  models/
  schemas/
  tests/
    api/
  conftest.py
  pyproject.toml
```

## 17. Первый запрос через `requests`

```python
import requests


response = requests.get(
    "https://api.example.com/users/42",
    timeout=5,
)

assert response.status_code == 200
```

В реальном проекте URL и timeout лучше хранить в конфигурации или API-клиенте.

## 18. Query parameters через `params`

```python
import requests


response = requests.get(
    "https://api.example.com/users",
    params={
        "role": "admin",
        "page": 2,
        "limit": 20,
    },
    timeout=5,
)
```

Не нужно вручную склеивать query string. Клиент корректно выполнит URL encoding.

Фактический URL:

```python
print(response.request.url)
```

## 19. Отправка JSON

```python
payload = {
    "name": "Alex",
    "email": "alex@example.com",
}

response = requests.post(
    "https://api.example.com/users",
    json=payload,
    timeout=5,
)
```

Параметр `json=` сериализует объект и добавляет подходящий `Content-Type`.

## 20. `json=` и `data=`

```python
requests.post(url, json={"name": "Alex"}, timeout=5)
```

Используется для JSON.

```python
requests.post(url, data={"name": "Alex"}, timeout=5)
```

Словарь в `data=` обычно отправляется как form data.

```python
requests.post(
    url,
    data='{"name": "Alex"}',
    headers={"Content-Type": "application/json"},
    timeout=5,
)
```

Так тоже можно отправить JSON, но код сложнее и легче ошибиться. Обычно лучше `json=`.

## 21. PUT, PATCH и DELETE

```python
response = requests.put(
    f"{base_url}/users/42",
    json={"name": "Alex", "email": "alex@example.com"},
    timeout=5,
)
```

```python
response = requests.patch(
    f"{base_url}/users/42",
    json={"name": "Bob"},
    timeout=5,
)
```

```python
response = requests.delete(
    f"{base_url}/users/42",
    timeout=5,
)
```

После изменения важно проверить не только ответ, но и фактическое состояние ресурса отдельным `GET`.

## 22. Объект `Response`

Полезные атрибуты и методы:

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

- `text` — декодированный текст;
- `content` — bytes;
- `json()` — попытка разобрать body как JSON.

Успешный `response.json()` не доказывает корректность бизнес-данных или схемы.

## 23. `raise_for_status()`

```python
response = requests.get(url, timeout=5)
response.raise_for_status()
```

Метод выбрасывает `HTTPError` для неуспешного HTTP-статуса.

Он удобен внутри общего API-клиента или служебного кода, но в негативном тесте часто требуется явно проверить `400`, `404` или другой ожидаемый код. Если клиент всегда вызывает `raise_for_status()`, он не должен мешать тестировать ожидаемые ошибки.

## 24. Timeout

У `requests` нет таймаута по умолчанию. Без `timeout` вызов может ждать очень долго.

```python
response = requests.get(url, timeout=5)
```

Один `float` применяется к connect и read timeout.

Можно задать отдельно:

```python
response = requests.get(
    url,
    timeout=(3.05, 10),
)
```

- первое значение — connect timeout;
- второе — read timeout.

Это не гарантированный предел полного времени запроса: timeout контролирует отдельные сетевые ожидания, а не весь сценарий по часам.

## 25. Исключения `requests`

```python
import requests


try:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
except requests.Timeout:
    print("Сервер не ответил вовремя")
except requests.ConnectionError:
    print("Не удалось установить соединение")
except requests.HTTPError as error:
    print(f"HTTP error: {error.response.status_code}")
except requests.RequestException as error:
    print(f"Request failed: {error}")
```

Все основные исключения библиотеки наследуются от `RequestException`.

В тесте не нужно бездумно ловить все исключения: необработанное исключение часто даёт более понятное падение.

## 26. Что такое `Session`

```python
import requests


with requests.Session() as session:
    session.headers.update({"Accept": "application/json"})
    response = session.get(
        "https://api.example.com/users",
        timeout=5,
    )
```

`Session`:

- хранит cookies между запросами;
- позволяет задать общие headers и auth;
- переиспользует соединения через connection pool;
- имеет те же основные методы: `get`, `post`, `put` и другие.

Для набора API-тестов `Session` обычно удобнее одиночных вызовов `requests.get()`.

## 27. Session как pytest-фикстура

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

Scope выбирают осознанно:

- `function` даёт максимальную изоляцию;
- `session` уменьшает накладные расходы, но создаёт общее состояние.

Если токены, cookies или headers меняются в тестах, общая session-scoped фикстура может привести к зависимости тестов друг от друга.

## 28. Авторизованная фикстура

```python
import pytest
import requests


@pytest.fixture
def authorized_session(api_url, credentials):
    with requests.Session() as session:
        login_response = session.post(
            f"{api_url}/auth/login",
            json=credentials,
            timeout=5,
        )
        assert login_response.status_code == 200

        token = login_response.json()["access_token"]
        session.headers.update(
            {"Authorization": f"Bearer {token}"}
        )
        yield session
```

Если подготовка авторизации упала, pytest покажет fixture error, а не падение бизнес-проверки.

## 29. Зачем нужен API client layer

Плохо:

```python
def test_user():
    response = requests.get(
        "https://api.example.com/users/42",
        headers={"Authorization": "Bearer token"},
        timeout=5,
    )
    assert response.status_code == 200
```

Если такие вызовы повторяются во всех тестах, дублируются:

- `base_url`;
- headers;
- timeout;
- auth;
- logging;
- построение URL.

API client layer скрывает транспортные детали за понятными методами.

## 30. Простой API-клиент

```python
from typing import Any

import requests


class UsersClient:
    def __init__(
        self,
        base_url: str,
        session: requests.Session,
        timeout: float = 5,
    ) -> None:
        self.base_url = base_url.rstrip("/")
        self.session = session
        self.timeout = timeout

    def get_user(self, user_id: int) -> requests.Response:
        return self.session.get(
            f"{self.base_url}/users/{user_id}",
            timeout=self.timeout,
        )

    def create_user(
        self,
        payload: dict[str, Any],
    ) -> requests.Response:
        return self.session.post(
            f"{self.base_url}/users",
            json=payload,
            timeout=self.timeout,
        )
```

Тест остаётся читаемым:

```python
def test_create_user(users_client, user_payload):
    response = users_client.create_user(user_payload)

    assert response.status_code == 201
    assert response.json()["email"] == user_payload["email"]
```

## 31. Базовый клиент и клиенты ресурсов

```python
from typing import Any

import requests


class BaseApiClient:
    def __init__(
        self,
        base_url: str,
        session: requests.Session,
        timeout: float = 5,
    ) -> None:
        self.base_url = base_url.rstrip("/")
        self.session = session
        self.timeout = timeout

    def request(
        self,
        method: str,
        path: str,
        **kwargs: Any,
    ) -> requests.Response:
        kwargs.setdefault("timeout", self.timeout)
        return self.session.request(
            method=method,
            url=f"{self.base_url}/{path.lstrip('/')}",
            **kwargs,
        )


class UsersClient:
    def __init__(self, api: BaseApiClient) -> None:
        self.api = api

    def get(self, user_id: int) -> requests.Response:
        return self.api.request("GET", f"/users/{user_id}")

    def create(self, payload: dict[str, Any]) -> requests.Response:
        return self.api.request("POST", "/users", json=payload)
```

Базовый клиент не должен превращаться в огромный класс со всей бизнес-логикой.

## 32. Что должен и не должен делать API-клиент

Клиент может:

- строить URL;
- добавлять общие headers;
- устанавливать timeout;
- отправлять запрос;
- логировать безопасные данные;
- выполнять общую техническую обработку.

Тест обычно должен:

- задавать сценарий;
- передавать тестовые данные;
- проверять ожидаемый status code;
- проверять бизнес-результат.

Если клиент сам делает все assertions, тесту становится трудно проверять негативные ответы.

## 33. Логирование request и response

При падении полезно видеть:

- метод;
- URL;
- request headers без секретов;
- request body;
- status code;
- response headers;
- response body;
- длительность;
- correlation ID.

```python
import logging


logger = logging.getLogger(__name__)


def log_response(response) -> None:
    logger.info(
        "%s %s -> %s in %s",
        response.request.method,
        response.request.url,
        response.status_code,
        response.elapsed,
    )
```

Не выводи в логи пароли, access tokens, refresh tokens, cookies и персональные данные.

## 34. Какие проверки делать в API-тесте

Типичный порядок:

1. status code;
2. `Content-Type`;
3. структура body;
4. обязательные поля;
5. типы;
6. конкретные бизнес-значения;
7. изменение состояния;
8. побочные эффекты.

```python
response = users_client.get(42)

assert response.status_code == 200
assert response.headers["Content-Type"].startswith(
    "application/json"
)

body = response.json()
assert body["id"] == 42
assert body["status"] == "active"
```

Не нужно проверять каждое поле во всех тестах. Выбирай проверки, значимые для сценария.

## 35. Почему одного status code недостаточно

Ответ `200` может содержать:

- неправильного пользователя;
- устаревшие данные;
- ошибку внутри JSON;
- пустой список вместо ожидаемых данных;
- неправильные типы;
- неверные права доступа.

Status code показывает общий результат HTTP-операции, но не заменяет проверку бизнес-содержимого.

## 36. Negative testing

Негативные проверки:

- отсутствует обязательное поле;
- поле имеет неверный тип;
- значение меньше или больше границы;
- строка слишком длинная;
- неизвестное поле;
- невалидный формат;
- ресурс не существует;
- пользователь не авторизован;
- пользователю запрещена операция;
- нарушено уникальное ограничение;
- некорректный `Content-Type`.

Проверяй не только ошибочный код, но и понятное стабильное описание ошибки.

## 37. Параметризация негативных тестов

```python
import pytest


@pytest.mark.parametrize(
    ("payload", "expected_field"),
    [
        ({}, "email"),
        ({"email": "not-an-email"}, "email"),
        ({"email": "a" * 300}, "email"),
    ],
    ids=[
        "missing-email",
        "invalid-email",
        "too-long-email",
    ],
)
def test_create_user_validation(
    users_client,
    payload,
    expected_field,
):
    response = users_client.create(payload)

    assert response.status_code == 422
    assert response.json()["field"] == expected_field
```

Параметризация хороша, когда шаги одинаковы, а меняются данные и ожидаемый результат.

## 38. Тестовые данные и builders

Плохо использовать один общий изменяемый словарь:

```python
USER = {"name": "Alex", "email": "alex@example.com"}
```

Один тест может изменить его и повлиять на другие.

Простой builder:

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

Каждый вызов создаёт новые данные.

## 39. Создание и очистка данных

```python
import pytest


@pytest.fixture
def created_user(users_client):
    payload = build_user_payload()
    create_response = users_client.create(payload)
    assert create_response.status_code == 201

    user = create_response.json()
    yield user

    users_client.delete(user["id"])
```

Очистка должна выполняться через `yield`/finalizer даже при падении теста.

При этом удаление не должно скрывать исходную ошибку. Ошибки teardown нужно логировать отдельно и понятно.

## 40. Проверка создания ресурса

Надёжный сценарий:

1. отправить `POST`;
2. проверить `201`;
3. проверить тело ответа;
4. получить созданный ресурс через `GET`;
5. сравнить важные поля;
6. удалить данные.

Так тест проверяет не только ответ `POST`, но и сохранённое состояние.

## 41. Проверка обновления ресурса

После `PUT` или `PATCH` проверь:

- status code;
- изменённое поле;
- неизменённые поля;
- `updated_at`, если это часть контракта;
- результат последующего `GET`;
- поведение при повторном запросе.

Для `PATCH` особенно важно убедиться, что неуказанные поля не сбросились.

## 42. Проверка удаления

После `DELETE`:

- проверить ожидаемый status code;
- выполнить `GET`;
- убедиться, что ресурс недоступен или помечен удалённым;
- проверить повторный `DELETE`;
- при необходимости проверить связанные данные.

Soft delete и hard delete ведут себя по-разному, поэтому ожидаемое поведение нужно брать из требований.

## 43. Виды авторизации

Частые варианты:

- Basic Auth;
- API Key;
- Bearer Token;
- OAuth 2.0;
- cookies/session;
- mTLS.

Пример Basic Auth:

```python
response = requests.get(
    url,
    auth=("username", "password"),
    timeout=5,
)
```

Bearer Token:

```python
headers = {"Authorization": f"Bearer {token}"}
response = requests.get(url, headers=headers, timeout=5)
```

## 44. `401` и `403`

- `401 Unauthorized` — клиент не прошёл аутентификацию: токена нет, он неверный или истёк;
- `403 Forbidden` — пользователь распознан, но у него нет права на действие.

Практические проверки:

- без токена;
- с невалидным токеном;
- с истёкшим токеном;
- с токеном другого пользователя;
- с правильным токеном, но неправильной ролью;
- доступ к чужому ресурсу.

## 45. Access token и refresh token

- access token используется для запросов к API и обычно живёт недолго;
- refresh token используется для получения нового access token и обычно защищается строже.

Нужно проверить:

- успешное обновление;
- истёкший refresh token;
- отозванный token;
- повторное использование, если оно запрещено;
- старый access token после logout или revoke;
- корректные права нового token.

Нельзя логировать содержимое токенов.

## 46. Валидация JSON Schema

```python
from jsonschema import validate


USER_SCHEMA = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "required": ["id", "name", "email"],
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string", "minLength": 1},
        "email": {"type": "string"},
    },
    "additionalProperties": False,
}


validate(
    instance=response.json(),
    schema=USER_SCHEMA,
)
```

`validate()` выбросит `ValidationError`, если данные не соответствуют схеме.

`format: email` не обязательно проверяется автоматически. Для проверки format нужно явно использовать `FormatChecker` и убедиться, что нужные зависимости установлены.

## 47. Что проверяет JSON Schema

JSON Schema может описывать:

- тип объекта;
- обязательные поля;
- типы полей;
- ограничения строк и чисел;
- элементы массива;
- допустимые значения;
- вложенные объекты;
- дополнительные поля.

Схема не заменяет бизнес-проверки. Например, тип `integer` не доказывает, что API вернул ID именно созданного пользователя.

## 48. Валидация ответа через Pydantic

```python
from pydantic import BaseModel, ConfigDict, EmailStr


class UserResponse(BaseModel):
    model_config = ConfigDict(extra="forbid")

    id: int
    name: str
    email: EmailStr


user = UserResponse.model_validate(response.json())

assert user.id > 0
assert user.name == "Alex"
```

Для API-тестов полезно явно выбрать политику дополнительных полей:

- `extra="ignore"` — лишние поля игнорируются;
- `extra="allow"` — сохраняются;
- `extra="forbid"` — приводят к ошибке.

По умолчанию Pydantic может преобразовывать некоторые значения. Если важны точные типы входа, используй strict mode или строгие типы.

## 49. JSON Schema и Pydantic — разница

| JSON Schema | Pydantic |
|---|---|
| Языконезависимая схема данных | Python-модели |
| Удобна для проверки формального контракта | Удобен для работы с типизированными данными |
| Может соответствовать OpenAPI | Может генерировать JSON Schema |
| Проверяет структуру | Валидирует и может преобразовывать данные |

На проекте можно использовать оба подхода. Выбор зависит от источника контракта и архитектуры тестов.

## 50. Что такое контрактное тестирование

Контракт определяет, как сервисы взаимодействуют:

- endpoint;
- метод;
- параметры;
- headers;
- request body;
- status codes;
- response body;
- схема;
- правила совместимости.

Контрактный тест проверяет, что provider и consumer одинаково понимают интерфейс.

Проверка одной JSON Schema — часть контрактного тестирования, но не весь контракт.

## 51. Backward compatibility API

Изменение считается потенциально ломающим, если:

- удалён endpoint;
- удалено или переименовано поле;
- изменён тип;
- обязательное поле добавлено в request;
- изменена семантика status code;
- сузился набор допустимых значений;
- изменилась авторизация.

Добавление необязательного response-поля обычно совместимо, но может сломать клиента, который запрещает дополнительные поля. Поэтому политика `extra="forbid"` должна быть осознанной.

## 52. Версионирование API

Варианты:

```text
/api/v1/users
```

```http
Accept: application/vnd.company.v2+json
```

Иногда версия передаётся через query parameter, но это встречается реже.

При тестировании новой версии важно проверить:

- старые клиенты;
- миграционный период;
- одинаковые правила авторизации;
- обратную совместимость;
- удаление deprecated-полей.

## 53. Пагинация

Частые виды:

- page/limit;
- offset/limit;
- cursor-based.

Проверки:

- размер страницы;
- первая и последняя страницы;
- пустая страница;
- значения `0`, отрицательные и слишком большие;
- отсутствие дублей между страницами;
- стабильность сортировки;
- корректный next cursor;
- поведение при изменении данных между запросами.

Нельзя полагаться на порядок без явно заданной сортировки.

## 54. Фильтрация, сортировка и поиск

Проверяй:

- один фильтр;
- несколько фильтров;
- комбинацию фильтра и сортировки;
- отсутствие результатов;
- регистр;
- специальные символы;
- пробелы;
- неизвестное поле;
- направление сортировки;
- `null`;
- стабильность порядка при одинаковых значениях.

Сравнивай весь полученный набор, а не только первый элемент.

## 55. Загрузка файлов

```python
from pathlib import Path


file_path = Path("test_data/report.pdf")

with file_path.open("rb") as file:
    response = requests.post(
        f"{base_url}/files",
        files={
            "file": (
                file_path.name,
                file,
                "application/pdf",
            )
        },
        timeout=10,
    )
```

Проверки:

- имя;
- размер;
- MIME type;
- содержимое;
- пустой файл;
- слишком большой файл;
- запрещённое расширение;
- одинаковые имена;
- вредоносное или повреждённое содержимое.

Файлы открывают в бинарном режиме.

## 56. Скачивание файлов

```python
import hashlib


response = requests.get(
    f"{base_url}/files/42",
    timeout=10,
)

assert response.status_code == 200
assert response.headers["Content-Type"] == "application/pdf"

checksum = hashlib.sha256(response.content).hexdigest()
assert checksum == expected_checksum
```

Для больших файлов используй streaming, чтобы не загружать всё содержимое в память.

## 57. Redirect

`requests` обычно следует redirect для `GET`.

```python
response = requests.get(
    url,
    allow_redirects=False,
    timeout=5,
)

assert response.status_code == 302
assert response.headers["Location"] == expected_url
```

При включённых redirect историю можно посмотреть через `response.history`.

Иногда тест должен проверять именно первый ответ, поэтому автоматический redirect нужно отключить.

## 58. Cookies

```python
with requests.Session() as session:
    login_response = session.post(
        f"{base_url}/login",
        json=credentials,
        timeout=5,
    )
    assert login_response.status_code == 200

    profile_response = session.get(
        f"{base_url}/profile",
        timeout=5,
    )
```

Session сохраняет cookies между запросами.

Проверки cookies могут включать:

- `Secure`;
- `HttpOnly`;
- `SameSite`;
- expiration;
- domain;
- path;
- удаление после logout.

## 59. Retry

По умолчанию `requests` не повторяет все неуспешные запросы автоматически.

Пример настройки:

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util import Retry


retry = Retry(
    total=3,
    backoff_factor=0.5,
    status_forcelist=[429, 502, 503, 504],
    allowed_methods={"GET", "HEAD", "OPTIONS"},
    respect_retry_after_header=True,
)

session = requests.Session()
session.mount("https://", HTTPAdapter(max_retries=retry))
session.mount("http://", HTTPAdapter(max_retries=retry))
```

Retry не должен скрывать стабильный дефект.

## 60. Почему опасно повторять POST

Сервер мог создать ресурс, но клиент не получил ответ из-за сетевого сбоя. Повторный `POST` создаст дубль.

Безопасные варианты:

- не повторять неидемпотентный запрос автоматически;
- использовать `Idempotency-Key`;
- после неопределённого результата проверить состояние;
- применять бизнес-ключ уникальности;
- понимать гарантии конкретного API.

Добавлять `POST` в `allowed_methods` можно только при понятной идемпотентности операции.

## 61. Rate limiting

Сервис может ограничивать количество запросов.

Частое поведение:

- ответ `429 Too Many Requests`;
- header `Retry-After`;
- headers с лимитом и остатком;
- сброс лимита через определённое время.

Проверяй:

- границу лимита;
- разные токены или пользователей;
- разные endpoints;
- восстановление после окна;
- отсутствие частично выполненной операции;
- корректный `Retry-After`.

Не запускай агрессивный тест против общего или production-окружения без согласования.

## 62. Асинхронная операция и polling

API может вернуть `202 Accepted`, а обработка завершится позже.

```python
import time


deadline = time.monotonic() + 30

while time.monotonic() < deadline:
    response = jobs_client.get(job_id)
    assert response.status_code == 200

    status = response.json()["status"]
    if status == "completed":
        break
    if status == "failed":
        raise AssertionError("Job failed")

    time.sleep(1)
else:
    raise AssertionError("Job did not complete in 30 seconds")
```

Это контролируемый polling:

- есть общий deadline;
- есть интервал;
- обрабатывается failed state;
- ошибка объясняет причину.

Один фиксированный `sleep(30)` хуже: он либо тормозит, либо не дожидается.

## 63. Eventual consistency

В распределённой системе запись может быть принята одним сервисом, а в другом появиться позже.

Тест должен:

- знать, какое состояние станет конечным;
- ждать конкретное условие;
- иметь deadline;
- логировать промежуточные статусы;
- отличать задержку от окончательной ошибки.

Не нужно ждать eventual consistency там, где система обещает строгую синхронность: это может скрыть дефект.

## 64. Webhooks

Webhook — HTTP-вызов, который система отправляет внешнему получателю при событии.

Проверки:

- правильный URL;
- метод;
- headers;
- подпись;
- payload;
- повторная доставка;
- порядок событий;
- таймаут получателя;
- ответы `2xx`, `4xx`, `5xx`;
- защита от дубликатов;
- DLQ или журнал недоставленных событий.

В тестовой среде нужен управляемый endpoint-приёмник, который сохраняет полученные вызовы.

## 65. Моки и service virtualization

Мок внешнего сервиса полезен, когда нужно:

- воспроизвести редкую ошибку;
- вернуть конкретный status code;
- проверить timeout;
- убрать нестабильную зависимость;
- проверить отправленный request;
- сделать тест детерминированным.

Но мок подтверждает поведение против модели сервиса, а не реальную интеграцию. Поэтому нужны и отдельные интеграционные проверки с настоящей зависимостью.

## 66. Что именно мокировать

Есть два разных уровня:

1. Замокировать вызов Python-функции, чтобы протестировать свой код без сети.
2. Поднять HTTP stub server, чтобы проверить настоящий HTTP-клиент.

Первый вариант быстрее, но не проверяет:

- URL;
- сериализацию;
- headers;
- timeout;
- обработку реального HTTP-ответа.

Для теста API-клиента чаще полезнее stub server.

## 67. Проверка побочных эффектов

После API-запроса результат может появиться:

- в БД;
- в очереди;
- в другом сервисе;
- в email;
- в журнале аудита;
- в файловом хранилище.

Проверять каждый внутренний слой во всех тестах не нужно. Выбирай наблюдаемый результат, важный для сценария.

Если тест напрямую проверяет БД, он сильнее связан с реализацией и требует аккуратной очистки данных.

## 68. API и база данных

Пример сценария:

1. создать сущность через API;
2. получить её через API;
3. при необходимости проверить критичное поле в БД;
4. удалить данные через API или служебный fixture.

Не стоит подготавливать абсолютно всё прямыми `INSERT`, если тест должен проверить публичный процесс создания.

Прямой доступ к БД полезен для:

- сложной подготовки;
- очистки;
- проверки миграции;
- диагностики;
- данных, которые нельзя получить через API.

## 69. Параллельный запуск API-тестов

Для параллельности нужны:

- уникальные данные;
- независимые пользователи;
- отдельные ресурсы;
- отсутствие зависимости от порядка;
- безопасная очистка;
- отсутствие общего изменяемого файла;
- учёт rate limit.

Не используй один email, один заказ или одну корзину во всех worker-процессах.

## 70. Конфигурация окружений

Конфигурация может включать:

- `base_url`;
- timeout;
- credentials;
- feature flags;
- адрес БД;
- адрес брокера;
- включение логирования.

```python
import os


BASE_URL = os.environ["API_BASE_URL"]
API_TOKEN = os.environ["API_TOKEN"]
```

Секреты не должны храниться в репозитории.

Тест должен явно падать при отсутствии обязательной настройки, а не незаметно использовать production URL или случайное значение.

## 71. HTTPX

HTTPX имеет похожий интерфейс и поддерживает sync и async API.

Sync:

```python
import httpx


with httpx.Client(
    base_url="https://api.example.com",
    timeout=5,
) as client:
    response = client.get("/users/42")

assert response.status_code == 200
```

Async:

```python
import httpx


async def get_user(user_id: int) -> dict:
    async with httpx.AsyncClient(
        base_url="https://api.example.com",
        timeout=5,
    ) as client:
        response = await client.get(f"/users/{user_id}")
        response.raise_for_status()
        return response.json()
```

Не создавай новый `AsyncClient` внутри каждой итерации большого цикла: клиент нужен для переиспользования соединений.

## 72. Requests и HTTPX — разница

| Requests | HTTPX |
|---|---|
| Простой зрелый sync-клиент | Sync и async API |
| HTTP/1.1 | HTTP/1.1 и опциональный HTTP/2 |
| `requests.Session` | `httpx.Client` или `AsyncClient` |
| Нет timeout по умолчанию | Есть timeout по умолчанию |
| Очень распространён | Удобен для async-проектов |

Выбор зависит от проекта. Для обычного синхронного pytest-фреймворка `requests` часто достаточно.

## 73. Как тестировать GraphQL

GraphQL-запрос обычно отправляется через `POST`:

```python
query = """
query User($id: ID!) {
  user(id: $id) {
    id
    name
  }
}
"""

response = requests.post(
    f"{base_url}/graphql",
    json={
        "query": query,
        "variables": {"id": "42"},
    },
    timeout=5,
)
```

У GraphQL HTTP status может быть `200`, даже если внутри `errors` есть ошибка. Поэтому нужно проверять:

- поле `errors`;
- поле `data`;
- типы и значения;
- права на поля;
- неизвестные поля;
- сложность и глубину запроса;
- batching, если он поддерживается.

## 74. Как тестировать gRPC

gRPC использует Protocol Buffers и обычно HTTP/2.

Проверяются:

- request/response messages;
- gRPC status;
- metadata;
- deadlines;
- unary и streaming calls;
- backward compatibility `.proto`;
- обработка недоступного сервиса;
- retries;
- авторизация.

HTTP status codes и gRPC status — не одно и то же.

Для Python обычно используется сгенерированный client stub.

## 75. Как тестировать SOAP

SOAP использует XML-сообщения и контракт WSDL.

Проверяются:

- XML structure;
- namespaces;
- обязательные элементы;
- SOAP headers;
- SOAP Fault;
- соответствие WSDL/XSD;
- авторизация;
- бизнес-результат.

Не следует сравнивать XML как обычную строку: порядок атрибутов и форматирование могут отличаться без изменения смысла.

## 76. Как тестировать API security на базовом уровне

Минимальные проверки:

- доступ без аутентификации;
- доступ с другой ролью;
- доступ к чужому объекту;
- изменение защищённых полей;
- массовое присваивание;
- injection;
- утечка секретов в ответе;
- подробные stack traces;
- rate limiting;
- небезопасные CORS-настройки;
- передача токена только по HTTPS;
- отсутствие чувствительных данных в логах.

Security-тестирование нельзя сводить к отправке случайных строк. Нужны модель угроз и разрешённый scope.

## 77. Проверка производительности API

Обычный functional test может проверить грубое ограничение:

```python
assert response.elapsed.total_seconds() < 2
```

Но это не нагрузочный тест: одиночный запрос зависит от CI и сети.

Для performance testing нужны:

- ожидаемая нагрузка;
- количество пользователей;
- профиль запросов;
- ramp-up;
- длительность;
- percentiles, например p95 и p99;
- error rate;
- инфраструктурные метрики;
- согласованное окружение.

Жёсткие performance assertions в обычном CI часто дают flaky-результат.

## 78. Частые причины flaky API-тестов

- общие тестовые данные;
- зависимость от порядка;
- отсутствие timeout;
- слишком короткий timeout;
- фиксированный `sleep`;
- нестабильная внешняя система;
- eventual consistency без polling;
- автоматический retry скрывает ошибку;
- неправильная очистка;
- одинаковые пользователи при параллельном запуске;
- зависимость от текущего времени;
- неконтролируемый rate limit.

Сначала собирают факты, затем исправляют причину. Бесконечные reruns — не решение.

## 79. Как диагностировать падение API-теста

Порядок:

1. Посмотреть метод, URL и окружение.
2. Проверить status code.
3. Посмотреть response body.
4. Проверить request body и headers без секретов.
5. Найти correlation ID.
6. Проверить логи сервиса.
7. Посмотреть связанные вызовы, БД или брокер.
8. Повторить запрос через `curl`, если это помогает изолировать клиент.
9. Определить: проблема теста, данных, окружения или продукта.

Не начинай с увеличения timeout, пока не выяснено, что именно зависло.

## 80. Типовая структура API-проекта

```text
project/
  clients/
    base_client.py
    users_client.py
    orders_client.py
  models/
    user.py
    order.py
  schemas/
    user.json
  data/
    builders.py
  assertions/
    api_assertions.py
  tests/
    api/
      test_users.py
      test_orders.py
  conftest.py
  pyproject.toml
```

Принцип:

- clients отправляют запросы;
- models/schemas описывают данные;
- builders создают тестовые данные;
- fixtures управляют lifecycle;
- tests описывают сценарии;
- assertions остаются понятными и не скрывают смысл.

## 81. Что хранить в `conftest.py`

Подходящие вещи:

- чтение конфигурации;
- `requests.Session`;
- авторизация;
- создание API-клиентов;
- общие fixtures;
- cleanup.

Не нужно складывать в один `conftest.py` весь код проекта. Бизнес-клиенты и модели лучше хранить в отдельных модулях.

## 82. Helper assertions

```python
def assert_error(
    response,
    *,
    status_code: int,
    error_code: str,
) -> None:
    assert response.status_code == status_code

    body = response.json()
    assert body["error"]["code"] == error_code
    assert body["error"]["message"]
```

Helper полезен для повторяющегося формата.

Плохо:

```python
def assert_response_is_correct(response):
    ...
```

Название не объясняет, что именно проверяется, а слишком большой helper скрывает сценарий.

## 83. Когда использовать прямые assertions

Прямые проверки лучше, если они короткие и уникальны:

```python
assert response.status_code == 200
assert response.json()["status"] == "paid"
```

Выносить каждую строку в helper не нужно.

Главная цель — чтобы по тесту было понятно:

- что сделали;
- что ожидали;
- где упало.

## 84. Частые ошибки в API-автоматизации

1. Нет timeout.
2. `requests.get()` разбросан по всем тестам.
3. Проверяется только status code.
4. Все ответы проверяются одной огромной схемой.
5. Клиент автоматически падает на любой `4xx`, мешая негативным тестам.
6. Токены и пароли пишутся в логи.
7. Один пользователь используется во всех тестах.
8. Retry включён для любого метода.
9. Тест зависит от результата предыдущего теста.
10. Cleanup выполняется только в конце успешного теста.
11. Мок полностью заменяет настоящую интеграцию.
12. В тесте смешаны HTTP, SQL, UI и десятки бизнес-шагов.

## Дополнение. TLS-сертификаты и `verify=False`

По умолчанию `requests` проверяет TLS-сертификат HTTPS-сервера.

Если стенд использует внутренний центр сертификации, правильнее передать доверенный CA bundle:

```python
response = requests.get(
    url,
    verify="certs/company-ca.pem",
    timeout=5,
)
```

```python
requests.get(
    url,
    verify=False,
    timeout=5,
)
```

`verify=False` отключает проверку сертификата. Это может временно помочь локализовать проблему на тестовом стенде, но не является нормальным исправлением: клиент перестаёт подтверждать подлинность сервера.

При TLS-ошибке нужно проверить:

- срок действия сертификата;
- hostname;
- цепочку доверия;
- наличие intermediate certificates;
- используемый CA bundle;
- системное время;
- proxy, который может подменять сертификат.

## 85. Частые вопросы на собеседовании

### Что проверять у API endpoint?

Метод, URL, авторизацию, status code, headers, body, schema, бизнес-правила, ошибки, изменение состояния, идемпотентность и побочные эффекты.

### Чем `json=` отличается от `data=`?

`json=` сериализует Python-объект в JSON и устанавливает подходящий `Content-Type`. Словарь в `data=` обычно отправляется как form data.

### Зачем нужен timeout?

Чтобы клиент не ждал ответ неограниченно долго. У `requests` timeout по умолчанию не задан.

### Зачем нужен `Session`?

Для общих headers/auth, сохранения cookies и переиспользования соединений.

### Почему нельзя автоматически retry любой POST?

Первый запрос мог выполниться, а ответ потеряться. Повтор создаст дубль или повторит побочный эффект.

### Чем `401` отличается от `403`?

`401` — нет корректной аутентификации. `403` — пользователь известен, но операция ему запрещена.

### JSON Schema и Pydantic — одно и то же?

Нет. JSON Schema — языконезависимое описание структуры. Pydantic — Python-модели с runtime validation и возможным преобразованием данных.

### Что такое контрактный тест?

Проверка того, что взаимодействующие стороны одинаково понимают формат и семантику API.

### Что такое polling?

Периодическая проверка статуса асинхронной операции до конечного состояния или deadline.

### Чем mock отличается от настоящей интеграции?

Mock проверяет поведение против заданной модели ответа. Настоящая интеграция проверяет реальное взаимодействие систем.

## 86. Практические задания

На собеседовании могут попросить:

- написать `GET`-тест;
- создать ресурс и проверить его через `GET`;
- параметризовать негативные данные;
- написать fixture с авторизацией;
- реализовать API-клиент;
- проверить JSON Schema;
- описать cleanup;
- реализовать polling;
- настроить безопасный retry;
- разобрать падение `502` или timeout;
- протестировать пагинацию;
- объяснить параллельный запуск.

## 87. Сильный ответ: как построить API-автоматизацию

> Я отделяю тестовые сценарии от HTTP-транспорта. Общий клиент отвечает за base URL, session, timeout, безопасное логирование и отправку запросов, а клиенты ресурсов дают методы уровня users или orders. Pytest-фикстуры управляют авторизацией, данными и cleanup. Тесты проверяют status code, контракт и бизнес-результат, остаются независимыми и используют уникальные данные. Для асинхронных процессов применяю polling с deadline, а retry включаю только для безопасных или явно идемпотентных операций.

## 88. Что повторить в первую очередь

1. Структура HTTP request/response.
2. Методы, status codes и идемпотентность.
3. `params`, `json`, `data`, headers.
4. `Response`, timeout и исключения.
5. `Session`.
6. API client layer.
7. Negative testing.
8. Авторизация и `401`/`403`.
9. JSON Schema и Pydantic.
10. Retry и `Idempotency-Key`.
11. Polling и eventual consistency.
12. Изоляция, данные и cleanup.

## Официальные источники

- [Requests: Quickstart](https://requests.readthedocs.io/en/stable/user/quickstart/)
- [Requests: Advanced Usage](https://requests.readthedocs.io/en/stable/user/advanced/)
- [urllib3: Retry](https://urllib3.readthedocs.io/en/stable/reference/urllib3.util.html)
- [pytest: fixtures](https://docs.pytest.org/en/stable/how-to/fixtures.html)
- [Pydantic: models](https://docs.pydantic.dev/latest/concepts/models/)
- [jsonschema: API](https://python-jsonschema.readthedocs.io/en/stable/api/)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [HTTPX: QuickStart](https://www.python-httpx.org/quickstart/)
- [HTTPX: Clients](https://www.python-httpx.org/advanced/clients/)
- [HTTPX: Async Support](https://www.python-httpx.org/async/)
