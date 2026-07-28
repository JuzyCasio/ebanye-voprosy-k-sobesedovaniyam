# Pytest — полный конспект — часть 11

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 77-80.

## 77. Как тестировать брокеры

Для Kafka/Rabbit/NATS можно использовать разные уровни:  

1. Unit: мок producer/consumer.  
2. Integration: поднять брокер в Docker.  
3. Contract: проверить формат сообщения.  
4. E2E: отправить событие и дождаться результата в другой системе.  

### Пример unit-теста producer:

from unittest.mock import Mock  

def test_publish_user_created_event() -> None:  
    producer = Mock()  
    service = UserEventService(producer=producer)  

    service.publish_user_created(user_id=123)  

    producer.publish.assert_called_once_with(  
        topic="user.created",  
        message={"user_id": 123},  
    )  

### На собеседовании:

Для брокеров важно проверять topic/queue, payload, headers, key, schema, idempotency, retry, dead letter queue и обработку дублей.  

## 78. Как тестировать конкурентное выполнение

### Пример:

from concurrent.futures import ThreadPoolExecutor  

def test_concurrent_create_user(api_client) -> None:  
    payloads = [  
        {"username": f"user_{i}", "password": "Qwerty123"}  
        for i in range(10)  
    ]  

    with ThreadPoolExecutor(max_workers=5) as executor:  
        responses = list(executor.map(api_client.create_user, payloads))  

    assert all(response.status_code == 201 for response in responses)  

### На собеседовании:

Конкурентные тесты нужны для проверки race condition, уникальности, блокировок, идемпотентности, дублей и конфликтов.  

## 79. Как тестировать права доступа

import pytest  

@pytest.mark.parametrize(  
    "role, expected_status",  
    [  
        ("admin", 200),  
        ("manager", 403),  
        ("user", 403),  
    ],  
)  
def test_delete_user_permissions(api_client_factory, role: str, expected_status: int) -> None:  
    client = api_client_factory(role=role)  

    response = client.delete_user(user_id=123)  

    assert response.status_code == expected_status  

### На собеседовании:

Права доступа хорошо ложатся на параметризацию: роль, действие, ожидаемый статус.  

## 80. Как тестировать валидацию

import pytest  

@pytest.mark.parametrize(  
    "username",  
    [  
        "",  
        " ",  
        "a" * 256,  
        "admin<script>",  
        "тест",  
    ],  
)  
def test_invalid_username(api_client, username: str) -> None:  
    response = api_client.create_user(  
        {  
            "username": username,  
            "password": "Qwerty123",  
        }  
    )  

    assert response.status_code == 400  

### На собеседовании:

Для валидации удобно использовать классы эквивалентности и граничные значения, а в pytest это хорошо выражается через parametrize.  

