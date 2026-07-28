# Pytest — полный конспект — часть 3

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 14-20.

## 14. Параметризация тестов

Параметризация позволяет запустить один тест с разными наборами данных.  

Официальная документация pytest описывает несколько уровней параметризации: через pytest.fixture(params=...), через @pytest.mark.parametrize, а также через pytest_generate_tests для кастомных схем.  

### Пример:

import pytest  

@pytest.mark.parametrize(  
    "password, expected",  
    [  
        ("Qwerty123", True),  
        ("short", False),  
        ("withoutdigits", False),  
        ("12345678", False),  
    ],  
)  
def test_password_validation(password: str, expected: bool) -> None:  
    assert is_valid_password(password) is expected  

### На собеседовании:

parametrize уменьшает дублирование и позволяет явно описать наборы тестовых данных. Один тест запускается несколько раз с разными аргументами.  

## 15. ids в parametrize

Чтобы в отчёте было понятно, какой набор данных упал:  

import pytest  

@pytest.mark.parametrize(  
    "username, expected_status",  
    [  
        ("alex", 201),  
        ("", 400),  
        ("a" * 256, 400),  
    ],  
    ids=[  
        "valid_username",  
        "empty_username",  
        "too_long_username",  
    ],  
)  
def test_create_user(username: str, expected_status: int) -> None:  
    response = create_user(username=username)  

    assert response.status_code == expected_status  

### На собеседовании:

ids нужны для читаемого вывода в консоли и отчётах, особенно когда параметров много.  

## 16. pytest.param

Можно помечать конкретный набор данных:  

import pytest  

@pytest.mark.parametrize(  
    "value, expected",  
    [  
        (1, 2),  
        (2, 4),  
        pytest.param(3, 6, marks=pytest.mark.smoke),  
        pytest.param(4, 8, marks=pytest.mark.xfail(reason="known bug")),  
    ],  
)  
def test_double(value: int, expected: int) -> None:  
    assert value * 2 == expected  

### На собеседовании:

pytest.param позволяет повесить mark, xfail, skip или id на конкретный набор параметров.  

## 17. Параметризация фикстур

Фикстуру тоже можно параметризовать:  

import pytest  

@pytest.fixture(params=["chrome", "firefox", "webkit"])  
def browser_name(request) -> str:  
    return request.param  

def test_open_page(browser_name: str) -> None:  
    assert browser_name in ["chrome", "firefox", "webkit"]  

### На собеседовании:

Параметризованная фикстура запускает все тесты, которые её используют, для каждого значения из params.  

## 18. indirect parametrization

indirect=True означает: параметр передаётся не напрямую в тест, а в фикстуру через request.param.  

import pytest  

@pytest.fixture  
def user(request) -> dict[str, str]:  
    role = request.param  

    return {  
        "name": "Alex",  
        "role": role,  
    }  

@pytest.mark.parametrize("user", ["admin", "manager"], indirect=True)  
def test_user_role(user: dict[str, str]) -> None:  
    assert user["role"] in ["admin", "manager"]  

### На собеседовании:

indirect используют, когда тестовые данные должны пройти через фикстуру, например для создания пользователя, подключения к окружению или подготовки сложного объекта.  

## 19. Маркеры

Маркер — это метка на тесте.  

import pytest  

@pytest.mark.smoke  
def test_login() -> None:  
    assert True  

@pytest.mark.regression  
def test_create_order() -> None:  
    assert True  

Запуск:  

pytest -m smoke  

Исключить slow:  

pytest -m "not slow"  

Комбинация:  

pytest -m "smoke or critical"  

Лучше регистрировать маркеры в конфиге:  

[tool.pytest.ini_options]  
markers = [  
    "smoke: быстрые smoke-тесты",  
    "regression: регрессионные тесты",  
    "critical: критичные тесты",  
    "slow: медленные тесты",  
]  

### На собеседовании:

Маркеры позволяют группировать тесты: smoke, regression, slow, api, ui, db. В CI удобно запускать разные наборы тестов по маркерам.  

## 20. skip, skipif, xfail

skip — тест не запускаем.  

import pytest  

@pytest.mark.skip(reason="temporarily disabled")  
def test_old_feature() -> None:  
    assert False  

skipif — пропускаем по условию:  

import sys  
import pytest  

@pytest.mark.skipif(sys.platform == "win32", reason="Linux only test")  
def test_linux_command() -> None:  
    assert True  

xfail — ожидаем, что тест упадёт:  

import pytest  

@pytest.mark.xfail(reason="known bug")  
def test_known_bug() -> None:  
    assert 1 == 2  

Разница:  

skip  — тест не запускается  
xfail — тест запускается, но падение ожидаемое  
xpass — тест неожиданно прошёл, хотя был помечен xfail  

Документация pytest формулирует это так: skip используется, когда тест должен проходить только при определённых условиях, а xfail — когда тест ожидаемо падает, например из-за известного бага или ещё не реализованной функциональности.  

### На собеседовании:

xfail лучше использовать для известного бага с ссылкой на задачу. Если баг починили и тест стал проходить, pytest покажет XPASS, и это сигнал убрать xfail.  

