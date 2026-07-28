# Pytest — полный конспект — часть 7

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 40-46.

## 40. Allure + pytest

Обычно установка:  

pip install allure-pytest  

Запуск:  

pytest --alluredir=allure-results  

Генерация отчёта:  

allure serve allure-results  

Allure-документация для pytest описывает интеграцию для генерации отчётов, улучшения читаемости и навигации, steps, attachments, histories, retries, visual analytics и quality gate.  

### Пример:

import allure  

@allure.feature("Users")  
@allure.story("Create user")  
def test_create_user(api_client) -> None:  
    with allure.step("Create user via API"):  
        response = api_client.create_user(  
            {  
                "username": "alex",  
                "password": "Qwerty123",  
            }  
        )  

    with allure.step("Check response"):  
        assert response.status_code == 201  

Attachment:  

import allure  

def test_response_body(api_client) -> None:  
    response = api_client.get_user(user_id=1)  

    allure.attach(  
        response.text,  
        name="Response body",  
        attachment_type=allure.attachment_type.JSON,  
    )  

    assert response.status_code == 200  

### На собеседовании:

В Allure я бы добавлял steps на бизнес-действия, attachments на request/response/logs/screenshots, labels для feature/story/severity и links на задачи или test cases.  

## 41. Как запускать pytest в CI

### Пример GitLab CI:

stages:  
  - test  

api_tests:  
  stage: test  
  image: python:3.11  
  script:  
    - pip install -r requirements.txt  
    - pytest tests/api -m "smoke" --alluredir=allure-results  
  artifacts:  
    when: always  
    paths:  
      - allure-results  

### Пример Jenkins pipeline:

pipeline {  
    agent any  

    stages {  
        stage('Install dependencies') {  
            steps {  
                sh 'python -m venv .venv'  
                sh '. .venv/bin/activate && pip install -r requirements.txt'  
            }  
        }  

        stage('Run tests') {  
            steps {  
                sh '. .venv/bin/activate && pytest tests -m smoke --alluredir=allure-results'  
            }  
        }  
    }  

    post {  
        always {  
            archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true  
        }  
    }  
}  

### На собеседовании:

В CI обычно разделяют smoke, regression, nightly, pre-merge тесты. Важно сохранять артефакты: Allure results, логи, скриншоты, видео, request/response dump.  

## 42. Flaky tests

Flaky-тест — тест, который иногда проходит, иногда падает без изменения кода.  

Причины:  

1. Зависимость от порядка запуска.  
2. Неочищенные тестовые данные.  
3. Race condition.  
4. Асинхронщина без ожиданий.  
5. Нестабильные внешние сервисы.  
6. Общие пользователи/файлы/порты.  
7. Слишком короткие timeout.  
8. Тест зависит от текущего времени.  
9. UI не дождался состояния.  
10. Параллельный запуск ломает общий state.  

### Что делать:

1. Воспроизвести локально.  
2. Запустить много раз.  
3. Посмотреть логи и артефакты.  
4. Проверить изоляцию данных.  
5. Убрать sleep, заменить на явные ожидания.  
6. Сделать уникальные данные.  
7. Замокать нестабильные внешние зависимости.  
8. Добавить диагностику.  
9. Разделить тест и подготовку данных.  
10. Не прятать проблему бесконечными rerun.  

### На собеседовании:

Rerun может быть временной мерой, но не решением. Сначала нужно понять причину нестабильности.  

## 43. pytest-rerunfailures

### Пример:

pytest --reruns 2 --reruns-delay 1  

### На собеседовании:

Я бы использовал rerun осторожно: например, для нестабильных внешних интеграций, но обязательно с анализом причины flaky.  

## 44. pytest-cov

Запуск:  

pytest --cov=app tests/  

HTML-отчёт:  

pytest --cov=app --cov-report=html tests/  

### На собеседовании:

Coverage показывает, какой код был выполнен тестами, но высокий coverage не гарантирует хорошее качество тестов. Важно проверять смысл assert’ов.  

## 45. Page Object + pytest

### Пример UI-подхода:

class LoginPage:  
    def __init__(self, page):  
        self.page = page  

    def open(self) -> None:  
        self.page.goto("/login")  

    def login(self, username: str, password: str) -> None:  
        self.page.fill("[data-testid='username']", username)  
        self.page.fill("[data-testid='password']", password)  
        self.page.click("[data-testid='login-button']")  

    def error_message(self) -> str:  
        return self.page.text_content("[data-testid='error']")  

Тест:  

def test_login_invalid_password(page) -> None:  
    login_page = LoginPage(page)  

    login_page.open()  
    login_page.login("alex", "wrong-password")  

    assert login_page.error_message() == "Invalid credentials"  

### На собеседовании:

Page Object нужен, чтобы отделить детали UI-локаторов от тестовой логики. Тест должен описывать сценарий, а не набор низкоуровневых кликов.  

## 46. Какие фикстуры делать session scope, а какие function scope

session:  

1. base_url  
2. config  
3. auth token, если он безопасно переиспользуется  
4. подключение к read-only сервису  
5. browser engine  
6. docker/test environment setup  

function:  

1. тестовый пользователь  
2. заказ  
3. данные в БД  
4. состояние корзины  
5. временный файл  
6. browser context/page  
7. транзакция БД  

### На собеседовании:

Всё, что может быть изменено тестом, лучше делать function scope или тщательно очищать. Session scope хорош для дорогих и неизменяемых ресурсов.  

