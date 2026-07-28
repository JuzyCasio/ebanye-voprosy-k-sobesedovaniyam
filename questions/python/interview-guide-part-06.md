# Python — полный конспект к собеседованию — часть 6

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 43-44.

## 43. SOLID коротко для Python

S — Single Responsibility  

Класс должен иметь одну ответственность.  

### Плохо:

class UserService:  
    def create_user(self):  
        ...  

    def send_email(self):  
        ...  

    def write_log(self):  
        ...  

Лучше:  

class UserService:  
    def create_user(self):  
        ...  

class EmailService:  
    def send_email(self):  
        ...  

class Logger:  
    def write_log(self):  
        ...  
O — Open/Closed  

Код должен быть открыт для расширения, но закрыт для изменения.  

### Пример через Strategy:

from typing import Protocol  

class AuthStrategy(Protocol):  
    def get_headers(self) -> dict[str, str]:  
        ...  

class TokenAuth:  
    def __init__(self, token: str) -> None:  
        self.token = token  

    def get_headers(self) -> dict[str, str]:  
        return {"Authorization": f"Bearer {self.token}"}  

class ApiClient:  
    def __init__(self, auth: AuthStrategy) -> None:  
        self.auth = auth  
L — Liskov Substitution  

Дочерний класс должен быть заменяемым вместо родительского.  

I — Interface Segregation  

Лучше несколько маленьких интерфейсов, чем один большой.  

D — Dependency Inversion  

Зависеть лучше от абстракций, а не от конкретных реализаций.  

## 44. Частые паттерны

Page Object  

Для UI-автотестов.  

from selenium.webdriver.remote.webdriver import WebDriver  
from selenium.webdriver.common.by import By  

class LoginPage:  
    USERNAME_INPUT = (By.ID, "username")  
    PASSWORD_INPUT = (By.ID, "password")  
    LOGIN_BUTTON = (By.ID, "login")  

    def __init__(self, driver: WebDriver) -> None:  
        self.driver = driver  

    def open(self) -> None:  
        self.driver.get("https://example.com/login")  

    def login(self, username: str, password: str) -> None:  
        self.driver.find_element(*self.USERNAME_INPUT).send_keys(username)  
        self.driver.find_element(*self.PASSWORD_INPUT).send_keys(password)  
        self.driver.find_element(*self.LOGIN_BUTTON).click()  
Factory  

Создание объектов.  

from dataclasses import dataclass  

@dataclass  
class User:  
    name: str  
    email: str  

class UserFactory:  
    @staticmethod  
    def create_user(name: str = "Alex") -> User:  
        return User(  
            name=name,  
            email=f"{name.lower()}@example.com",  
        )  
Builder  

### Когда объект сложный.

class UserBuilder:  
    def __init__(self) -> None:  
        self.data = {  
            "name": "Alex",  
            "age": 30,  
            "role": "user",  
        }  

    def with_name(self, name: str) -> "UserBuilder":  
        self.data["name"] = name  
        return self  

    def with_role(self, role: str) -> "UserBuilder":  
        self.data["role"] = role  
        return self  

    def build(self) -> dict[str, object]:  
        return self.data  

Использование:  

user = (  
    UserBuilder()  
    .with_name("Ivan")  
    .with_role("admin")  
    .build()  
)  
Strategy  

### Когда нужно подставлять разные алгоритмы.

from typing import Protocol  

class PaymentStrategy(Protocol):  
    def pay(self, amount: int) -> str:  
        ...  

class CardPayment:  
    def pay(self, amount: int) -> str:  
        return f"Paid {amount} by card"  

class CashPayment:  
    def pay(self, amount: int) -> str:  
        return f"Paid {amount} by cash"  

class PaymentService:  
    def __init__(self, strategy: PaymentStrategy) -> None:  
        self.strategy = strategy  

    def process(self, amount: int) -> str:  
        return self.strategy.pay(amount)  

