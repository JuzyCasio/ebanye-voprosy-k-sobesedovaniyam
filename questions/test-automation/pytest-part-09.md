# Pytest — полный конспект — часть 9

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 54-62.

## 54. Как объяснить fixture vs setup_method

В pytest можно использовать xUnit-style методы:  

class TestUser:  
    def setup_method(self) -> None:  
        self.user = {"name": "Alex"}  

    def test_user_name(self) -> None:  
        assert self.user["name"] == "Alex"  

Но чаще лучше фикстуры:  

import pytest  

@pytest.fixture  
def user() -> dict[str, str]:  
    return {"name": "Alex"}  

def test_user_name(user: dict[str, str]) -> None:  
    assert user["name"] == "Alex"  

### На собеседовании:

xUnit-style setup/teardown поддерживается, но фикстуры гибче: у них есть scope, dependency injection, переиспользование, параметризация и teardown через yield.  

## 55. Как работает порядок setup/teardown фикстур

### Пример:

import pytest  

@pytest.fixture  
def first():  
    print("setup first")  
    yield  
    print("teardown first")  

@pytest.fixture  
def second(first):  
    print("setup second")  
    yield  
    print("teardown second")  

def test_example(second):  
    print("test")  

Логика:  

setup first  
setup second  
test  
teardown second  
teardown first  

### На собеседовании:

Teardown идёт в обратном порядке setup. Если фикстура зависит от другой, сначала будет создана зависимость.  

## 56. Как работать с внешними сервисами

### Подходы:

1. Реальный сервис на тестовом окружении.  
2. Мок через monkeypatch/mock.  
3. Stub-сервис.  
4. Fake-сервис.  
5. Contract testing.  
6. Запуск зависимости в Docker.  

### На собеседовании:

Если проверяем интеграцию — нужен реальный сервис или стабильный test env. Если проверяем бизнес-логику нашего сервиса — внешнюю зависимость лучше замокать или заменить stub/fake.  

## 57. Пример мок-сервиса

class FakePaymentService:  
    def __init__(self) -> None:  
        self.payments: list[dict] = []  

    def pay(self, user_id: int, amount: int) -> dict:  
        payment = {  
            "user_id": user_id,  
            "amount": amount,  
            "status": "success",  
        }  
        self.payments.append(payment)  
        return payment  

Тест:  

def test_order_payment() -> None:  
    payment_service = FakePaymentService()  
    order_service = OrderService(payment_service=payment_service)  

    order = order_service.create_paid_order(user_id=1, amount=100)  

    assert order["payment_status"] == "success"  

## 58. API client layer

### Хорошая практика — не писать requests.get прямо в каждом тесте.

### Плохо:

def test_get_user(base_url) -> None:  
    response = requests.get(f"{base_url}/users/1")  
    assert response.status_code == 200  

Лучше:  

class UserApiClient:  
    def __init__(self, base_url: str) -> None:  
        self.base_url = base_url  

    def get_user(self, user_id: int):  
        return requests.get(f"{self.base_url}/users/{user_id}", timeout=5)  

    def create_user(self, payload: dict):  
        return requests.post(f"{self.base_url}/users", json=payload, timeout=5)  

Тест:  

def test_get_user(user_api_client: UserApiClient) -> None:  
    response = user_api_client.get_user(user_id=1)  

    assert response.status_code == 200  

### На собеседовании:

API client layer уменьшает дублирование, централизует base_url, headers, auth, timeout, logging и обработку response.  

## 59. Проверка негативных сценариев

### Пример:

import pytest  

@pytest.mark.parametrize(  
    "payload, expected_error",  
    [  
        ({}, "username_required"),  
        ({"username": ""}, "username_empty"),  
        ({"username": "alex"}, "password_required"),  
    ],  
)  
def test_create_user_invalid_payload(api_client, payload: dict, expected_error: str) -> None:  
    response = api_client.create_user(payload)  

    assert response.status_code == 400  

    body = response.json()  

    assert body["error_type"] == expected_error  

### На собеседовании:

В негативных тестах важно проверять не только статус, но и понятную ошибку: code/error_type/message/details.  

## 60. Timeout в API-тестах

### Плохо:

requests.get(url)  

Лучше:  

requests.get(url, timeout=5)  

### На собеседовании:

В тестовом фреймворке обязательно ставлю timeout на сетевые запросы, чтобы тесты не зависали бесконечно.  

## 61. Логирование request/response

import logging  

logger = logging.getLogger(__name__)  

class ApiClient:  
    def __init__(self, base_url: str) -> None:  
        self.base_url = base_url  

    def post(self, path: str, json: dict):  
        url = f"{self.base_url}{path}"  

        logger.info("POST %s payload=%s", url, json)  

        response = requests.post(url, json=json, timeout=5)  

        logger.info(  
            "Response status=%s body=%s",  
            response.status_code,  
            response.text,  
        )  

        return response  

### На собеседовании:

При падении API-теста нужны request, response, status code, headers, body, correlation id. Это сильно ускоряет разбор.  

## 62. Как оформлять helper assertions

def assert_error_response(response, expected_status: int, expected_error_type: str) -> None:  
    assert response.status_code == expected_status  

    body = response.json()  

    assert body["error_type"] == expected_error_type  

Использование:  

def test_create_duplicate_user(api_client, existing_user) -> None:  
    response = api_client.create_user(existing_user)  

    assert_error_response(  
        response=response,  
        expected_status=409,  
        expected_error_type="user_already_exists",  
    )  

### На собеседовании:

Повторяющиеся проверки можно выносить в helper assertions, но не надо прятать всю суть теста. Тест должен оставаться читаемым.  

