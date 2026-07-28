# Pytest — полный конспект — часть 6

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 31-39.

## 31. Проверка схемы ответа

### Пример без внешних библиотек:

def assert_user_schema(body: dict) -> None:  
    assert isinstance(body["id"], int)  
    assert isinstance(body["username"], str)  
    assert isinstance(body["is_active"], bool)  

def test_get_user(api_client) -> None:  
    response = api_client.get_user(user_id=1)  

    assert response.status_code == 200  

    body = response.json()  
    assert_user_schema(body)  

С jsonschema:  

from jsonschema import validate  

USER_SCHEMA = {  
    "type": "object",  
    "required": ["id", "username", "is_active"],  
    "properties": {  
        "id": {"type": "integer"},  
        "username": {"type": "string"},  
        "is_active": {"type": "boolean"},  
    },  
}  

def test_get_user_schema(api_client) -> None:  
    response = api_client.get_user(user_id=1)  

    assert response.status_code == 200  

    validate(instance=response.json(), schema=USER_SCHEMA)  

## 32. Работа с БД в pytest

### Типичный подход:

1. Поднять тестовую БД.  
2. Накатить миграции.  
3. Перед тестом подготовить данные.  
4. После теста откатить транзакцию или удалить данные.  
5. Не использовать продовую БД.  

### Пример фикстуры с rollback:

import pytest  

@pytest.fixture  
def db_session():  
    session = create_db_session()  
    transaction = session.begin()  

    yield session  

    transaction.rollback()  
    session.close()  

Тест:  

def test_user_saved_to_db(db_session) -> None:  
    user = User(name="Alex")  

    db_session.add(user)  
    db_session.flush()  

    assert user.id is not None  

### На собеседовании:

Для БД-тестов важно изолировать данные. Обычно используют транзакции с rollback, отдельные схемы, временные таблицы или отдельные БД на worker при параллельном запуске.  

## 33. Моки

Через стандартный unittest.mock:  

from unittest.mock import Mock  

def test_send_email() -> None:  
    email_sender = Mock()  
    service = UserService(email_sender=email_sender)  

    service.register_user("alex@example.com")  

    email_sender.send.assert_called_once_with("alex@example.com")  

Через monkeypatch:  

def test_external_service(monkeypatch) -> None:  
    def fake_get_rate() -> float:  
        return 100.0  

    monkeypatch.setattr("app.currency.get_rate", fake_get_rate)  

    assert calculate_price(10) == 1000.0  

### На собеседовании:

Моки нужны, чтобы изолировать тестируемую логику от внешних зависимостей: сети, БД, брокеров, файловой системы, времени, сторонних API.  

## 34. Тестирование исключений

import pytest  

def divide(a: int, b: int) -> float:  
    if b == 0:  
        raise ValueError("division by zero")  

    return a / b  

def test_divide_by_zero() -> None:  
    with pytest.raises(ValueError, match="division by zero"):  
        divide(10, 0)  

### На собеседовании:

pytest.raises проверяет, что код выбрасывает ожидаемое исключение. Через match можно проверить текст ошибки.  

## 35. Проверка логов

import logging  

def create_user(name: str) -> None:  
    logging.info("Creating user %s", name)  

def test_create_user_logs(caplog) -> None:  
    with caplog.at_level(logging.INFO):  
        create_user("Alex")  

    assert "Creating user Alex" in caplog.text  

### На собеседовании:

caplog полезен, когда часть поведения выражается через логи: ошибки интеграций, audit events, retry, fallback.  

## 36. Проверка print/stdout

def greet(name: str) -> None:  
    print(f"Hello, {name}")  

def test_greet(capsys) -> None:  
    greet("Alex")  

    captured = capsys.readouterr()  

    assert captured.out == "Hello, Alex\n"  

## 37. tmp_path для файлов

def test_report_created(tmp_path) -> None:  
    report_path = tmp_path / "report.txt"  

    report_path.write_text("OK")  

    assert report_path.exists()  
    assert report_path.read_text() == "OK"  

### На собеседовании:

tmp_path лучше, чем писать в фиксированный путь, потому что каждый тест получает временную директорию, и тесты не конфликтуют между собой.  

## 38. Тестирование времени

### Плохой вариант:

from datetime import datetime  

def is_new_year() -> bool:  
    return datetime.now().month == 1  

Такой код трудно тестировать.  

Лучше:  

from datetime import datetime  

def is_new_year(now: datetime) -> bool:  
    return now.month == 1  

Тест:  

from datetime import datetime  

def test_is_new_year() -> None:  
    assert is_new_year(datetime(2026, 1, 1))  

### На собеседовании:

Лучше внедрять время как зависимость, а не вызывать datetime.now() глубоко внутри бизнес-логики. Тогда тесты проще и стабильнее.  

## 39. Async tests

Для async-тестов часто используют pytest-asyncio.  

import pytest  

@pytest.mark.asyncio  
async def test_async_get_user() -> None:  
    user = await get_user(user_id=1)  

    assert user.id == 1  

### На собеседовании:

Обычный pytest не await’ит async-функции сам по себе. Для async-кода используют плагины, например pytest-asyncio.  

