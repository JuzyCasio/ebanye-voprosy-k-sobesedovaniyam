# Pytest — полный конспект — часть 13

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 82-86.

## 82. Мини-шаблон тестового фреймворка

tests/  
├── conftest.py  
├── api/  
│   ├── clients/  
│   │   └── user_client.py  
│   ├── test_users.py  
│   └── test_orders.py  
├── data/  
│   └── user_payloads.py  
├── helpers/  
│   └── assertions.py  
└── factories/  
    └── user_factory.py  

clients/user_client.py:  

import requests  

class UserClient:  
    def __init__(self, base_url: str, token: str | None = None) -> None:  
        self.base_url = base_url  
        self.token = token  

    def _headers(self) -> dict[str, str]:  
        headers = {"Content-Type": "application/json"}  

        if self.token:  
            headers["Authorization"] = f"Bearer {self.token}"  

        return headers  

    def create_user(self, payload: dict) -> requests.Response:  
        return requests.post(  
            f"{self.base_url}/users",  
            json=payload,  
            headers=self._headers(),  
            timeout=5,  
        )  

    def get_user(self, user_id: int) -> requests.Response:  
        return requests.get(  
            f"{self.base_url}/users/{user_id}",  
            headers=self._headers(),  
            timeout=5,  
        )  

conftest.py:  

import pytest  

from tests.api.clients.user_client import UserClient  

def pytest_addoption(parser) -> None:  
    parser.addoption("--base-url", action="store", default="https://api.dev.example.com")  
    parser.addoption("--token", action="store", default=None)  

@pytest.fixture(scope="session")  
def base_url(pytestconfig) -> str:  
    return pytestconfig.getoption("--base-url")  

@pytest.fixture(scope="session")  
def token(pytestconfig) -> str | None:  
    return pytestconfig.getoption("--token")  

@pytest.fixture  
def user_client(base_url: str, token: str | None) -> UserClient:  
    return UserClient(base_url=base_url, token=token)  

factories/user_factory.py:  

from uuid import uuid4  

def build_user_payload(  
    username: str | None = None,  
    password: str = "Qwerty123",  
) -> dict[str, str]:  
    return {  
        "username": username or f"user_{uuid4().hex}",  
        "password": password,  
    }  

helpers/assertions.py:  

def assert_error_response(response, expected_status: int, expected_error_type: str) -> None:  
    assert response.status_code == expected_status  

    body = response.json()  

    assert body["error_type"] == expected_error_type  

test_users.py:  

import pytest  

from tests.factories.user_factory import build_user_payload  
from tests.helpers.assertions import assert_error_response  

def test_create_user_with_valid_payload(user_client) -> None:  
    payload = build_user_payload()  

    response = user_client.create_user(payload)  

    assert response.status_code == 201  

    body = response.json()  

    assert isinstance(body["id"], int)  
    assert body["username"] == payload["username"]  

@pytest.mark.parametrize(  
    "payload, expected_error",  
    [  
        ({}, "username_required"),  
        ({"username": ""}, "username_empty"),  
        ({"username": "alex", "password": "short"}, "password_invalid"),  
    ],  
)  
def test_create_user_with_invalid_payload(  
    user_client,  
    payload: dict,  
    expected_error: str,  
) -> None:  
    response = user_client.create_user(payload)  

    assert_error_response(  
        response=response,  
        expected_status=400,  
        expected_error_type=expected_error,  
    )  

## 83. Что сказать, если спросят «как бы ты построил pytest-фреймворк?»

### Хороший ответ:

Я бы разделил проект на слои. В тестах оставил бы только сценарии и проверки. Работу с API вынес бы в client layer, генерацию данных — в factories/builders, общие проверки — в helpers, подготовку окружения — в fixtures внутри conftest.py. Для запуска добавил бы CLI-опции вроде --base-url, --env, --browser. Для группировки использовал бы markers: smoke, regression, slow. Для отчётности подключил бы Allure, а в CI сохранял бы allure-results, логи и request/response attachments.  

## 84. Что сказать, если спросят «как бороться с flaky?»

Ответ:  

Сначала нужно понять причину, а не просто добавить rerun. Я бы проверил изоляцию данных, порядок запуска, параллельность, внешние зависимости, ожидания, timeout, текущую дату/время и артефакты. Потом добавил бы уникальные тестовые данные, явные ожидания, cleanup, моки или стабилизировал test environment. Rerun — только временная мера.  

## 85. Что сказать, если спросят «как тесты запускаются в CI?»

Ответ:  

В CI обычно есть несколько уровней запуска. На merge request — быстрые smoke/API/unit. По расписанию — regression. Перед релизом — полный набор. Тесты запускаются командой pytest с нужными маркерами и параметрами окружения. После запуска сохраняются отчёты: Allure results, логи, скриншоты, request/response, coverage.  

## 86. Что сказать, если спросят «как ускорить автотесты?»

Ответ:  

Сначала измерить: --durations, отчёты CI, Allure timeline. Потом разделить тесты по уровням и маркерам, убрать лишние end-to-end проверки, параллелить через xdist, переиспользовать дорогие ресурсы, оптимизировать подготовку данных, заменить sleep на ожидания и мокать внешние зависимости там, где не проверяется интеграция.  

