# Pytest — полный конспект — часть 5

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 26-30.

## 26. Плагины pytest

pytest расширяется плагинами.  

Популярные:  

pytest-xdist      — параллельный запуск  
pytest-cov        — coverage  
pytest-rerunfailures — перезапуск flaky-тестов  
allure-pytest     — Allure-отчёты  
pytest-mock       — удобная работа с mock  
pytest-asyncio    — async-тесты  
pytest-timeout    — timeout на тесты  

### На собеседовании:

Плагины в pytest — это расширения, которые добавляют фикстуры, хуки, CLI-опции или отчётность. Например, xdist добавляет параллельный запуск, allure-pytest — генерацию Allure results, pytest-cov — coverage.  

## 27. pytest-xdist

Установка:  

pip install pytest-xdist  

Запуск:  

pytest -n auto  

Или конкретное число воркеров:  

pytest -n 4  

Документация pytest-xdist говорит, что плагин добавляет режимы выполнения тестов, самый частый из которых — распределение тестов по нескольким CPU для ускорения запуска; при pytest -n auto создаются worker-процессы по числу доступных CPU.  

Важный вопрос на собесе:  

### Как pytest-xdist распределяет тесты?

Обычный ответ:  

xdist запускает несколько worker-процессов. Основной процесс собирает тесты и распределяет их между worker’ами. Поэтому тесты должны быть независимыми, не должны конфликтовать за одни и те же файлы, пользователей, БД-записи или порты.  

Проблемы при параллельном запуске:  

1. Общая БД без изоляции.  
2. Один и тот же тестовый пользователь.  
3. Общий файл для записи.  
4. Общий порт.  
5. Тесты зависят от порядка запуска.  
6. Фикстура session scope создаёт общий mutable state.  

### Как решать:

import uuid  

def unique_email() -> str:  
    return f"user_{uuid.uuid4().hex}@example.com"  

Или учитывать worker id:  

import pytest  

@pytest.fixture  
def user_email(worker_id: str) -> str:  
    return f"user_{worker_id}@example.com"  

## 28. Как изолировать тесты

### На собеседовании это прям частый вопрос.

Ответ:  

Изоляция означает, что тест не зависит от других тестов и не оставляет после себя состояние, которое может повлиять на следующий тест.  

Способы:  

1. Уникальные тестовые данные.  
2. Очистка данных после теста.  
3. Транзакции с rollback.  
4. Отдельная схема/БД на worker.  
5. Моки внешних сервисов.  
6. tmp_path вместо общих файлов.  
7. Не полагаться на порядок тестов.  
8. Не использовать общий mutable state.  
9. Для API — создавать данные через API/fixture и удалять после теста.  
10. Для UI — использовать независимых пользователей или сбрасывать состояние.  

### Пример cleanup:

import pytest  

@pytest.fixture  
def created_user(api_client):  
    user = api_client.create_user(name="Alex")  

    yield user  

    api_client.delete_user(user["id"])  

## 29. Как тестировать API через pytest

### Пример клиента:

from dataclasses import dataclass  

import requests  

@dataclass  
class ApiClient:  
    base_url: str  

    def get_user(self, user_id: int) -> requests.Response:  
        return requests.get(f"{self.base_url}/users/{user_id}", timeout=5)  

    def create_user(self, payload: dict) -> requests.Response:  
        return requests.post(f"{self.base_url}/users", json=payload, timeout=5)  

Фикстура:  

import pytest  

@pytest.fixture(scope="session")  
def api_client(base_url: str) -> ApiClient:  
    return ApiClient(base_url=base_url)  

Тест:  

def test_create_user(api_client: ApiClient) -> None:  
    payload = {  
        "username": "alex",  
        "password": "Qwerty123",  
    }  

    response = api_client.create_user(payload)  

    assert response.status_code == 201  

    body = response.json()  

    assert "id" in body  
    assert body["username"] == payload["username"]  

### Что проверять в API:

1. status code  
2. response body  
3. JSON schema  
4. обязательные поля  
5. типы данных  
6. ошибки валидации  
7. headers  
8. авторизацию  
9. права доступа  
10. идемпотентность  
11. таймауты  
12. негативные сценарии  
13. контракты между сервисами  

## 30. Пример параметризованного API-теста

import pytest  

@pytest.mark.parametrize(  
    "payload, expected_status",  
    [  
        (  
            {"username": "alex", "password": "Qwerty123"},  
            201,  
        ),  
        (  
            {"username": "", "password": "Qwerty123"},  
            400,  
        ),  
        (  
            {"username": "alex", "password": "short"},  
            400,  
        ),  
        (  
            {"username": "a" * 256, "password": "Qwerty123"},  
            400,  
        ),  
    ],  
    ids=[  
        "valid_user",  
        "empty_username",  
        "short_password",  
        "too_long_username",  
    ],  
)  
def test_create_user_validation(api_client, payload: dict, expected_status: int) -> None:  
    response = api_client.create_user(payload)  

    assert response.status_code == expected_status  

### На собеседовании:

Я бы вынес API-клиент в отдельный слой, тестовые данные — в фикстуры или фабрики, а проверки — в читаемые assert’ы или helper-функции, если они повторяются.  

