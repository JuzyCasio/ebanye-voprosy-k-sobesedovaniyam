# Python — полный конспект к собеседованию — часть 10

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 55.

## 55. Самое важное для Senior QA Automation

Тебя могут оценивать не только по знанию синтаксиса, а по тому, как ты рассуждаешь.  

### Хорошо говорить так:

Я стараюсь не писать всю логику прямо в тесте. Обычно разделяю код на API-клиенты, фикстуры, модели данных, builders/factories, helpers и assertions. В тесте должна быть видна бизнес-суть: подготовка данных, действие, проверка результата. Низкоуровневые детали лучше держать отдельно.  

### Пример хорошего тестового подхода:

def test_user_can_be_created(user_client, user_factory) -> None:  
    payload = user_factory.build()  

    created_user = user_client.create_user(payload)  

    assert created_user["id"] is not None  
    assert created_user["username"] == payload["username"]  

### Плохой подход:

def test_user_can_be_created() -> None:  
    response = requests.post(  
        "http://host/api/users",  
        json={"username": "alex", "password": "Password1"},  
    )  

    assert response.status_code == 201  

### Почему первый лучше:

тест читается как сценарий;  
меньше дублирования;  
проще менять API;  
проще переиспользовать подготовку данных;  
проще поддерживать большой проект.  

Эту шпаргалку можно дальше развернуть в формат “вопрос → короткий ответ как на собеседовании → пример кода”. Это будет удобнее именно для тренировки перед интервью.  
