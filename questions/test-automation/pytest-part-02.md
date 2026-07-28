# Pytest — полный конспект — часть 2

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 7-13.

## 7. Фикстуры

Фикстура — это способ подготовить данные, объект, подключение, клиент, пользователя, браузер, БД или окружение для теста.  

Официальная документация описывает фикстуру как механизм, который даёт тестам заранее определённый и воспроизводимый контекст, например настроенную БД или подготовленные данные.  

### Пример:

import pytest  

@pytest.fixture  
def user() -> dict[str, str]:  
    return {"name": "Alex", "role": "admin"}  

def test_user_role(user: dict[str, str]) -> None:  
    assert user["role"] == "admin"  

### Главная идея:

pytest передаёт фикстуру в тест по имени аргумента.  

То есть здесь:  

def test_user_role(user):  
    ...  

pytest видит аргумент user, ищет фикстуру с таким именем и выполняет её.  

## 8. Scope фикстур

У фикстур есть область жизни:  

@pytest.fixture(scope="function")  
def fixture_func():  
    ...  

### Основные scope:

function — на каждый тест  
class    — на класс  
module   — на файл  
package  — на пакет  
session  — на весь запуск pytest  

### Пример:

import pytest  

@pytest.fixture(scope="session")  
def auth_token() -> str:  
    return "token-123"  

def test_one(auth_token: str) -> None:  
    assert auth_token  

def test_two(auth_token: str) -> None:  
    assert auth_token.startswith("token")  

### На собеседовании:

Scope определяет, как часто создаётся фикстура. function — перед каждым тестом, session — один раз за весь тестовый запуск. Чем шире scope, тем аккуратнее нужно быть с изменяемым состоянием.  

## 9. Yield fixture: setup и teardown

Фикстура может не только вернуть объект, но и подчистить ресурсы после теста.  

import pytest  

@pytest.fixture  
def db_connection():  
    connection = create_connection()  
    yield connection  
    connection.close()  

Всё до yield — setup.  
Всё после yield — teardown.  

### Пример попроще:

import pytest  

@pytest.fixture  
def file_resource(tmp_path):  
    file_path = tmp_path / "data.txt"  
    file_path.write_text("hello")  

    yield file_path  

    # teardown  
    if file_path.exists():  
        file_path.unlink()  

Тест:  

def test_file_content(file_resource) -> None:  
    assert file_resource.read_text() == "hello"  

### На собеседовании:

yield-фикстура удобна для ресурсов, которые нужно закрывать: соединение с БД, браузер, временный пользователь, файл, мок-сервер. Код после yield выполнится после завершения теста.  

## 10. addfinalizer

Альтернатива yield — request.addfinalizer.  

import pytest  

@pytest.fixture  
def resource(request):  
    item = create_resource()  

    def cleanup() -> None:  
        delete_resource(item)  

    request.addfinalizer(cleanup)  

    return item  

Но чаще в обычных проектах используют yield, потому что он читается проще.  

### На собеседовании:

addfinalizer полезен, когда нужно динамически регистрировать cleanup-функции, но для простого setup/teardown чаще читаемее yield.  

## 11. Autouse fixtures

Autouse fixture применяется автоматически, даже если тест явно её не запросил.  

import pytest  

@pytest.fixture(autouse=True)  
def clean_state() -> None:  
    print("before test")  

Официальная документация описывает autouse-фикстуры как способ автоматически запрашивать фикстуру для тестов, чтобы не дублировать её в аргументах.  

### Пример реального использования:

import pytest  

@pytest.fixture(autouse=True)  
def reset_env(monkeypatch: pytest.MonkeyPatch) -> None:  
    monkeypatch.delenv("DEBUG", raising=False)  

### На собеседовании:

Autouse хорош для глобальной подготовки: очистка состояния, настройка env, логирование, reset моков. Но им нельзя злоупотреблять, потому что скрытые зависимости ухудшают читаемость тестов.  

## 12. Зависимости между фикстурами

Фикстура может зависеть от другой фикстуры:  

import pytest  

@pytest.fixture  
def token() -> str:  
    return "secret-token"  

@pytest.fixture  
def api_client(token: str):  
    return ApiClient(token=token)  

def test_get_user(api_client) -> None:  
    response = api_client.get_user(user_id=1)  
    assert response.status_code == 200  

### На собеседовании:

Фикстуры можно строить как граф зависимостей. pytest сам вычисляет порядок выполнения по зависимостям.  

## 13. Фабрики через фикстуры

Иногда фикстура должна не сразу создавать объект, а возвращать функцию-фабрику.  

import pytest  

@pytest.fixture  
def user_factory():  
    def create_user(name: str = "Alex", role: str = "user") -> dict[str, str]:  
        return {  
            "name": name,  
            "role": role,  
        }  

    return create_user  

def test_admin_user(user_factory) -> None:  
    user = user_factory(name="Ivan", role="admin")  

    assert user["role"] == "admin"  

### На собеседовании:

Если в тесте нужно создавать много объектов с разными параметрами, удобно делать fixture factory: фикстура возвращает функцию создания данных.  

