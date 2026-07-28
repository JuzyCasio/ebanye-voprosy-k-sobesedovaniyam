# Pytest — полный конспект — часть 4

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 21-25.

## 21. Встроенные фикстуры pytest

tmp_path  

Создаёт временную директорию как pathlib.Path.  

def test_write_file(tmp_path) -> None:  
    file_path = tmp_path / "test.txt"  
    file_path.write_text("hello")  

    assert file_path.read_text() == "hello"  
monkeypatch  

Позволяет временно менять:  

переменные окружения;  
атрибуты объектов;  
функции;  
словари;  
sys.path;  
текущую директорию.  
def test_env(monkeypatch) -> None:  
    monkeypatch.setenv("APP_ENV", "test")  

    assert get_env() == "test"  

Мок функции:  

def test_get_current_user(monkeypatch) -> None:  
    def fake_get_user_id() -> int:  
        return 123  

    monkeypatch.setattr("app.auth.get_user_id", fake_get_user_id)  

    assert get_current_user_id() == 123  

Документация pytest описывает monkeypatch как встроенную фикстуру для временного изменения объектов, словарей и os.environ; изменения автоматически откатываются после теста.  

capsys  

Перехват stdout и stderr.  

def test_print(capsys) -> None:  
    print("hello")  

    captured = capsys.readouterr()  

    assert captured.out == "hello\n"  
caplog  

### Проверка логов.

import logging  

def test_logging(caplog) -> None:  
    with caplog.at_level(logging.INFO):  
        logging.info("User created")  

    assert "User created" in caplog.text  
pytestconfig  

Доступ к конфигурации pytest.  

def test_config(pytestconfig) -> None:  
    verbose = pytestconfig.getoption("verbose")  

    assert isinstance(verbose, int)  
request  

Доступ к контексту текущего теста или фикстуры.  

import pytest  

@pytest.fixture  
def test_name(request) -> str:  
    return request.node.name  

def test_example(test_name: str) -> None:  
    assert test_name == "test_example"  

## 22. Кастомные CLI-опции

В conftest.py:  

def pytest_addoption(parser) -> None:  
    parser.addoption(  
        "--env",  
        action="store",  
        default="dev",  
        help="Environment: dev, stage, prod",  
    )  

Фикстура:  

import pytest  

@pytest.fixture  
def env(pytestconfig) -> str:  
    return pytestconfig.getoption("--env")  

Тест:  

def test_env(env: str) -> None:  
    assert env in ["dev", "stage", "prod"]  

Запуск:  

pytest --env=stage  

### На собеседовании:

Кастомные CLI-опции удобно использовать для выбора окружения, base_url, браузера, запуска against mock/real service, включения debug-режима.  

## 23. Пример base_url через CLI

# conftest.py  

import pytest  

def pytest_addoption(parser) -> None:  
    parser.addoption(  
        "--base-url",  
        action="store",  
        default="https://api.dev.example.com",  
    )  

@pytest.fixture(scope="session")  
def base_url(pytestconfig) -> str:  
    return pytestconfig.getoption("--base-url")  

Тест:  

def test_healthcheck(base_url: str) -> None:  
    response = requests.get(f"{base_url}/health")  

    assert response.status_code == 200  

Запуск:  

pytest --base-url=https://api.stage.example.com  

## 24. Хуки pytest

Хуки позволяют вмешиваться в жизненный цикл pytest.  

### Частые хуки:

pytest_addoption              — добавить CLI-опции  
pytest_configure              — настройка после парсинга конфига  
pytest_collection_modifyitems — изменить список собранных тестов  
pytest_generate_tests         — динамическая параметризация  
pytest_runtest_setup          — перед запуском теста  
pytest_runtest_call           — сам вызов теста  
pytest_runtest_teardown       — после теста  
pytest_sessionstart           — старт сессии  
pytest_sessionfinish          — конец сессии  

### Пример: автоматически добавлять marker api всем тестам из папки api.

import pytest  

def pytest_collection_modifyitems(items) -> None:  
    for item in items:  
        if "api" in str(item.fspath):  
            item.add_marker(pytest.mark.api)  

### Пример: пропускать slow-тесты без флага.

import pytest  

def pytest_addoption(parser) -> None:  
    parser.addoption(  
        "--run-slow",  
        action="store_true",  
        default=False,  
        help="Run slow tests",  
    )  

def pytest_collection_modifyitems(config, items) -> None:  
    if config.getoption("--run-slow"):  
        return  

    skip_slow = pytest.mark.skip(reason="need --run-slow option to run")  

    for item in items:  
        if "slow" in item.keywords:  
            item.add_marker(skip_slow)  

Официальная документация описывает хуки как механизм плагинов pytest; один hook может иметь несколько реализаций, а hook wrapper позволяет выполнить код «вокруг» других hook-реализаций.  

### На собеседовании:

Хуки нужны, когда стандартных фикстур уже мало: например, нужно менять коллекцию тестов, добавлять опции запуска, динамически параметризовать тесты или интегрироваться с отчётами/CI.  

## 25. pytest_generate_tests

Используется для динамической параметризации.  

def pytest_addoption(parser) -> None:  
    parser.addoption(  
        "--users",  
        action="store",  
        default="admin,manager",  
    )  

def pytest_generate_tests(metafunc) -> None:  
    if "role" in metafunc.fixturenames:  
        roles = metafunc.config.getoption("--users").split(",")  
        metafunc.parametrize("role", roles)  

Тест:  

def test_user_role(role: str) -> None:  
    assert role in ["admin", "manager", "operator"]  

Запуск:  

pytest --users=admin,operator  

### На собеседовании:

pytest_generate_tests применяют, когда набор параметров неизвестен заранее: например, он приходит из CLI, файла, базы, API или конфигурации окружения.  

