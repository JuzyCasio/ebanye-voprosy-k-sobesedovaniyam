# Pytest — полный конспект — часть 10

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 63-76.

## 63. Что такое request fixture

request даёт доступ к текущему тестовому контексту.  

### Пример с именем теста:

import pytest  

@pytest.fixture  
def current_test_name(request) -> str:  
    return request.node.name  

### Пример с параметром:

import pytest  

@pytest.fixture  
def role(request) -> str:  
    return request.param  

@pytest.mark.parametrize("role", ["admin", "user"], indirect=True)  
def test_role(role: str) -> None:  
    assert role in ["admin", "user"]  

### На собеседовании:

request нужен, когда фикстуре нужен доступ к контексту: параметрам, имени теста, markers, config, node.  

## 64. Доступ к markers из фикстуры

import pytest  

@pytest.fixture  
def user_role(request) -> str:  
    marker = request.node.get_closest_marker("role")  

    if marker is None:  
        return "user"  

    return marker.args[0]  

@pytest.mark.role("admin")  
def test_admin_permissions(user_role: str) -> None:  
    assert user_role == "admin"  

### На собеседовании:

Иногда фикстура может менять поведение в зависимости от marker на тесте.  

## 65. Пример роли через marker

import pytest  

@pytest.fixture  
def user(request, api_client):  
    marker = request.node.get_closest_marker("user_role")  
    role = marker.args[0] if marker else "user"  

    created_user = api_client.create_user({"role": role}).json()  

    yield created_user  

    api_client.delete_user(created_user["id"])  

@pytest.mark.user_role("admin")  
def test_admin_can_create_user(user) -> None:  
    assert user["role"] == "admin"  

## 66. pytest.ini vs pyproject.toml

pytest.ini:  

[pytest]  
addopts = -ra -q  
testpaths =  
    tests  
markers =  
    smoke: smoke tests  
    regression: regression tests  

pyproject.toml:  

[tool.pytest.ini_options]  
addopts = "-ra -q"  
testpaths = ["tests"]  
markers = [  
    "smoke: smoke tests",  
    "regression: regression tests",  
]  

### На собеседовании:

В современных проектах часто используют pyproject.toml, потому что там можно держать настройки разных инструментов: pytest, black, ruff, mypy. Но pytest.ini тоже нормальный вариант.  

## 67. Что такое -ra

pytest -ra  

Показывает дополнительную summary-информацию:  

s — skipped  
x — xfailed  
X — xpassed  
f — failed  
E — error  

Полезно в CI.  

## 68. Разница FAILED и ERROR

FAILED — тест запустился, но assert упал.  
ERROR  — ошибка случилась на setup/teardown/fixture/collection.  

### Пример FAILED:

def test_failed() -> None:  
    assert 1 == 2  

### Пример ERROR:

import pytest  

@pytest.fixture  
def broken_fixture():  
    raise RuntimeError("Cannot prepare data")  

def test_error(broken_fixture) -> None:  
    assert True  

### На собеседовании:

Failed — это проблема проверки. Error — тест даже нормально не дошёл до проверки, например упала фикстура.  

## 69. Collection errors

### Пример:

import not_existing_module  

def test_example() -> None:  
    assert True  

pytest может упасть ещё на этапе collection.  

### На собеседовании:

Collection error возникает до запуска теста: например, ошибка импорта, синтаксиса, неправильная параметризация.  

## 70. Как дебажить pytest

### Команды:

pytest -s  
pytest -vv  
pytest --tb=long  
pytest --pdb  
pytest --maxfail=1  
pytest tests/test_file.py::test_name  
pytest --collect-only  

В коде:  

def test_debug() -> None:  
    value = calculate()  

    breakpoint()  

    assert value == 10  

### На собеседовании:

Я обычно сужаю запуск до одного теста, включаю подробный traceback, смотрю фикстуры, данные, request/response, логи и при необходимости запускаю с --pdb или breakpoint().  

## 71. Как понять, какие фикстуры доступны

pytest --fixtures  

Или для конкретного пути:  

pytest --fixtures tests/api  

### На собеседовании:

pytest --fixtures показывает доступные фикстуры, включая встроенные и добавленные плагинами.  

## 72. pytest.importorskip

import pytest  

numpy = pytest.importorskip("numpy")  

def test_numpy_available() -> None:  
    assert numpy.array([1, 2, 3]).sum() == 6  

### На собеседовании:

importorskip полезен, если тест зависит от необязательной библиотеки.  

## 73. Как тестировать CLI

### Пример функции:

def main(args: list[str]) -> int:  
    if "--help" in args:  
        print("Usage: app")  
        return 0  

    return 1  

Тест:  

def test_main_help(capsys) -> None:  
    exit_code = main(["--help"])  

    captured = capsys.readouterr()  

    assert exit_code == 0  
    assert "Usage" in captured.out  

## 74. Как тестировать файлы

def parse_file(path) -> list[str]:  
    return path.read_text().splitlines()  

def test_parse_file(tmp_path) -> None:  
    file_path = tmp_path / "users.txt"  
    file_path.write_text("alex\nivan\n")  

    result = parse_file(file_path)  

    assert result == ["alex", "ivan"]  

## 75. Как тестировать переменные окружения

import os  

def get_mode() -> str:  
    return os.getenv("APP_MODE", "dev")  

def test_get_mode(monkeypatch) -> None:  
    monkeypatch.setenv("APP_MODE", "test")  

    assert get_mode() == "test"  

def test_get_mode_default(monkeypatch) -> None:  
    monkeypatch.delenv("APP_MODE", raising=False)  

    assert get_mode() == "dev"  

## 76. Как тестировать retry

from unittest.mock import Mock  

def test_retry_success_on_second_attempt() -> None:  
    client = Mock()  
    client.get.side_effect = [TimeoutError, {"status": "ok"}]  

    result = get_with_retry(client)  

    assert result == {"status": "ok"}  
    assert client.get.call_count == 2  

### На собеседовании:

Для retry удобно мокать зависимость и через side_effect задавать последовательность: сначала ошибка, потом успех.  

