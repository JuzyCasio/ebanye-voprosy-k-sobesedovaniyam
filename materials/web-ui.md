# Web и UI-автоматизация — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/web-ui.md)

Материал для подготовки AQA/SDET Python. Основной акцент сделан на Selenium, потому что его часто спрашивают на собеседованиях. Playwright разобран отдельно и сравнивается с Selenium.

## 1. Что такое UI-тестирование

UI-тестирование проверяет приложение через пользовательский интерфейс: браузер открывает страницу, находит элементы, выполняет действия пользователя и проверяет результат.

Пример сценария:

1. Открыть страницу входа.
2. Ввести логин и пароль.
3. Нажать кнопку.
4. Проверить, что открылась главная страница.

UI-тест проверяет сразу несколько слоёв:

- frontend;
- взаимодействие с backend;
- маршрутизацию;
- авторизацию;
- отображение данных;
- поведение браузера.

Из-за большого числа зависимостей UI-тесты медленнее и нестабильнее API- и unit-тестов.

### Короткий ответ

UI-тестирование — проверка приложения через интерфейс пользователя. Такие тесты полезны для критичных сквозных сценариев, но основную бизнес-логику выгоднее проверять на API- или компонентном уровне.

## 2. Что автоматизировать через UI

Хорошие кандидаты:

- вход и восстановление доступа;
- оформление заказа;
- критичный пользовательский путь;
- права и видимость элементов;
- интеграция нескольких частей системы;
- ограниченный smoke-набор;
- сценарии, которые невозможно полноценно проверить ниже UI.

Не стоит переносить в UI:

- все комбинации входных данных;
- подробную проверку API-контракта;
- сложную бизнес-логику;
- подготовку большого объёма данных;
- проверки, которые быстрее и надёжнее выполняются через API или БД.

Сильный подход: данные готовим через API, бизнес-правила проверяем на API, через UI оставляем несколько пользовательских сценариев.

## 3. Как браузер показывает страницу

Упрощённо:

1. Браузер получает HTML.
2. Строит DOM.
3. Получает CSS и строит представление стилей.
4. Выполняет JavaScript.
5. JavaScript может изменить DOM.
6. Браузер рассчитывает расположение элементов и рисует страницу.

Для автоматизации важно понимать, что элемент может:

- ещё не существовать в DOM;
- существовать, но быть невидимым;
- быть видимым, но перекрытым;
- быть отключённым;
- появиться заново после перерисовки;
- находиться внутри `iframe`;
- находиться в shadow DOM.

Поэтому «элемент найден» и «с элементом можно взаимодействовать» — разные состояния.

## 4. Что такое DOM

DOM — объектное представление HTML-документа в виде дерева.

```html
<form id="login-form">
  <label for="email">Email</label>
  <input id="email" name="email" type="email">
  <button type="submit">Войти</button>
</form>
```

Элементы связаны отношениями:

- родитель;
- ребёнок;
- предок;
- потомок;
- сосед.

Selenium и Playwright ищут элементы в текущем DOM, а не в исходном HTML-файле на сервере.

## 5. Selenium и WebDriver

Selenium — проект для автоматизации браузеров. В Python чаще всего используют Selenium WebDriver.

Основные участники:

- тестовый код;
- Selenium WebDriver API;
- драйвер браузера;
- браузер.

Тест вызывает метод WebDriver. Команда передаётся браузеру, браузер выполняет действие и возвращает результат.

Современный Selenium умеет автоматически управлять драйверами через Selenium Manager, поэтому ручная загрузка `chromedriver` во многих случаях не нужна.

## 6. Установка Selenium

```bash
pip install selenium
```

Минимальный пример:

```python
from selenium import webdriver


driver = webdriver.Chrome()
try:
    driver.get("https://example.com")
    assert driver.title
finally:
    driver.quit()
```

`quit()` завершает всю браузерную сессию. `close()` закрывает только текущее окно или вкладку.

## 7. WebDriver как pytest-фикстура

```python
from collections.abc import Generator

import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.remote.webdriver import WebDriver


@pytest.fixture
def driver() -> Generator[WebDriver, None, None]:
    options = Options()
    options.add_argument("--window-size=1440,900")

    browser = webdriver.Chrome(options=options)
    browser.set_page_load_timeout(30)

    yield browser

    browser.quit()
```

Для изоляции безопаснее создавать отдельную сессию WebDriver на тест. Если один драйвер используется всеми тестами, cookies, local storage и состояние страницы могут протечь из одного теста в другой.

## 8. Навигация в Selenium

```python
driver.get("https://example.com")
driver.back()
driver.forward()
driver.refresh()

current_url = driver.current_url
title = driver.title
source = driver.page_source
```

`driver.get()` ждёт завершения загрузки в соответствии со стратегией `page_load_strategy`, но это не гарантирует, что асинхронные данные интерфейса уже появились.

## 9. Поиск одного и нескольких элементов

```python
from selenium.webdriver.common.by import By


email = driver.find_element(By.ID, "email")
rows = driver.find_elements(By.CSS_SELECTOR, "table tbody tr")
```

- `find_element` возвращает первый найденный элемент или бросает `NoSuchElementException`.
- `find_elements` возвращает список; если совпадений нет, список будет пустым.

Если ищем внутри компонента:

```python
form = driver.find_element(By.ID, "login-form")
submit = form.find_element(By.CSS_SELECTOR, "button[type='submit']")
```

Так локатор становится точнее и меньше зависит от остальной страницы.

## 10. Стратегии локаторов Selenium

Частые варианты:

```python
By.ID
By.NAME
By.CSS_SELECTOR
By.XPATH
By.CLASS_NAME
By.TAG_NAME
By.LINK_TEXT
By.PARTIAL_LINK_TEXT
```

Приоритет обычно такой:

1. стабильный `data-testid` или другой тестовый атрибут;
2. уникальный `id`;
3. семантический атрибут: `name`, `role`, `aria-label`;
4. короткий CSS-селектор;
5. короткий понятный XPath.

Это не абсолютное правило. Лучший локатор — уникальный, понятный и устойчивый к безопасным изменениям интерфейса.

## 11. CSS-селекторы

```css
#email
.error-message
button[type="submit"]
form#login-form button[type="submit"]
[data-testid="login-button"]
ul.results > li
```

Плюсы:

- обычно короче XPath;
- хорошо знаком frontend-разработчикам;
- удобен для атрибутов и структуры.

Ограничение: обычный CSS не умеет двигаться от дочернего элемента к родителю так свободно, как XPath.

## 12. XPath

```xpath
//input[@name='email']
//button[normalize-space()='Войти']
//label[normalize-space()='Email']/following-sibling::input
//tr[td[normalize-space()='Alex']]//button[@data-action='delete']
```

XPath полезен, когда нужно:

- найти элемент по тексту;
- двигаться по связям между узлами;
- найти элемент относительно подписи или строки таблицы.

Плохой XPath:

```xpath
/html/body/div[2]/div/div[3]/form/div[1]/input
```

Он зависит почти от всей структуры страницы и ломается при небольшом изменении верстки.

## 13. Что такое хороший локатор

Хороший локатор:

- однозначно находит нужный элемент;
- выражает смысл элемента;
- не зависит от случайных CSS-классов;
- не содержит длинного пути по DOM;
- не использует индекс без необходимости;
- согласован с разработчиками.

Хорошо:

```python
LOGIN_BUTTON = (By.CSS_SELECTOR, "[data-testid='login-button']")
```

Рискованно:

```python
LOGIN_BUTTON = (
    By.XPATH,
    "/html/body/div[2]/div[3]/form/div[4]/button[1]",
)
```

Если в продукте нет стабильных атрибутов, полезно договориться с frontend-командой о `data-testid`.

## 14. WebElement

`WebElement` — ссылка Selenium на элемент в браузере.

```python
element.click()
element.send_keys("Alex")
element.clear()

text = element.text
value = element.get_attribute("value")
enabled = element.is_enabled()
visible = element.is_displayed()
selected = element.is_selected()
```

Эта ссылка относится к определённому узлу DOM. Если приложение перерисовало узел, старая ссылка может стать недействительной.

## 15. Почему `time.sleep()` — плохое ожидание

```python
import time

time.sleep(5)
```

Проблемы:

- если элемент появился за 200 мс, тест зря ждёт;
- если пяти секунд не хватило, тест всё равно упадёт;
- большое количество `sleep` сильно замедляет набор;
- причины ожидания не видно из кода.

`sleep` допустим как временный диагностический инструмент, но не как основной механизм синхронизации.

## 16. Неявное ожидание

```python
driver.implicitly_wait(5)
```

Неявное ожидание применяется глобально к поиску элементов. WebDriver повторяет поиск до появления элемента или истечения таймаута.

Недостатки:

- действует на все поиски;
- не ожидает конкретное состояние элемента;
- может скрывать проблемы с локаторами;
- усложняет расчёт времени ожидания.

Официальная документация Selenium предупреждает: не следует смешивать неявные и явные ожидания, потому что итоговое время может стать непредсказуемым.

## 17. Явное ожидание

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait


wait = WebDriverWait(driver, 10)

button = wait.until(
    EC.element_to_be_clickable(
        (By.CSS_SELECTOR, "[data-testid='login-button']")
    )
)
button.click()
```

Явное ожидание ждёт конкретное состояние:

- элемент присутствует;
- элемент видим;
- элемент кликабелен;
- текст появился;
- URL изменился;
- элемент исчез;
- открылось нужное количество окон.

Для UI-тестов это основной механизм синхронизации Selenium.

## 18. Presence, visibility и clickability

- **Presence** — элемент существует в DOM.
- **Visibility** — элемент видим: имеет размер и не скрыт.
- **Clickability** — ожидается, что элемент видим и enabled.

Наличие элемента в DOM не означает, что пользователь уже может с ним работать.

```python
locator = (By.ID, "status")

wait.until(EC.presence_of_element_located(locator))
wait.until(EC.visibility_of_element_located(locator))
```

## 19. Собственное условие ожидания

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.remote.webdriver import WebDriver


def order_has_status(expected_status: str):
    def condition(driver: WebDriver) -> bool:
        actual = driver.find_element(
            By.CSS_SELECTOR,
            "[data-testid='order-status']",
        ).text
        return actual == expected_status

    return condition


wait.until(order_has_status("PAID"))
```

Собственное условие полезно, когда стандартные `expected_conditions` не выражают бизнес-состояние.

## 20. `StaleElementReferenceException`

Ошибка означает: Selenium нашёл элемент, но DOM изменился, и ссылка на прежний узел больше не актуальна.

Плохой подход:

```python
button = driver.find_element(By.ID, "save")
# Страница перерисовалась.
button.click()
```

Лучше повторно найти элемент после изменения:

```python
save_locator = (By.ID, "save")
wait.until(EC.element_to_be_clickable(save_locator)).click()
```

Не нужно бесконечно ловить `StaleElementReferenceException`. Сначала следует понять, какое событие перерисовывает DOM и какое состояние нужно ожидать.

## 21. `ElementClickInterceptedException`

Selenium пытается нажать элемент, но клик получает другой элемент.

Частые причины:

- loader;
- modal;
- cookie banner;
- sticky header;
- анимация;
- элемент находится вне видимой области;
- неверный локатор.

Диагностика:

1. Сделать скриншот.
2. Проверить, что перекрывает элемент.
3. Дождаться исчезновения overlay.
4. Проверить размер окна и прокрутку.
5. Убедиться, что локатор находит правильный элемент.

JavaScript-click не должен быть первым решением: он обходит обычное пользовательское взаимодействие и может скрыть настоящий дефект.

## 22. `ElementNotInteractableException`

Элемент найден, но с ним нельзя выполнить действие.

Возможные причины:

- элемент скрыт;
- выбран невидимый дубликат;
- элемент отключён;
- страница ещё не готова;
- нужно сначала открыть список или модальное окно;
- элемент находится внутри `iframe`.

Нужно проверять состояние и контекст, а не просто увеличивать timeout.

## 23. Работа с полями и кнопками

```python
email = wait.until(
    EC.visibility_of_element_located((By.NAME, "email"))
)
email.clear()
email.send_keys("alex@example.com")

wait.until(
    EC.element_to_be_clickable((By.CSS_SELECTOR, "button[type='submit']"))
).click()
```

Проверяем:

- значение поля;
- placeholder;
- обязательность;
- enabled/disabled;
- маску;
- сообщение об ошибке;
- отправку Enter;
- двойной клик;
- повторную отправку формы.

## 24. Checkbox, radio и select

```python
from selenium.webdriver.support.ui import Select


checkbox = driver.find_element(By.ID, "terms")
if not checkbox.is_selected():
    checkbox.click()

country = Select(driver.find_element(By.ID, "country"))
country.select_by_value("RU")
```

`Select` работает с настоящим HTML-элементом `<select>`. Кастомный dropdown из `div` нужно открывать и выбирать как обычные элементы.

## 25. Клавиатура, мышь и ActionChains

```python
from selenium.webdriver import ActionChains
from selenium.webdriver.common.keys import Keys


menu = driver.find_element(By.ID, "profile-menu")
ActionChains(driver).move_to_element(menu).perform()

search = driver.find_element(By.NAME, "query")
search.send_keys("selenium", Keys.ENTER)
```

`ActionChains` используют для:

- hover;
- drag-and-drop;
- двойного клика;
- контекстного клика;
- сложных комбинаций клавиш.

Сначала стоит попробовать обычный `click()` или `send_keys()`: сложные действия труднее диагностировать.

## 26. Окна и вкладки

```python
original = driver.current_window_handle

driver.find_element(By.LINK_TEXT, "Открыть").click()

wait.until(EC.number_of_windows_to_be(2))

new_window = next(
    handle
    for handle in driver.window_handles
    if handle != original
)

driver.switch_to.window(new_window)
assert "Документ" in driver.title

driver.close()
driver.switch_to.window(original)
```

Нельзя полагаться на случайный порядок окон без явной проверки.

## 27. `iframe`

Элемент внутри `iframe` находится в другом документе. Сначала нужно переключить контекст:

```python
frame = wait.until(
    EC.presence_of_element_located((By.CSS_SELECTOR, "iframe.payment"))
)
driver.switch_to.frame(frame)

driver.find_element(By.NAME, "card-number").send_keys("4111111111111111")

driver.switch_to.default_content()
```

Если элемент виден глазами, но Selenium его не находит, один из первых вопросов — не находится ли он внутри `iframe`.

## 28. Alerts, confirms и prompts

```python
alert = wait.until(EC.alert_is_present())
text = alert.text
alert.accept()
```

Для confirm:

```python
alert.dismiss()
```

Для prompt:

```python
alert.send_keys("Alex")
alert.accept()
```

Пока системный диалог открыт, обычные элементы страницы могут быть недоступны.

## 29. Cookies и browser storage

```python
driver.add_cookie(
    {
        "name": "session_id",
        "value": "test-session",
    }
)

cookie = driver.get_cookie("session_id")
driver.delete_all_cookies()
```

Local storage:

```python
driver.execute_script(
    "window.localStorage.setItem(arguments[0], arguments[1]);",
    "feature_flag",
    "enabled",
)
```

Cookies, local storage и session storage относятся к состоянию браузера. Их нужно очищать или использовать новую сессию между тестами.

## 30. Выполнение JavaScript

```python
value = driver.execute_script(
    "return window.localStorage.getItem(arguments[0]);",
    "feature_flag",
)
```

JavaScript полезен для:

- чтения состояния браузера;
- работы со storage;
- диагностических проверок;
- действий, для которых WebDriver не предоставляет удобного API.

Не следует заменять JavaScript-кликом обычный пользовательский клик без причины.

## 31. Загрузка файла

Для `<input type="file">` не нужно открывать системный диалог:

```python
from pathlib import Path


file_path = Path("fixtures/avatar.png").resolve()
driver.find_element(By.CSS_SELECTOR, "input[type='file']").send_keys(
    str(file_path)
)
```

Проверяем:

- разрешённый тип;
- запрещённый тип;
- размер;
- пустой файл;
- одинаковые имена;
- отмену и повторную загрузку;
- ошибку backend;
- безопасность имени файла.

## 32. Скачивание файла

Надёжнее настроить отдельную временную директорию и дождаться появления завершённого файла.

Проверять следует не только наличие файла:

- имя;
- расширение;
- размер;
- содержимое;
- кодировку;
- отсутствие временного расширения;
- соответствие выбранным фильтрам.

Если файл доступен по API, его содержимое часто выгоднее проверять без браузера, оставив в UI только факт запуска скачивания.

## 33. Скриншоты и артефакты Selenium

```python
driver.save_screenshot("artifacts/failure.png")
```

При падении полезно сохранять:

- screenshot;
- HTML страницы;
- текущий URL;
- название теста;
- browser console logs;
- network logs, если инфраструктура это поддерживает;
- тестовые данные и correlation ID.

Один скриншот не всегда объясняет причину. Он должен дополняться логами и состоянием системы.

## 34. Headless и headed

- **Headed** — браузер виден.
- **Headless** — браузер работает без обычного окна.

Headless удобен в CI и обычно быстрее. Но при локальной диагностике headed-режим помогает увидеть анимации, overlays и последовательность действий.

Тесты не должны проходить только в одном режиме. Если поведение отличается, нужно проверить размер viewport, графическое окружение, шрифты, GPU и настройки браузера.

## 35. Remote WebDriver и Selenium Grid

Grid позволяет запускать браузеры на других машинах или контейнерах.

```python
from selenium import webdriver


driver = webdriver.Remote(
    command_executor="http://selenium-hub:4444",
    options=webdriver.ChromeOptions(),
)
```

Зачем:

- разные браузеры и ОС;
- параллельный запуск;
- централизованная инфраструктура;
- масштабирование CI.

Grid не исправляет нестабильные тесты. При параллельном запуске особенно важны уникальные данные и изоляция.

## 36. Selenium: что важно знать про BiDi

WebDriver BiDi — двунаправленный протокол, который позволяет получать события браузера и управлять некоторыми возможностями, для которых классической модели запрос-ответ недостаточно.

Применения:

- network events;
- console logs;
- события навигации;
- работа с browsing context.

На базовом собеседовании достаточно знать назначение. Не нужно утверждать, что BiDi полностью заменил обычный WebDriver.

## 37. Что такое Playwright

Playwright — инструмент браузерной автоматизации с API для Chromium, Firefox и WebKit.

Важные особенности:

- browser contexts;
- auto-waiting;
- retrying assertions;
- строгие locators;
- перехват сети;
- trace, video и screenshots;
- синхронный и асинхронный Python API;
- официальный pytest-плагин.

## 38. Установка Playwright для Python

```bash
pip install pytest-playwright
playwright install
```

Минимальный тест:

```python
from playwright.sync_api import Page, expect


def test_login(page: Page) -> None:
    page.goto("https://example.com/login")

    page.get_by_label("Email").fill("alex@example.com")
    page.get_by_label("Password").fill("secret")
    page.get_by_role("button", name="Войти").click()

    expect(page).to_have_url("https://example.com/")
```

Официальный pytest-плагин предоставляет фикстуру `page`.

## 39. Browser, BrowserContext и Page

- **Browser** — запущенный процесс браузера.
- **BrowserContext** — изолированный профиль: свои cookies, storage и сессия.
- **Page** — вкладка внутри контекста.

Playwright может держать один Browser и создавать дешёвый новый BrowserContext для каждого теста.

Это даёт изоляцию без запуска отдельного процесса браузера на каждый тест.

## 40. Locators в Playwright

Предпочтительные варианты:

```python
page.get_by_role("button", name="Войти")
page.get_by_label("Email")
page.get_by_placeholder("Поиск")
page.get_by_text("Заказ создан")
page.get_by_test_id("login-button")
```

Локаторы, ориентированные на роль, label и доступное имя, ближе к тому, как интерфейс воспринимает пользователь.

CSS и XPath тоже поддерживаются:

```python
page.locator("[data-testid='login-button']")
page.locator("//button[@type='submit']")
```

Но длинные структурные селекторы остаются хрупкими независимо от инструмента.

## 41. Строгость locators

Действие над locator, который должен обозначать один элемент, завершится ошибкой, если найдено несколько элементов.

```python
page.get_by_role("button", name="Удалить").click()
```

Если таких кнопок несколько, locator нужно уточнить:

```python
row = page.get_by_role("row").filter(has_text="Alex")
row.get_by_role("button", name="Удалить").click()
```

Не следует бездумно использовать `.first`: неоднозначность может означать плохой locator или неверное понимание страницы.

## 42. Auto-waiting и actionability

Перед обычным действием Playwright автоматически проверяет подходящие условия:

- locator однозначен;
- элемент видим;
- стабилен;
- получает события;
- enabled, если это нужно для действия.

```python
page.get_by_role("button", name="Сохранить").click()
```

Это уменьшает количество ручных ожиданий, но не отменяет ожидание бизнес-результата. После клика всё равно нужно проверить, что заказ создан или статус изменился.

## 43. Web-first assertions

```python
from playwright.sync_api import expect


status = page.get_by_test_id("order-status")
expect(status).to_have_text("PAID")
expect(status).to_be_visible()
```

Такие assertions повторяют проверку до успеха или timeout.

Плохо:

```python
assert page.get_by_test_id("order-status").inner_text() == "PAID"
```

Здесь значение читается один раз, поэтому асинхронное обновление может привести к случайному падению.

## 44. Окна, dialogs, downloads и frames в Playwright

Popup:

```python
with page.expect_popup() as popup_info:
    page.get_by_role("link", name="Открыть документ").click()

popup = popup_info.value
expect(popup).to_have_title("Документ")
```

Download:

```python
with page.expect_download() as download_info:
    page.get_by_role("button", name="Скачать").click()

download = download_info.value
download.save_as("artifacts/report.csv")
```

Frame:

```python
payment = page.frame_locator("iframe.payment")
payment.get_by_label("Номер карты").fill("4111111111111111")
```

Dialog:

```python
page.on("dialog", lambda dialog: dialog.accept())
```

Событие нужно начать ожидать до действия, которое его вызывает.

## 45. Авторизация и storage state Playwright

Playwright умеет сохранить состояние браузерного контекста и использовать его в новых тестах.

```python
context.storage_state(path="playwright/.auth/user.json")
```

Затем:

```python
context = browser.new_context(
    storage_state="playwright/.auth/user.json"
)
```

Файл может содержать чувствительные cookies и заголовки. Его нельзя коммитить в репозиторий.

Сохранённая авторизация ускоряет тесты, но отдельные проверки логина должны проходить через настоящий UI.

## 46. Перехват и мокирование сети Playwright

```python
import json

from playwright.sync_api import Route


def mock_user(route: Route) -> None:
    route.fulfill(
        status=200,
        content_type="application/json",
        body=json.dumps(
            {
                "id": 42,
                "name": "Alex",
            }
        ),
    )


page.route("**/api/users/42", mock_user)
page.goto("https://example.com/users/42")
```

Зачем:

- воспроизвести редкую ошибку;
- проверить frontend без нестабильного сервиса;
- управлять скоростью ответа;
- проверить пустые и ошибочные состояния.

Мок не заменяет интеграционный тест с настоящим backend.

## 47. Trace, screenshots и video Playwright

Примеры запуска:

```bash
pytest --tracing retain-on-failure
pytest --screenshot only-on-failure
pytest --video retain-on-failure
```

Trace Viewer показывает:

- действия;
- DOM-снимки до и после;
- screenshots;
- console;
- network;
- source line;
- время выполнения шагов.

Открыть trace:

```bash
playwright show-trace trace.zip
```

Trace особенно полезен для падений в CI, которые сложно воспроизвести локально.

## 48. Sync и async API Playwright

Синхронный стиль:

```python
from playwright.sync_api import Page


def test_title(page: Page) -> None:
    page.goto("https://example.com")
    assert page.title()
```

Асинхронный стиль:

```python
from playwright.async_api import Page


async def test_title(page: Page) -> None:
    await page.goto("https://example.com")
    assert await page.title()
```

Для обычного UI-набора с pytest синхронный API часто проще. Async имеет смысл, когда вся архитектура проекта уже асинхронная или тест управляет несколькими независимыми асинхронными операциями.

Нельзя механически смешивать sync и async API.

## 49. Selenium и Playwright: разница

| Вопрос | Selenium | Playwright |
|---|---|---|
| Основная модель | WebDriver | Собственный automation protocol и browser integrations |
| Браузеры | Chrome, Firefox, Edge, Safari и другие через WebDriver | Chromium, Firefox, WebKit и брендированные Chromium-браузеры |
| Ожидания | Часто явные ожидания | Auto-waiting и retrying assertions |
| Изоляция | Обычно отдельный WebDriver/профиль | BrowserContext на тест |
| Локаторы | CSS, XPath, id и другие стратегии | Role, label, text, test id, CSS, XPath |
| Сеть | Возможности зависят от WebDriver/BiDi и инфраструктуры | Встроенные interception и mocking |
| Диагностика | Screenshots, логи, внешние инструменты | Trace Viewer, screenshots, video, network |
| Экосистема | Очень зрелая и широко распространённая | Более современный API для E2E |

На собеседовании не нужно объявлять один инструмент «лучше вообще». Выбор зависит от браузеров, существующей инфраструктуры, навыков команды и требований проекта.

## 50. Page Object

Page Object прячет локаторы и низкоуровневые действия от теста.

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait


class LoginPage:
    EMAIL = (By.NAME, "email")
    PASSWORD = (By.NAME, "password")
    SUBMIT = (By.CSS_SELECTOR, "button[type='submit']")

    def __init__(self, driver: WebDriver) -> None:
        self.driver = driver
        self.wait = WebDriverWait(driver, 10)

    def open(self) -> "LoginPage":
        self.driver.get("https://example.com/login")
        return self

    def login(self, email: str, password: str) -> None:
        self.wait.until(
            EC.visibility_of_element_located(self.EMAIL)
        ).send_keys(email)
        self.driver.find_element(*self.PASSWORD).send_keys(password)
        self.wait.until(
            EC.element_to_be_clickable(self.SUBMIT)
        ).click()
```

Тест:

```python
def test_user_can_login(driver) -> None:
    LoginPage(driver).open().login(
        email="alex@example.com",
        password="secret",
    )

    assert driver.current_url == "https://example.com/"
```

Page Object не должен превращаться в огромный класс на всю систему.

## 51. Page Component

Повторяющийся блок лучше оформить отдельным компонентом:

```python
class Header:
    def __init__(self, driver) -> None:
        self.driver = driver

    def open_profile(self) -> None:
        self.driver.find_element(
            By.CSS_SELECTOR,
            "[data-testid='profile-menu']",
        ).click()
```

Компоненты:

- header;
- sidebar;
- modal;
- таблица;
- карточка товара;
- date picker.

Композиция компонентов обычно удобнее глубокого наследования `BasePage → AuthorizedPage → AdminPage`.

## 52. Где должны находиться assertions

Универсального запрета нет, но полезно разделять:

- Page Object выполняет действия и предоставляет состояние;
- тест описывает сценарий и бизнес-ожидание.

Хорошо:

```python
login_page.login(user.email, user.password)
assert home_page.is_opened()
```

Внутри Page Object допустимы технические проверки, подтверждающие, что страница готова к работе. Бизнес-assertions лучше оставлять в тесте, чтобы смысл сценария был виден.

## 53. Структура UI-проекта

```text
tests/
  ui/
    test_login.py
    test_orders.py
pages/
  login_page.py
  orders_page.py
components/
  header.py
  modal.py
fixtures/
  browser.py
clients/
  users_api.py
data/
  builders.py
config/
  settings.py
```

Принцип:

- тесты описывают сценарии;
- pages/components знают интерфейс;
- API clients готовят данные;
- fixtures управляют lifecycle;
- builders создают тестовые данные;
- config хранит настройки окружения.

## 54. Подготовка данных для UI-теста

Плохо создавать все данные через UI, если тест проверяет не создание этих данных.

Пример:

1. Через API создать пользователя и заказ.
2. Открыть страницу заказа.
3. Через UI проверить отображение и выполнить нужное действие.
4. Через API удалить данные.

Это ускоряет тест и уменьшает число зависимых шагов.

Нельзя изменять данные напрямую в БД, если это обходит важные бизнес-инварианты и делает состояние нереалистичным.

## 55. Изоляция UI-тестов

Каждый тест должен:

- иметь собственную браузерную сессию или контекст;
- использовать уникальные данные;
- не зависеть от порядка запуска;
- не использовать результат предыдущего теста;
- очищать созданные данные;
- работать отдельно и параллельно.

Selenium рекомендует не разделять изменяемое состояние и создавать новый WebDriver на тест. Playwright изолирует тесты через BrowserContext.

## 56. Параллельный запуск

```bash
pytest -n auto
```

Перед включением параллельности нужно проверить:

- уникальность пользователей и заказов;
- независимость файлов скачивания;
- отсутствие общего WebDriver;
- потокобезопасность отчёта;
- лимиты окружения;
- ограничения Selenium Grid;
- разделение портов и временных директорий.

Если после включения `xdist` тесты падают, проблема часто не в `xdist`, а в общем состоянии.

## 57. Cross-browser testing

Не обязательно запускать весь набор во всех браузерах на каждый commit.

Пример стратегии:

- smoke: Chrome на каждый commit;
- критичный regression: Chrome и Firefox;
- полный cross-browser: по расписанию или перед релизом;
- Safari/WebKit: если это значимая часть пользователей.

Матрица определяется статистикой пользователей и рисками продукта.

## 58. Как бороться с flaky UI-тестами

Частые причины:

- `sleep`;
- хрупкие локаторы;
- общие данные;
- неправильные ожидания;
- анимации и overlays;
- нестабильное окружение;
- зависимость от порядка;
- слишком широкий E2E-сценарий;
- проблема возникает ниже UI;
- разные timezone, locale или viewport.

Алгоритм:

1. Сохранить артефакты.
2. Определить точный шаг и состояние.
3. Проверить локатор.
4. Проверить ожидание.
5. Проверить данные и изоляцию.
6. Проверить network и backend.
7. Упростить сценарий или перенести часть проверки ниже.

`rerun` может временно снизить шум, но не устраняет причину.

## 59. Проверка доступности

Минимум для QA:

- элементы доступны с клавиатуры;
- фокус виден и идёт в логичном порядке;
- кнопки имеют понятные доступные имена;
- поля связаны с label;
- ошибки не передаются только цветом;
- модальное окно удерживает фокус;
- семантические роли соответствуют поведению.

Устойчивые role- и label-локаторы часто одновременно улучшают тестируемость и показывают проблемы доступности.

Автоматические accessibility-проверки полезны, но не заменяют ручную проверку клавиатурой и screen reader.

## 60. Визуальное тестирование

Визуальный тест сравнивает screenshot с эталоном.

Проверяет:

- layout;
- размеры;
- отступы;
- цвета;
- шрифты;
- исчезнувшие элементы.

Источники шума:

- разные ОС и шрифты;
- анимации;
- текущее время;
- случайные данные;
- реклама;
- разный viewport;
- антиалиасинг.

Визуальные проверки нужно запускать в контролируемом окружении и маскировать динамические области.

## 61. Что проверять в браузерных DevTools

При ручной диагностике:

- Console: JavaScript-ошибки;
- Network: запрос, статус, headers, timing, response;
- Elements: реальный DOM и стили;
- Application: cookies, local storage, session storage;
- Performance: долгие операции;
- Security: сертификат и mixed content.

UI-тест упал на сообщении «Не удалось загрузить данные» — это ещё не доказывает frontend-баг. В Network может быть `500`, timeout или неверный контракт backend.

## 62. Как диагностировать падение UI-теста

Нужно ответить на вопросы:

1. На каком действии упал тест?
2. Какой locator использован и сколько элементов найдено?
3. Как выглядела страница?
4. Какой был URL?
5. Были ли console errors?
6. Какие network-запросы выполнялись?
7. Правильные ли данные и окружение?
8. Воспроизводится ли отдельно?
9. Воспроизводится ли вручную?
10. Это дефект продукта, теста, данных или инфраструктуры?

Сильный SDET локализует слой проблемы, а не просто увеличивает timeout.

## 63. Selenium: частые вопросы на собеседовании

### `close()` и `quit()`

- `close()` закрывает текущее окно.
- `quit()` завершает всю WebDriver-сессию.

### `find_element` и `find_elements`

- первый возвращает элемент или исключение;
- второй возвращает список, в том числе пустой.

### Implicit и explicit wait

- implicit действует глобально на поиск;
- explicit ждёт конкретное условие;
- смешивать их не рекомендуется.

### Почему возникает stale element

DOM изменился, и сохранённая ссылка указывает на старый узел.

### CSS или XPath

Выбираем самый устойчивый и понятный locator. CSS часто короче, XPath удобен для текста и отношений между элементами.

## 64. Playwright: частые вопросы на собеседовании

### Что такое BrowserContext

Изолированный браузерный профиль с собственными cookies и storage.

### Что такое auto-waiting

Playwright перед действиями автоматически ждёт, когда элемент удовлетворит необходимым actionability-проверкам.

### Почему `expect(locator)` лучше обычного `assert`

Web-first assertion повторяет проверку до timeout и подходит для асинхронного UI.

### Почему locator лучше ElementHandle

Locator заново разрешается при каждом действии и лучше переносит перерисовку DOM.

### Чем полезен Trace Viewer

Он объединяет действия, DOM snapshots, screenshots, console, network и source для диагностики падения.

## 65. Типовые ошибки кандидатов

- «UI нужно покрыть на 100%».
- «Для ожидания всегда использую sleep».
- «Если тест нестабилен, добавляю rerun».
- «XPath всегда хуже CSS».
- «Playwright вообще не требует ожиданий».
- «Page Object должен содержать все assertions».
- «Один WebDriver на весь набор быстрее и поэтому правильнее».
- «Если тест упал в браузере, это frontend-баг».
- «Headless полностью идентичен headed при любых настройках».
- «JavaScript-click — нормальная замена обычному click».

## 66. Как отвечать «Как бы вы построили UI-автоматизацию»

Пример сильного ответа:

> Сначала я определю критичные пользовательские сценарии и проверю, что их действительно нужно автоматизировать через UI. Данные буду по возможности готовить через API. Тесты построю на pytest, lifecycle браузера вынесу в fixtures, интерфейс — в Page Objects и компоненты. Использую стабильные локаторы и явные ожидания в Selenium либо locators и web-first assertions в Playwright. Каждый тест получит изолированную сессию и уникальные данные. При падении буду сохранять screenshot, URL, DOM, console и network-артефакты. Набор разделю на быстрый smoke и более широкий regression, а cross-browser-матрицу выберу по рискам и статистике пользователей.

## 67. Практические задачи, которые могут дать

Нужно уметь написать:

- фикстуру WebDriver;
- вход на сайт;
- явное ожидание элемента;
- локатор строки таблицы по содержимому;
- переключение в `iframe`;
- работу с новой вкладкой;
- загрузку файла;
- Page Object;
- Playwright-тест с `expect`;
- перехват API-ответа;
- сохранение screenshot при падении;
- параметрический запуск в нескольких браузерах.

## 68. Что повторить перед собеседованием

В первую очередь:

1. DOM и состояния элемента.
2. CSS и XPath.
3. Explicit wait.
4. Основные исключения Selenium.
5. Окна, iframe, alerts и файлы.
6. Page Object и компоненты.
7. Изоляция и тестовые данные.
8. Причины flaky-тестов.
9. BrowserContext, locators и auto-waiting Playwright.
10. Диагностика через screenshots, logs, network и trace.

## Официальные источники

- [Selenium: локаторы](https://www.selenium.dev/documentation/webdriver/elements/locators/)
- [Selenium: ожидания](https://www.selenium.dev/documentation/webdriver/waits/)
- [Selenium: рекомендации по тестам](https://www.selenium.dev/documentation/test_practices/)
- [Selenium: изоляция состояния](https://www.selenium.dev/documentation/test_practices/encouraged/avoid_sharing_state/)
- [Playwright Python: установка](https://playwright.dev/python/docs/intro)
- [Playwright Python: pytest-плагин](https://playwright.dev/python/docs/test-runners)
- [Playwright Python: locators](https://playwright.dev/python/docs/locators)
- [Playwright Python: изоляция](https://playwright.dev/python/docs/browser-contexts)
- [Playwright Python: Trace Viewer](https://playwright.dev/python/docs/trace-viewer)
