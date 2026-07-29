# Web и UI — короткая шпаргалка

[← Все шпаргалки](README.md) · [Полный материал](../materials/web-ui.md)

Короткий повтор перед собеседованием AQA/SDET Python. Здесь собраны определения, отличия, типовые ошибки и минимальные примеры по Selenium и Playwright.

## 1. Что проверяет UI-тест

UI-тест работает с приложением через пользовательский интерфейс:

1. открывает страницу;
2. находит элементы;
3. выполняет действия пользователя;
4. проверяет видимый результат.

UI-тесты хорошо подходят для критичных сквозных сценариев: вход, оформление заказа, оплата, работа основных форм.

Большую часть бизнес-логики выгоднее проверять на уровне API или компонентов: такие тесты быстрее и стабильнее.

## 2. Что такое DOM

DOM — объектное представление HTML-страницы в виде дерева. Инструмент автоматизации ищет узлы этого дерева и взаимодействует с ними.

Важно различать состояния элемента:

- присутствует в DOM;
- видим;
- доступен для взаимодействия;
- включён;
- перекрыт другим элементом.

Наличие элемента в DOM ещё не означает, что по нему уже можно кликнуть.

## 3. Как работает Selenium

Selenium WebDriver управляет настоящим браузером через стандартизированный WebDriver API.

Минимальный пример:

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("https://example.com")
assert "Example" in driver.title
driver.quit()
```

`quit()` завершает всю сессию браузера. `close()` закрывает только текущее окно или вкладку.

## 4. Фикстура браузера в pytest

```python
import pytest
from selenium import webdriver


@pytest.fixture
def driver():
    browser = webdriver.Chrome()
    browser.maximize_window()
    yield browser
    browser.quit()
```

Для изоляции тестов безопаснее создавать отдельную сессию WebDriver на каждый тест.

## 5. Поиск элементов

```python
from selenium.webdriver.common.by import By

button = driver.find_element(By.CSS_SELECTOR, "[data-testid='submit']")
rows = driver.find_elements(By.CSS_SELECTOR, "table tbody tr")
```

- `find_element()` возвращает первый найденный элемент или выбрасывает `NoSuchElementException`.
- `find_elements()` возвращает список. Если совпадений нет, список будет пустым.

## 6. Локаторы

Основные стратегии Selenium:

- `By.ID`;
- `By.NAME`;
- `By.CLASS_NAME`;
- `By.TAG_NAME`;
- `By.LINK_TEXT`;
- `By.PARTIAL_LINK_TEXT`;
- `By.CSS_SELECTOR`;
- `By.XPATH`.

Приоритет обычно такой:

1. стабильный `data-testid`, `data-qa` или аналогичный тестовый атрибут;
2. уникальный стабильный `id`;
3. доступная роль и имя, если инструмент это поддерживает;
4. короткий CSS-селектор;
5. XPath, когда он действительно удобнее.

Плохой локатор:

```text
html > body > div:nth-child(2) > div > button
```

Он зависит от структуры страницы и легко ломается после изменения вёрстки.

Не стоит привязываться к:

- случайным динамическим классам;
- порядку элемента без необходимости;
- длинному абсолютному XPath;
- тексту, который часто меняется или переводится.

## 7. CSS и XPath

CSS:

```css
#login
.submit-button
input[name='email']
[data-testid='save']
form button[type='submit']
```

XPath:

```xpath
//button[@type='submit']
//label[normalize-space()='Email']/following::input[1]
//div[contains(@class, 'alert')]
```

CSS обычно короче и проще. XPath полезен для поиска по тексту, родственным элементам и сложным связям в DOM.

## 8. WebElement

Частые методы:

```python
element.click()
element.send_keys("text")
element.clear()
element.get_attribute("value")
element.is_displayed()
element.is_enabled()
element.is_selected()
```

`element.text` возвращает видимый текст элемента.

## 9. Почему `time.sleep()` — плохое ожидание

```python
time.sleep(5)
```

Такой код всегда ждёт пять секунд:

- если страница готова раньше — тест зря тормозит;
- если страница не готова через пять секунд — тест падает;
- причина ожидания не видна из кода.

Используй ожидание конкретного состояния.

## 10. Implicit wait и explicit wait

Implicit wait задаёт время повторного поиска элемента:

```python
driver.implicitly_wait(5)
```

Explicit wait ждёт определённого условия:

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait

button = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.ID, "submit"))
)
button.click()
```

Explicit wait точнее и лучше показывает намерение теста.

Не смешивай implicit и explicit waits: суммарное время ожидания может стать непредсказуемым.

## 11. Основные условия ожидания

- `presence_of_element_located` — элемент появился в DOM;
- `visibility_of_element_located` — элемент есть и видим;
- `element_to_be_clickable` — элемент видим и включён;
- `invisibility_of_element_located` — элемент исчез или перестал быть видимым;
- `url_contains` — URL содержит ожидаемый фрагмент;
- `title_is` — заголовок страницы совпал;
- `alert_is_present` — появился alert.

Выбирай условие под следующее действие. Перед кликом обычно нужна кликабельность, а не простое присутствие.

## 12. Частые исключения Selenium

### `NoSuchElementException`

Элемент не найден. Возможные причины:

- неправильный локатор;
- элемент ещё не появился;
- выбран не тот iframe;
- открыта не та страница или вкладка.

### `StaleElementReferenceException`

Сохранённый `WebElement` относится к старому узлу DOM. Страница перерисовала этот участок.

Решение: дождаться нужного состояния и найти элемент заново. Не хранить динамический `WebElement` слишком долго.

### `ElementClickInterceptedException`

Клик перехватил другой элемент: loader, модальное окно, sticky-header или анимация.

Решение: дождаться исчезновения перекрытия и кликабельности целевого элемента.

### `ElementNotInteractableException`

Элемент существует, но взаимодействовать с ним сейчас нельзя: он скрыт, отключён или не готов.

## 13. Формы и элементы управления

```python
field.clear()
field.send_keys("user@example.com")
button.click()
```

Для `<select>`:

```python
from selenium.webdriver.support.ui import Select

select = Select(driver.find_element(By.ID, "country"))
select.select_by_value("RU")
```

Для checkbox и radio проверяй `is_selected()` перед кликом, если требуется установить конкретное состояние.

## 14. Окна и вкладки

```python
original = driver.current_window_handle

driver.find_element(By.LINK_TEXT, "Open").click()
WebDriverWait(driver, 10).until(EC.number_of_windows_to_be(2))

new_window = next(
    handle for handle in driver.window_handles if handle != original
)
driver.switch_to.window(new_window)
```

После работы закрой новую вкладку и явно вернись в исходную.

## 15. iframe

Элемент внутри `iframe` нельзя найти из основного документа.

```python
WebDriverWait(driver, 10).until(
    EC.frame_to_be_available_and_switch_to_it((By.CSS_SELECTOR, "iframe"))
)

driver.find_element(By.ID, "pay").click()
driver.switch_to.default_content()
```

## 16. Alerts

```python
alert = WebDriverWait(driver, 10).until(EC.alert_is_present())
text = alert.text
alert.accept()
```

- `accept()` — подтвердить;
- `dismiss()` — отменить;
- `send_keys()` — ввести значение в prompt.

## 17. Cookies и Web Storage

```python
driver.add_cookie({"name": "token", "value": "abc"})
cookies = driver.get_cookies()

driver.execute_script(
    "localStorage.setItem(arguments[0], arguments[1])",
    "theme",
    "dark",
)
```

Перед `add_cookie()` обычно нужно открыть страницу нужного домена.

Cookies и storage удобно использовать для подготовки состояния, но тест не должен случайно зависеть от состояния предыдущего теста.

## 18. Загрузка и скачивание файлов

Файл загружается передачей абсолютного пути в `<input type="file">`:

```python
file_input.send_keys(r"C:\tests\data\report.pdf")
```

Для скачивания заранее настрой каталог загрузки, дождись появления файла и проверь:

- имя;
- расширение;
- размер;
- при необходимости содержимое.

## 19. JavaScript в Selenium

```python
driver.execute_script(
    "arguments[0].scrollIntoView({block: 'center'})",
    element,
)
```

JavaScript — запасной инструмент. Если обычный пользовательский клик не работает, сначала найди причину. Принудительный JS-click может скрыть дефект интерфейса или проблему синхронизации.

## 20. Headless и headed

- headed — браузер с видимым окном, удобен при локальной отладке;
- headless — без видимого окна, удобен в CI.

Падение только в headless часто указывает на разный размер окна, timing, анимацию, шрифты или окружение. Это нужно расследовать, а не просто добавлять `sleep`.

## 21. Selenium Grid

Grid позволяет удалённо запускать тесты:

- в нескольких браузерах;
- на разных версиях;
- на нескольких машинах;
- параллельно.

Grid решает задачу распределённого запуска, но сам по себе не исправляет зависимые или flaky-тесты.

## 22. Что такое Playwright

Playwright — инструмент автоматизации браузеров. Для Python есть официальный pytest-плагин.

Установка:

```bash
pip install pytest-playwright
playwright install
```

Минимальный тест:

```python
from playwright.sync_api import Page, expect


def test_login(page: Page):
    page.goto("https://example.com/login")
    page.get_by_label("Email").fill("user@example.com")
    page.get_by_role("button", name="Sign in").click()
    expect(page.get_by_role("heading", name="Dashboard")).to_be_visible()
```

## 23. Browser, BrowserContext и Page

- `Browser` — запущенный браузер;
- `BrowserContext` — изолированная браузерная сессия;
- `Page` — вкладка внутри контекста.

Контексты одного браузера изолируют cookies и storage. Они создаются быстрее, чем отдельные процессы браузера, поэтому удобны для независимых тестов.

## 24. Локаторы Playwright

Предпочтительные локаторы:

```python
page.get_by_role("button", name="Save")
page.get_by_label("Email")
page.get_by_placeholder("Search")
page.get_by_text("Success")
page.get_by_test_id("submit")
```

Они описывают то, как пользователь или accessibility-дерево воспринимает элемент.

CSS и XPath тоже поддерживаются:

```python
page.locator("[data-testid='submit']")
page.locator("xpath=//button[@type='submit']")
```

## 25. Strictness в Playwright

Операция над одним элементом должна однозначно находить один элемент. Если локатор совпал с несколькими элементами, Playwright обычно сообщит об ошибке strict mode.

Не исправляй неоднозначный локатор механическим `.first`, если по смыслу должен существовать ровно один элемент. Лучше уточнить локатор.

## 26. Auto-waiting

Перед действием Playwright автоматически ждёт, пока элемент:

- будет найден;
- станет видимым;
- перестанет двигаться;
- сможет принимать события;
- будет включён.

Auto-waiting уменьшает количество ручных ожиданий, но не заменяет понимание состояния приложения.

## 27. Web-first assertions

```python
from playwright.sync_api import expect

expect(page.get_by_text("Saved")).to_be_visible()
expect(page).to_have_url("https://example.com/profile")
expect(page.get_by_role("row")).to_have_count(5)
```

Такие проверки повторяются до успеха или таймаута.

Плохо:

```python
assert page.get_by_text("Saved").is_visible()
```

Эта проверка получает состояние только в конкретный момент и может быть flaky.

## 28. Popup, download и dialog в Playwright

Popup:

```python
with page.expect_popup() as popup_info:
    page.get_by_text("Open").click()

popup = popup_info.value
```

Download:

```python
with page.expect_download() as download_info:
    page.get_by_role("button", name="Download").click()

download = download_info.value
download.save_as("report.pdf")
```

Dialog:

```python
page.on("dialog", lambda dialog: dialog.accept())
```

## 29. Аутентификация в Playwright

Состояние авторизации можно сохранить и использовать в новых контекстах:

```python
context.storage_state(path="auth.json")
```

Файл может содержать чувствительные cookies и токены. Не коммить его в Git и не используй одну изменяемую учётную запись в конфликтующих параллельных тестах.

## 30. Работа с сетью в Playwright

```python
page.route(
    "**/api/profile",
    lambda route: route.fulfill(
        status=200,
        content_type="application/json",
        body='{"name": "Test User"}',
    ),
)
```

Перехват сети позволяет:

- подготовить редкий ответ;
- проверить ошибку backend;
- убрать нестабильную внешнюю зависимость;
- дождаться конкретного запроса или ответа.

Но тест с замоканным backend не заменяет настоящий интеграционный сценарий.

## 31. Selenium и Playwright — отличие

| Вопрос | Selenium | Playwright |
|---|---|---|
| Подход | WebDriver-экосистема | Современная browser automation |
| Языки | Много языков | Несколько основных языков |
| Браузеры | Широкая WebDriver-совместимость | Chromium, Firefox, WebKit |
| Ожидания | Часто явные | Много встроенного auto-waiting |
| Изоляция | Обычно новая driver-сессия | Быстрые BrowserContext |
| Сеть и trace | Зависит от инструментов и BiDi | Встроенные возможности |
| Зрелость | Очень большая экосистема | Сильный современный DX |

Выбор зависит от проекта, команды, браузерной матрицы и существующей инфраструктуры. Нельзя честно сказать, что один инструмент всегда лучше другого.

## 32. Page Object

Page Object скрывает детали страницы за понятным интерфейсом:

```python
class LoginPage:
    def __init__(self, page):
        self.page = page
        self.email = page.get_by_label("Email")
        self.password = page.get_by_label("Password")
        self.submit = page.get_by_role("button", name="Sign in")

    def login(self, email: str, password: str) -> None:
        self.email.fill(email)
        self.password.fill(password)
        self.submit.click()
```

Хороший Page Object:

- хранит локаторы и действия страницы;
- использует понятные бизнес-методы;
- не превращается в огромный класс;
- не прячет все проверки без ясной причины.

Повторяющиеся части интерфейса можно вынести в Component Object: header, sidebar, modal, table.

## 33. Структура UI-проекта

```text
tests/
  ui/
pages/
components/
fixtures/
test_data/
utils/
```

Принцип важнее названий каталогов:

- тест описывает сценарий и проверки;
- Page Object работает со страницей;
- фикстуры управляют lifecycle;
- данные отделены от механики UI;
- низкоуровневые детали не дублируются по тестам.

## 34. Независимость тестов

Каждый тест должен:

- сам создавать необходимые данные;
- не зависеть от порядка запуска;
- не использовать результат другого теста;
- очищать созданное состояние;
- иметь отдельную браузерную сессию или контекст.

Данные удобнее готовить через API или БД, если это разрешено задачей: создавать их через UI обычно медленнее и хрупче.

## 35. Параллельный запуск

Перед включением параллельности проверь:

- уникальность пользователей и данных;
- отсутствие общего изменяемого файла;
- отдельные контексты или driver-сессии;
- отсутствие зависимости от порядка тестов;
- достаточность ресурсов CI.

`pytest-xdist`:

```bash
pytest -n auto
```

Параллельный запуск ускоряет suite, но быстрее проявляет проблемы с изоляцией.

## 36. Что такое flaky-тест

Flaky-тест то проходит, то падает без изменения проверяемого поведения.

Частые причины:

- `sleep` вместо ожидания состояния;
- нестабильный локатор;
- общие данные;
- зависимость от порядка;
- анимации и overlays;
- медленная сеть;
- неочищенное состояние;
- слишком жёсткий таймаут;
- различия окружений.

Retry может временно снизить шум, но не устраняет причину.

## 37. Что сохранять при падении

Минимально полезные артефакты:

- screenshot;
- URL;
- название теста;
- stack trace;
- версия браузера;
- HTML или DOM-снимок;
- browser console;
- network logs;
- видео или Playwright trace.

Playwright Trace Viewer позволяет посмотреть действия, DOM-снимки, screenshots, console и network вокруг падения.

## 38. Короткие ответы на частые вопросы

**Почему UI-тесты медленнее API-тестов?**  
Они запускают браузер, рендерят интерфейс и зависят от большего числа компонентов.

**Почему нельзя всё покрыть UI-тестами?**  
Такой suite будет медленным, дорогим в поддержке и нестабильным. На UI оставляют важные пользовательские пути.

**Чем presence отличается от visibility?**  
Presence означает наличие в DOM, visibility — что элемент ещё и видим.

**Почему не надо смешивать implicit и explicit waits?**  
Ожидания влияют друг на друга, и реальный таймаут становится труднее предсказать.

**Что такое stale element?**  
Это ссылка на элемент, который DOM уже заменил или удалил. Элемент нужно найти заново.

**Что лучше: CSS или XPath?**  
Обычно CSS проще. XPath полезен для текста и связей между узлами. Главный критерий — стабильность и читаемость.

**Зачем нужен Page Object?**  
Он отделяет сценарий теста от локаторов и деталей взаимодействия со страницей.

**Почему тесты должны быть независимыми?**  
Чтобы их можно было запускать отдельно, в любом порядке и параллельно.

**Что даёт BrowserContext?**  
Отдельную изолированную сессию с собственными cookies и storage без запуска нового процесса браузера.

**Почему Playwright часто требует меньше явных ожиданий?**  
Locators, действия и web-first assertions используют встроенное повторение и auto-waiting.

**Заменяет ли auto-waiting все ожидания?**  
Нет. Иногда нужно ждать бизнес-событие, ответ API, исчезновение loader или изменение конкретного состояния.

**Что делать с flaky-тестом?**  
Собрать артефакты, найти нестабильное место и исправить синхронизацию, данные, локатор или окружение. Не маскировать проблему бесконечными retry.

## 39. Сильный короткий ответ про UI-архитектуру

> Тесты описывают сценарий и бизнес-проверки, Page и Component Objects скрывают локаторы и действия, а фикстуры управляют браузером, контекстом и данными. Каждый тест изолирован и сам готовит своё состояние. Для синхронизации используются ожидания конкретных условий или auto-waiting, а при падении сохраняются screenshot, логи и trace.

## 40. Что повторить в первую очередь

1. DOM и состояния элемента.
2. CSS, XPath и устойчивые локаторы.
3. Explicit wait и expected conditions.
4. `stale element`, `click intercepted`, `not interactable`.
5. Окна, iframe, alerts и файлы.
6. Page Object и независимость тестов.
7. Playwright locators, BrowserContext, auto-waiting и assertions.
8. Flaky-тесты и диагностика падений.

