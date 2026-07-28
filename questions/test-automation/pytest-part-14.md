# Pytest — полный конспект — часть 14

[← Оглавление](pytest.md) · [← К разделу](../test-automation.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/pytest.md)

Темы 87-89.

## 87. Частые ошибки новичков в pytest

1. Держать всю логику в одном огромном conftest.py.  
2. Делать слишком много autouse fixtures.  
3. Использовать session fixture для изменяемых данных.  
4. Не чистить данные после теста.  
5. Писать sleep вместо ожиданий.  
6. Не ставить timeout на API-запросы.  
7. Делать тесты зависимыми от порядка.  
8. Использовать один и тот же user/email во всех тестах.  
9. Прятать важные assert’ы в непонятных helper’ах.  
10. Не регистрировать custom markers.  
11. Проверять только status code и не проверять body.  
12. Игнорировать flaky, просто добавляя rerun.  
13. Писать UI-тесты на всё подряд вместо API/unit.  

## 88. Короткая финальная шпаргалка перед собеседованием

pytest — тестовый фреймворк Python.  

Тесты:  
- test_*.py  
- *_test.py  
- test_* функции  
- Test* классы  

Запуск:  
- pytest  
- pytest -v  
- pytest -s  
- pytest -k "login"  
- pytest -m smoke  
- pytest -x  
- pytest --maxfail=1  
- pytest --lf  
- pytest --ff  
- pytest --collect-only  
- pytest --durations=10  

Фикстуры:  
- @pytest.fixture  
- передаются по имени аргумента  
- scope: function/class/module/package/session  
- yield для teardown  
- autouse=True для автоматического применения  
- params для параметризации фикстур  
- request для доступа к контексту  

Параметризация:  
- @pytest.mark.parametrize  
- ids для читаемых названий  
- pytest.param для marks на конкретном кейсе  
- indirect=True для передачи параметра в фикстуру  

Marks:  
- @pytest.mark.smoke  
- @pytest.mark.regression  
- @pytest.mark.skip  
- @pytest.mark.skipif  
- @pytest.mark.xfail  

conftest.py:  
- фикстуры  
- хуки  
- CLI options  
- общая настройка тестов  

Хуки:  
- pytest_addoption  
- pytest_configure  
- pytest_collection_modifyitems  
- pytest_generate_tests  
- pytest_sessionstart  
- pytest_sessionfinish  

Встроенные фикстуры:  
- tmp_path  
- monkeypatch  
- caplog  
- capsys  
- pytestconfig  
- request  

Параллельность:  
- pytest-xdist  
- pytest -n auto  
- тесты должны быть независимыми  

Allure:  
- pytest --alluredir=allure-results  
- steps  
- attachments  
- feature/story/severity  

CI:  
- smoke на MR  
- regression по расписанию  
- Allure/logs/screenshots как artifacts  

## 89. Самый сильный ответ про pytest на собесе

Можно выучить почти дословно:  

Я использую pytest как основу автотестового фреймворка. Обычно разделяю тесты, API-клиенты, фикстуры, фабрики тестовых данных и helper assertions. Через фикстуры готовлю окружение, пользователей, токены, БД-сессии и клиентов. Через parametrize покрываю валидацию, роли и граничные значения. Через markers разделяю smoke, regression и slow тесты. Для CI добавляю параметры запуска вроде --base-url и --env, отчётность через Allure, а для ускорения — xdist. При этом слежу за изоляцией тестов: уникальные данные, cleanup, rollback транзакций, отсутствие зависимости от порядка и аккуратное использование session fixtures.  
