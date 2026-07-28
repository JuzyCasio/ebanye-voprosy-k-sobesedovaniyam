# Pytest — полный конспект — часть 1

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 1-6.

## 1. Что такое pytest

pytest — это тестовый фреймворк для Python, который используют для unit, integration, API, UI и end-to-end тестов. Его сильные стороны: простые assert, фикстуры, параметризация, плагины, удобный запуск из CLI, интеграция с Allure, CI/CD и параллельный запуск через плагины. В официальной документации pytest отдельно выделены фикстуры, параметризация, marks, плагины, конфигурация и работа с кэшем как основные механизмы фреймворка.  

### На собеседовании можно сказать так:

pytest удобен тем, что тесты пишутся как обычные Python-функции, зависимости передаются через фикстуры, тестовые данные удобно раскладываются через parametrize, а поведение фреймворка можно расширять через хуки и плагины.  

Простейший тест:  

def test_sum() -> None:  
    assert 1 + 1 == 2  

## 2. Как pytest находит тесты

По умолчанию pytest ищет:  

test_*.py  
*_test.py  

Внутри файлов:  

def test_something():  
    ...  

class TestUser:  
    def test_create_user(self):  
        ...  

### Важно:

class TestUser:  
    def test_user_created(self):  
        assert True  

Но если у класса есть __init__, pytest обычно не будет нормально собирать такой класс как тестовый:  

class TestUser:  
    def __init__(self):  
        ...  

### На собесе:

pytest собирает тесты по naming convention: файлы test_*.py или *_test.py, функции test_*, классы Test*. Для классов тестов не нужен __init__, состояние лучше готовить через фикстуры.  

## 3. Основные команды запуска

pytest  

Запустить все тесты.  

pytest tests/test_users.py  

Запустить конкретный файл.  

pytest tests/test_users.py::test_create_user  

Запустить конкретный тест.  

pytest tests/test_users.py::TestUserApi::test_create_user  

Запустить конкретный тест внутри класса.  

pytest -v  

Подробный вывод.  

pytest -q  

### Короткий вывод.

pytest -s  

Не перехватывать print.  

pytest -x  

Остановиться после первого падения.  

pytest --maxfail=3  

Остановиться после трёх падений.  

pytest -k "user and not slow"  

Запустить тесты, где имя содержит выражение.  

pytest -m smoke  

Запустить тесты с маркером smoke.  

pytest --tb=short  

Сократить traceback.  

pytest --collect-only  

Только собрать тесты, не запускать.  

pytest --durations=10  

Показать 10 самых медленных тестов.  

pytest --lf  
pytest --last-failed  

Запустить только тесты, которые упали в прошлый раз.  

pytest --ff  
pytest --failed-first  

Сначала запустить упавшие в прошлый раз, потом остальные.  

Опции --lf, --ff и --cache-clear относятся к встроенному cache-механизму pytest.  

## 4. Assert в pytest

В pytest не нужно писать специальные assert-методы как в unittest:  

def test_user_name() -> None:  
    user = {"name": "Alex"}  
    assert user["name"] == "Alex"  

pytest сам покажет разницу:  

def test_lists() -> None:  
    assert [1, 2, 3] == [1, 2, 4]  

Можно добавлять сообщение:  

def test_status_code() -> None:  
    status_code = 500  

    assert status_code == 200, f"Expected 200, got {status_code}"  

### На собеседовании:

В pytest используются обычные Python assert, но pytest переписывает assert-выражения и даёт подробный diff при падении.  

## 5. Структура проекта

### Типичная структура:

project/  
├── app/  
│   └── users.py  
├── tests/  
│   ├── conftest.py  
│   ├── test_users.py  
│   └── test_orders.py  
├── pyproject.toml  
└── requirements.txt  

### Пример pyproject.toml:

[tool.pytest.ini_options]  
pythonpath = ["."]  
testpaths = ["tests"]  
addopts = "-ra -q"  
markers = [  
    "smoke: быстрые smoke-тесты",  
    "regression: регрессионные тесты",  
    "slow: медленные тесты",  
]  

Настройки pytest можно хранить в конфигурационных файлах в корне проекта; в актуальной документации перечислены поддерживаемые форматы, включая pytest.toml, pytest.ini, pyproject.toml, tox.ini и setup.cfg.  

## 6. conftest.py

conftest.py — специальный файл pytest, куда обычно кладут:  

фикстуры;  
хуки;  
кастомные CLI-опции;  
общую конфигурацию тестов;  
подготовку окружения.  

### Пример:

# tests/conftest.py  

import pytest  

@pytest.fixture  
def base_url() -> str:  
    return "https://api.example.com"  

Использование:  

def test_base_url(base_url: str) -> None:  
    assert base_url.startswith("https://")  

### На собеседовании:

conftest.py позволяет объявлять фикстуры и хуки без явного импорта в тестах. pytest сам находит этот файл и делает фикстуры доступными для тестов в текущей директории и ниже.  

