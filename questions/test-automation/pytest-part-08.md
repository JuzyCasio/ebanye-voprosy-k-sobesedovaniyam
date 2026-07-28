# Pytest — полный конспект — часть 8

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 47-53.

## 47. Частая ошибка: mutable state в фикстуре

### Плохо:

import pytest  

@pytest.fixture(scope="session")  
def shared_list() -> list[int]:  
    return []  

def test_one(shared_list: list[int]) -> None:  
    shared_list.append(1)  
    assert shared_list == [1]  

def test_two(shared_list: list[int]) -> None:  
    assert shared_list == []  

test_two может упасть, потому что список общий на всю сессию.  

### Хорошо:

import pytest  

@pytest.fixture  
def empty_list() -> list[int]:  
    return []  

### На собеседовании:

Нужно аккуратно использовать изменяемые объекты в широких scope, потому что тесты могут влиять друг на друга.  

## 48. Как ускорять pytest-тесты

Ответ на собеседовании:  

1. Параллелить через pytest-xdist.  
2. Разделить тесты по маркерам: smoke/regression/slow.  
3. Убрать лишние UI-тесты, часть проверок перенести на API/unit.  
4. Переиспользовать дорогие ресурсы через session fixtures.  
5. Оптимизировать подготовку данных.  
6. Убрать sleep, заменить на ожидания.  
7. Использовать test selection: -k, -m, changed tests.  
8. Не ходить во внешние сервисы там, где можно mock/stub.  
9. Анализировать самые медленные тесты через --durations.  
10. Запускать разные группы тестов в разных CI jobs.  

## 49. Как выбирать тесты для запуска

pytest -m smoke  

По marker.  

pytest -k "login"  

По имени.  

pytest tests/api  

По директории.  

pytest tests/api/test_users.py::test_create_user  

Конкретный тест.  

pytest --lf  

Только прошлые падения.  

pytest --ff  

Сначала прошлые падения.  

### На собеседовании:

В CI я бы запускал smoke на каждый merge request, regression — по расписанию или перед релизом, а тяжёлые e2e/performance — отдельно.  

## 50. Тестовые данные

### Подходы:

1. Inline данные прямо в parametrize.  
2. Фабрики.  
3. Faker.  
4. JSON/YAML fixtures.  
5. Builder pattern.  
6. Создание данных через API.  
7. Создание данных напрямую в БД.  
8. Предзагруженный seed.  

### Пример builder:

from dataclasses import dataclass, field  
from uuid import uuid4  

@dataclass  
class UserPayloadBuilder:  
    username: str = field(default_factory=lambda: f"user_{uuid4().hex}")  
    password: str = "Qwerty123"  

    def with_username(self, username: str) -> "UserPayloadBuilder":  
        self.username = username  
        return self  

    def with_password(self, password: str) -> "UserPayloadBuilder":  
        self.password = password  
        return self  

    def build(self) -> dict[str, str]:  
        return {  
            "username": self.username,  
            "password": self.password,  
        }  

Тест:  

def test_create_user(api_client) -> None:  
    payload = UserPayloadBuilder().with_username("alex").build()  

    response = api_client.create_user(payload)  

    assert response.status_code == 201  

### На собеседовании:

Для API-тестов я люблю factory/builder подход: тестовые данные читаемые, переиспользуемые и легко варьируются.  

## 51. Где хранить тестовые данные

### Плохо:

def test_create_user() -> None:  
    payload = {  
        "username": "test_user_1",  
        "password": "123",  
        "email": "test@test.com",  
        "phone": "123",  
        # огромный JSON на 200 строк  
    }  

Лучше:  

def test_create_user(user_payload_factory, api_client) -> None:  
    payload = user_payload_factory(username="alex")  

    response = api_client.create_user(payload)  

    assert response.status_code == 201  

### На собеседовании:

Если данные маленькие — можно держать прямо в тесте. Если данные сложные и переиспользуются — лучше фабрики, билдеры или отдельные test data modules.  

## 52. Хороший тест в pytest

### Хороший тест:

1. Понятное имя.  
2. Один основной сценарий.  
3. Явная подготовка данных.  
4. Понятное действие.  
5. Понятные проверки.  
6. Не зависит от других тестов.  
7. Убирает за собой данные.  
8. Даёт полезную диагностику при падении.  

### Пример:

def test_create_user_with_valid_payload_returns_created_user(api_client, user_payload_factory) -> None:  
    payload = user_payload_factory(username="alex")  

    response = api_client.create_user(payload)  

    assert response.status_code == 201  

    body = response.json()  

    assert body["username"] == payload["username"]  
    assert isinstance(body["id"], int)  

## 53. Плохой тест

def test_1(api_client) -> None:  
    r = api_client.create_user({"u": "a"})  
    assert r.status_code == 200 or r.status_code == 201  

### Что плохо:

1. Непонятное имя.  
2. Непонятные данные.  
3. Неясно, какой статус ожидается.  
4. Слишком мягкий assert.  
5. Нет проверки тела ответа.  

Лучше:  

def test_create_user_with_valid_payload_returns_201(api_client, user_payload_factory) -> None:  
    payload = user_payload_factory()  

    response = api_client.create_user(payload)  

    assert response.status_code == 201  

