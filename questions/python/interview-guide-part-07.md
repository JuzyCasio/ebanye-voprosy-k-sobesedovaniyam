# Python — полный конспект к собеседованию — часть 7

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 45-46.

## 45. Частые задачи на лайвкодинге

1. Удалить дубликаты с сохранением порядка  
def remove_duplicates(items: list[int]) -> list[int]:  
    seen = set()  
    result = []  

    for item in items:  
        if item not in seen:  
            seen.add(item)  
            result.append(item)  

    return result  

### Пример:

print(remove_duplicates([1, 2, 1, 3, 2]))  # [1, 2, 3]  
2. Первый неповторяющийся символ  
from collections import Counter  

def first_unique_char(text: str) -> str | None:  
    counter = Counter(text)  

    for char in text:  
        if counter[char] == 1:  
            return char  

    return None  
3. Развернуть строку  
def reverse_string(text: str) -> str:  
    return text[::-1]  
4. Проверить палиндром  
def is_palindrome(text: str) -> bool:  
    normalized = text.lower().replace(" ", "")  
    return normalized == normalized[::-1]  
5. Посчитать частоту слов  
from collections import Counter  

def count_words(text: str) -> dict[str, int]:  
    words = text.lower().split()  
    return dict(Counter(words))  
6. Сгруппировать заказы по пользователю  
from collections import defaultdict  

def group_orders_by_user(  
    orders: list[dict[str, int]],  
) -> dict[int, int]:  
    result = defaultdict(int)  

    for order in orders:  
        result[order["user_id"]] += order["amount"]  

    return dict(result)  
7. Найти top-N частых элементов  
from collections import Counter  

def top_n(items: list[str], n: int) -> list[str]:  
    counter = Counter(items)  
    return [item for item, _ in counter.most_common(n)]  

### Пример:

items = ["a", "b", "a", "c", "b", "a"]  

print(top_n(items, 2))  # ['a', 'b']  
8. Инвертировать словарь  

Простой случай, когда значения уникальны:  

def invert_dict(data: dict[str, int]) -> dict[int, str]:  
    return {value: key for key, value in data.items()}  

Если значения не уникальны:  

from collections import defaultdict  

def invert_dict_grouped(data: dict[str, int]) -> dict[int, list[str]]:  
    result = defaultdict(list)  

    for key, value in data.items():  
        result[value].append(key)  

    return dict(result)  
9. Сравнить два списка  
def compare_lists(expected: list[int], actual: list[int]) -> dict[str, set[int]]:  
    expected_set = set(expected)  
    actual_set = set(actual)  

    return {  
        "missing": expected_set - actual_set,  
        "extra": actual_set - expected_set,  
        "common": expected_set & actual_set,  
    }  
10. Найти разницу между двумя словарями  
from typing import Any  

def dict_diff(  
    before: dict[str, Any],  
    after: dict[str, Any],  
) -> dict[str, tuple[Any, Any]]:  
    result = {}  

    all_keys = before.keys() | after.keys()  

    for key in all_keys:  
        before_value = before.get(key)  
        after_value = after.get(key)  

        if before_value != after_value:  
            result[key] = (before_value, after_value)  

    return result  
11. Проверить пароль  

Условия:  

минимум 8 символов;  
есть цифра;  
есть заглавная буква.  
def is_valid_password(password: str) -> bool:  
    if len(password) < 8:  
        return False  

    if not any(char.isdigit() for char in password):  
        return False  

    if not any(char.isupper() for char in password):  
        return False  

    return True  
12. Объединить два отсортированных списка  
def merge_sorted(left: list[int], right: list[int]) -> list[int]:  
    result = []  
    i = 0  
    j = 0  

    while i < len(left) and j < len(right):  
        if left[i] <= right[j]:  
            result.append(left[i])  
            i += 1  
        else:  
            result.append(right[j])  
            j += 1  

    result.extend(left[i:])  
    result.extend(right[j:])  

    return result  

## 46. Сложность алгоритмов

Основные обозначения  
Сложность	Что значит  
O(1)	Константное время  
O(log n)	Логарифмическое  
O(n)	Линейное  
O(n log n)	Часто сортировка  
O(n²)	Два вложенных цикла  
O(2ⁿ)	Экспоненциальное  
Примеры  
items = [1, 2, 3, 4, 5]  

print(items[0])  # O(1)  
for item in items:  
    print(item)  # O(n)  
for a in items:  
    for b in items:  
        print(a, b)  # O(n²)  
Словарь  

В среднем:  

data[key]  

Это O(1).  

Но важно:  

В худшем случае операции со словарём могут деградировать, но в нормальных условиях доступ по ключу считается O(1).  

