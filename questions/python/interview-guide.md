# Python â€” Ğ¿Ğ¾Ğ»Ğ½Ñ‹Ğ¹ ĞºĞ¾Ğ½ÑĞ¿ĞµĞºÑ‚ Ğº ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ñ

[â† Ğš Ñ€Ğ°Ğ·Ğ´ĞµĞ»Ñƒ](../python.md) Â· [âš¡ Ğ‘Ñ‹ÑÑ‚Ñ€Ğ°Ñ ÑˆĞ¿Ğ°Ñ€Ğ³Ğ°Ğ»ĞºĞ°](../../cheatsheets/python.md)

ĞŸĞ¾Ğ»Ğ½Ñ‹Ğ¹ Ğ¼Ğ°Ñ‚ĞµÑ€Ğ¸Ğ°Ğ» Ğ´Ğ»Ñ Ğ¿Ğ¾ÑĞ»ĞµĞ´Ğ¾Ğ²Ğ°Ñ‚ĞµĞ»ÑŒĞ½Ğ¾Ğ³Ğ¾ Ğ¸Ğ·ÑƒÑ‡ĞµĞ½Ğ¸Ñ Ñ‚ĞµĞ¼Ñ‹. ĞšĞ¾Ñ€Ğ¾Ñ‚ĞºĞ°Ñ ÑˆĞ¿Ğ°Ñ€Ğ³Ğ°Ğ»ĞºĞ° Ğ½ÑƒĞ¶Ğ½Ğ° Ğ´Ğ»Ñ Ğ¿Ğ¾Ğ²Ñ‚Ğ¾Ñ€ĞµĞ½Ğ¸Ñ, Ğ½Ğ¾ Ğ½Ğµ Ğ·Ğ°Ğ¼ĞµĞ½ÑĞµÑ‚ ÑÑ‚Ğ¾Ñ‚ Ñ€Ğ°Ğ·Ğ´ĞµĞ».

## 2. Ğ‘Ğ°Ğ·Ğ¾Ğ²Ñ‹Ğµ Ñ‚Ğ¸Ğ¿Ñ‹ Ğ´Ğ°Ğ½Ğ½Ñ‹Ñ…

ĞÑĞ½Ğ¾Ğ²Ğ½Ñ‹Ğµ Ñ‚Ğ¸Ğ¿Ñ‹  
x: int = 10  
price: float = 12.5  
name: str = "Alex"  
is_active: bool = True  
nothing: None = None  
ĞšĞ¾Ğ»Ğ»ĞµĞºÑ†Ğ¸Ğ¸  
numbers: list[int] = [1, 2, 3]  
point: tuple[int, int] = (10, 20)  
unique_ids: set[int] = {1, 2, 3}  
user: dict[str, str | int] = {"name": "Alex", "age": 30}  
Ğ˜Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ Ğ¸ Ğ½ĞµĞ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ Ñ‚Ğ¸Ğ¿Ñ‹  
ĞĞµĞ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ  

int, float, bool, str, tuple, frozenset, None  

text = "hello"  
text.upper()  

print(text)  # "hello", ÑÑ‚Ñ€Ğ¾ĞºĞ° Ğ½Ğµ Ğ¸Ğ·Ğ¼ĞµĞ½Ğ¸Ğ»Ğ°ÑÑŒ  
Ğ˜Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ  

list, dict, set, Ğ¾Ğ±ÑŠĞµĞºÑ‚Ñ‹ ĞºĞ»Ğ°ÑÑĞ¾Ğ²  

items = [1, 2, 3]  
items.append(4)  

print(items)  # [1, 2, 3, 4]  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸ Ğ²Ğ°Ğ¶Ğ½Ğ¾ ÑĞºĞ°Ğ·Ğ°Ñ‚ÑŒ:

Ğ’ Python Ğ¿ĞµÑ€ĞµĞ¼ĞµĞ½Ğ½Ğ°Ñ Ñ…Ñ€Ğ°Ğ½Ğ¸Ñ‚ ÑÑÑ‹Ğ»ĞºÑƒ Ğ½Ğ° Ğ¾Ğ±ÑŠĞµĞºÑ‚. ĞĞµĞºĞ¾Ñ‚Ğ¾Ñ€Ñ‹Ğµ Ğ¾Ğ±ÑŠĞµĞºÑ‚Ñ‹ Ğ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ, Ğ½ĞµĞºĞ¾Ñ‚Ğ¾Ñ€Ñ‹Ğµ Ğ½ĞµÑ‚. ĞŸĞ¾ÑÑ‚Ğ¾Ğ¼Ñƒ Ğ¿Ñ€Ğ¸ Ğ¿ĞµÑ€ĞµĞ´Ğ°Ñ‡Ğµ ÑĞ¿Ğ¸ÑĞºĞ° Ğ² Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ Ğ¼Ğ¾Ğ¶Ğ½Ğ¾ Ğ¸Ğ·Ğ¼ĞµĞ½Ğ¸Ñ‚ÑŒ Ğ¸ÑÑ…Ğ¾Ğ´Ğ½Ñ‹Ğ¹ ÑĞ¿Ğ¸ÑĞ¾Ğº.  

### ĞŸÑ€Ğ¸Ğ¼ĞµÑ€:

def add_item(items: list[int]) -> None:  
    items.append(100)  

numbers = [1, 2, 3]  
add_item(numbers)  

print(numbers)  # [1, 2, 3, 100]  

## 3. is Ğ¸ ==

==  

Ğ¡Ñ€Ğ°Ğ²Ğ½Ğ¸Ğ²Ğ°ĞµÑ‚ Ğ·Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ñ.  

a = [1, 2]  
b = [1, 2]  

print(a == b)  # True  
is  

Ğ¡Ñ€Ğ°Ğ²Ğ½Ğ¸Ğ²Ğ°ĞµÑ‚, Ğ¾Ğ´Ğ¸Ğ½ Ğ»Ğ¸ ÑÑ‚Ğ¾ Ğ¾Ğ±ÑŠĞµĞºÑ‚ Ğ² Ğ¿Ğ°Ğ¼ÑÑ‚Ğ¸.  

a = [1, 2]  
b = [1, 2]  

print(a is b)  # False  

ĞŸÑ€Ğ°Ğ²Ğ¸Ğ»ÑŒĞ½Ğ¾Ğµ Ğ¸ÑĞ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğµ:  

if value is None:  
    print("ĞĞµÑ‚ Ğ·Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ñ")  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

== Ğ¿Ñ€Ğ¾Ğ²ĞµÑ€ÑĞµÑ‚ Ñ€Ğ°Ğ²ĞµĞ½ÑÑ‚Ğ²Ğ¾ Ğ·Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ğ¹, Ğ° is Ğ¿Ñ€Ğ¾Ğ²ĞµÑ€ÑĞµÑ‚ Ğ¸Ğ´ĞµĞ½Ñ‚Ğ¸Ñ‡Ğ½Ğ¾ÑÑ‚ÑŒ Ğ¾Ğ±ÑŠĞµĞºÑ‚Ğ¾Ğ². Ğ”Ğ»Ñ None Ğ¿Ñ€Ğ°Ğ²Ğ¸Ğ»ÑŒĞ½Ğ¾ Ğ¸ÑĞ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ñ‚ÑŒ is None.  

## 4. Ğ¡Ğ¿Ğ¸ÑĞºĞ¸

Ğ¡Ğ¾Ğ·Ğ´Ğ°Ğ½Ğ¸Ğµ  
numbers = [1, 2, 3, 4, 5]  
ĞÑĞ½Ğ¾Ğ²Ğ½Ñ‹Ğµ Ğ¾Ğ¿ĞµÑ€Ğ°Ñ†Ğ¸Ğ¸  
numbers.append(6)  
numbers.extend([7, 8])  
numbers.insert(0, 100)  
numbers.remove(3)  
last = numbers.pop()  
Ğ¡Ñ€ĞµĞ·Ñ‹  
items = [10, 20, 30, 40, 50]  

print(items[0])      # 10  
print(items[-1])     # 50  
print(items[1:4])    # [20, 30, 40]  
print(items[::-1])   # [50, 40, 30, 20, 10]  
Ğ§Ğ°ÑÑ‚Ğ°Ñ Ğ¾ÑˆĞ¸Ğ±ĞºĞ°  
items = [1, 2, 3]  
result = items.append(4)  

print(result)  # None  

append() Ğ¼ĞµĞ½ÑĞµÑ‚ ÑĞ¿Ğ¸ÑĞ¾Ğº Ğ½Ğ° Ğ¼ĞµÑÑ‚Ğµ Ğ¸ Ğ²Ğ¾Ğ·Ğ²Ñ€Ğ°Ñ‰Ğ°ĞµÑ‚ None.  

## 5. ĞšĞ¾Ñ€Ñ‚ĞµĞ¶Ğ¸

ĞšĞ¾Ñ€Ñ‚ĞµĞ¶ â€” Ğ½ĞµĞ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ğ°Ñ Ğ¿Ğ¾ÑĞ»ĞµĞ´Ğ¾Ğ²Ğ°Ñ‚ĞµĞ»ÑŒĞ½Ğ¾ÑÑ‚ÑŒ.  

point = (10, 20)  
x, y = point  

Ğ˜ÑĞ¿Ğ¾Ğ»ÑŒĞ·ÑƒĞµÑ‚ÑÑ, ĞºĞ¾Ğ³Ğ´Ğ° Ğ½ÑƒĞ¶Ğ½Ğ¾ Ğ·Ğ°Ñ„Ğ¸ĞºÑĞ¸Ñ€Ğ¾Ğ²Ğ°Ñ‚ÑŒ Ğ½Ğ°Ğ±Ğ¾Ñ€ Ğ·Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ğ¹.  

def get_user() -> tuple[int, str]:  
    return 1, "Alex"  

user_id, username = get_user()  

### Ğ’Ğ°Ğ¶Ğ½Ğ¾:

single = (1,)  

Ğ‘ĞµĞ· Ğ·Ğ°Ğ¿ÑÑ‚Ğ¾Ğ¹ ÑÑ‚Ğ¾ Ğ±ÑƒĞ´ĞµÑ‚ Ğ½Ğµ ĞºĞ¾Ñ€Ñ‚ĞµĞ¶:  

not_tuple = (1)  
print(type(not_tuple))  # int  

## 6. Ğ¡Ğ»Ğ¾Ğ²Ğ°Ñ€Ğ¸

Ğ¡Ğ¾Ğ·Ğ´Ğ°Ğ½Ğ¸Ğµ  
user = {  
    "id": 1,  
    "name": "Alex",  
    "role": "QA",  
}  
Ğ”Ğ¾ÑÑ‚ÑƒĞ¿  
print(user["name"])  

Ğ•ÑĞ»Ğ¸ ĞºĞ»ÑÑ‡Ğ° Ğ½ĞµÑ‚, Ğ±ÑƒĞ´ĞµÑ‚ KeyError.  

Ğ‘ĞµĞ·Ğ¾Ğ¿Ğ°ÑĞ½ĞµĞµ:  

print(user.get("email"))  
print(user.get("email", "unknown"))  
ĞĞ±Ñ…Ğ¾Ğ´  
for key in user:  
    print(key)  

for key, value in user.items():  
    print(key, value)  

for value in user.values():  
    print(value)  
ĞŸÑ€Ğ¾Ğ²ĞµÑ€ĞºĞ° ĞºĞ»ÑÑ‡Ğ°  
if "name" in user:  
    print(user["name"])  
ĞĞ±ÑŠĞµĞ´Ğ¸Ğ½ĞµĞ½Ğ¸Ğµ ÑĞ»Ğ¾Ğ²Ğ°Ñ€ĞµĞ¹  
a = {"x": 1}  
b = {"y": 2}  

result = a | b  
print(result)  # {'x': 1, 'y': 2}  

Ğ˜Ğ»Ğ¸ ÑÑ‚Ğ°Ñ€Ñ‹Ğ¹ ÑĞ¿Ğ¾ÑĞ¾Ğ±:  

result = {**a, **b}  

## 7. ĞœĞ½Ğ¾Ğ¶ĞµÑÑ‚Ğ²Ğ°

ĞœĞ½Ğ¾Ğ¶ĞµÑÑ‚Ğ²Ğ¾ Ñ…Ñ€Ğ°Ğ½Ğ¸Ñ‚ ÑƒĞ½Ğ¸ĞºĞ°Ğ»ÑŒĞ½Ñ‹Ğµ ÑĞ»ĞµĞ¼ĞµĞ½Ñ‚Ñ‹.  

ids = {1, 2, 3, 3}  
print(ids)  # {1, 2, 3}  
ĞĞ¿ĞµÑ€Ğ°Ñ†Ğ¸Ğ¸  
a = {1, 2, 3}  
b = {3, 4, 5}  

print(a | b)  # Ğ¾Ğ±ÑŠĞµĞ´Ğ¸Ğ½ĞµĞ½Ğ¸Ğµ: {1, 2, 3, 4, 5}  
print(a & b)  # Ğ¿ĞµÑ€ĞµÑĞµÑ‡ĞµĞ½Ğ¸Ğµ: {3}  
print(a - b)  # Ñ€Ğ°Ğ·Ğ½Ğ¾ÑÑ‚ÑŒ: {1, 2}  
print(a ^ b)  # ÑĞ¸Ğ¼Ğ¼ĞµÑ‚Ñ€Ğ¸Ñ‡Ğ½Ğ°Ñ Ñ€Ğ°Ğ·Ğ½Ğ¾ÑÑ‚ÑŒ: {1, 2, 4, 5}  

### Ğ“Ğ´Ğµ Ğ¿Ñ€Ğ¸Ğ¼ĞµĞ½Ğ¸Ğ¼Ğ¾ Ğ² Ñ‚ĞµÑÑ‚Ğ¸Ñ€Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

expected_ids = {1, 2, 3}  
actual_ids = {2, 3, 4}  

missing = expected_ids - actual_ids  
extra = actual_ids - expected_ids  

print(missing)  # {1}  
print(extra)    # {4}  

## 8. Hashable / unhashable

Ğ¥ĞµÑˆĞ¸Ñ€ÑƒĞµĞ¼Ñ‹Ğµ Ğ¾Ğ±ÑŠĞµĞºÑ‚Ñ‹ Ğ¼Ğ¾Ğ¶Ğ½Ğ¾ Ğ¸ÑĞ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ñ‚ÑŒ ĞºĞ°Ğº ĞºĞ»ÑÑ‡Ğ¸ ÑĞ»Ğ¾Ğ²Ğ°Ñ€Ñ Ğ¸Ğ»Ğ¸ ÑĞ»ĞµĞ¼ĞµĞ½Ñ‚Ñ‹ Ğ¼Ğ½Ğ¾Ğ¶ĞµÑÑ‚Ğ²Ğ°.  

ĞœĞ¾Ğ¶Ğ½Ğ¾:  

data = {  
    "name": "Alex",  
    1: "one",  
    (1, 2): "point",  
}  

ĞĞµĞ»ÑŒĞ·Ñ:  

data = {  
    [1, 2]: "bad"  
}  

Ğ‘ÑƒĞ´ĞµÑ‚ Ğ¾ÑˆĞ¸Ğ±ĞºĞ°:  

TypeError: unhashable type: 'list'  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

ĞšĞ»ÑÑ‡ ÑĞ»Ğ¾Ğ²Ğ°Ñ€Ñ Ğ´Ğ¾Ğ»Ğ¶ĞµĞ½ Ğ±Ñ‹Ñ‚ÑŒ hashable, Ñ‚Ğ¾ ĞµÑÑ‚ÑŒ Ğ¸Ğ¼ĞµÑ‚ÑŒ ÑÑ‚Ğ°Ğ±Ğ¸Ğ»ÑŒĞ½Ñ‹Ğ¹ hash Ğ¸ ĞºĞ¾Ñ€Ñ€ĞµĞºÑ‚Ğ½Ğ¾Ğµ ÑÑ€Ğ°Ğ²Ğ½ĞµĞ½Ğ¸Ğµ. Ğ¡Ğ¿Ğ¸ÑĞºĞ¸ Ğ¸ ÑĞ»Ğ¾Ğ²Ğ°Ñ€Ğ¸ Ğ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ, Ğ¿Ğ¾ÑÑ‚Ğ¾Ğ¼Ñƒ Ğ¾Ğ½Ğ¸ Ğ½Ğµ Ğ¼Ğ¾Ğ³ÑƒÑ‚ Ğ±Ñ‹Ñ‚ÑŒ ĞºĞ»ÑÑ‡Ğ°Ğ¼Ğ¸.  

## 9. List comprehension

ĞĞ±Ñ‹Ñ‡Ğ½Ñ‹Ğ¹ Ñ†Ğ¸ĞºĞ»:  

result = []  

for number in range(10):  
    if number % 2 == 0:  
        result.append(number * number)  

Ğ§ĞµÑ€ĞµĞ· comprehension:  

result = [number * number for number in range(10) if number % 2 == 0]  

### ĞŸÑ€Ğ¸Ğ¼ĞµÑ€:

users = [  
    {"id": 1, "active": True},  
    {"id": 2, "active": False},  
    {"id": 3, "active": True},  
]  

active_ids = [user["id"] for user in users if user["active"]]  

print(active_ids)  # [1, 3]  

## 10. Dict comprehension

users = [  
    {"id": 1, "name": "Alex"},  
    {"id": 2, "name": "Ivan"},  
]  

users_by_id = {user["id"]: user for user in users}  

print(users_by_id)  

Ğ ĞµĞ·ÑƒĞ»ÑŒÑ‚Ğ°Ñ‚:  

{  
    1: {"id": 1, "name": "Alex"},  
    2: {"id": 2, "name": "Ivan"},  
}  

ĞÑ‡ĞµĞ½ÑŒ Ñ‡Ğ°ÑÑ‚Ğ°Ñ Ğ·Ğ°Ğ´Ğ°Ñ‡Ğ° Ğ½Ğ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸.  

## 11. Ğ¤ÑƒĞ½ĞºÑ†Ğ¸Ğ¸

ĞŸÑ€Ğ¾ÑÑ‚Ğ°Ñ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ  
def add(a: int, b: int) -> int:  
    return a + b  
Ğ—Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ñ Ğ¿Ğ¾ ÑƒĞ¼Ğ¾Ğ»Ñ‡Ğ°Ğ½Ğ¸Ñ  
def greet(name: str = "Guest") -> str:  
    return f"Hello, {name}"  
Ğ’Ğ°Ğ¶Ğ½Ğ°Ñ Ğ¾ÑˆĞ¸Ğ±ĞºĞ° Ñ mutable default argument  

### ĞŸĞ»Ğ¾Ñ…Ğ¾:

def add_item(item: str, items: list[str] = []) -> list[str]:  
    items.append(item)  
    return items  

### ĞŸÑ€Ğ¾Ğ±Ğ»ĞµĞ¼Ğ°:

print(add_item("a"))  # ['a']  
print(add_item("b"))  # ['a', 'b']  

ĞŸÑ€Ğ°Ğ²Ğ¸Ğ»ÑŒĞ½Ğ¾:  

def add_item(item: str, items: list[str] | None = None) -> list[str]:  
    if items is None:  
        items = []  

    items.append(item)  
    return items  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

Ğ—Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ñ Ğ¿Ğ¾ ÑƒĞ¼Ğ¾Ğ»Ñ‡Ğ°Ğ½Ğ¸Ñ Ğ²Ñ‹Ñ‡Ğ¸ÑĞ»ÑÑÑ‚ÑÑ Ğ¾Ğ´Ğ¸Ğ½ Ñ€Ğ°Ğ· Ğ¿Ñ€Ğ¸ ÑĞ¾Ğ·Ğ´Ğ°Ğ½Ğ¸Ğ¸ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ğ¸, Ğ° Ğ½Ğµ Ğ¿Ñ€Ğ¸ ĞºĞ°Ğ¶Ğ´Ğ¾Ğ¼ Ğ²Ñ‹Ğ·Ğ¾Ğ²Ğµ. ĞŸĞ¾ÑÑ‚Ğ¾Ğ¼Ñƒ Ğ¸Ğ·Ğ¼ĞµĞ½ÑĞµĞ¼Ñ‹Ğµ Ğ·Ğ½Ğ°Ñ‡ĞµĞ½Ğ¸Ñ Ğ¿Ğ¾ ÑƒĞ¼Ğ¾Ğ»Ñ‡Ğ°Ğ½Ğ¸Ñ Ğ¼Ğ¾Ğ³ÑƒÑ‚ Ğ¿Ñ€Ğ¸Ğ²ĞµÑÑ‚Ğ¸ Ğº Ğ½ĞµĞ¾Ğ¶Ğ¸Ğ´Ğ°Ğ½Ğ½Ğ¾Ğ¼Ñƒ Ğ¿Ğ¾Ğ²ĞµĞ´ĞµĞ½Ğ¸Ñ.  

## 12. *args Ğ¸ **kwargs

def func(*args: int, **kwargs: str) -> None:  
    print(args)  
    print(kwargs)  

func(1, 2, 3, name="Alex", role="QA")  

Ğ ĞµĞ·ÑƒĞ»ÑŒÑ‚Ğ°Ñ‚:  

(1, 2, 3)  
{'name': 'Alex', 'role': 'QA'}  
ĞšĞ¾Ğ³Ğ´Ğ° Ğ¸ÑĞ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ñ‚ÑŒ  

*args â€” ĞºĞ¾Ğ³Ğ´Ğ° Ğ½ĞµĞ¸Ğ·Ğ²ĞµÑÑ‚Ğ½Ğ¾ ĞºĞ¾Ğ»Ğ¸Ñ‡ĞµÑÑ‚Ğ²Ğ¾ Ğ¿Ğ¾Ğ·Ğ¸Ñ†Ğ¸Ğ¾Ğ½Ğ½Ñ‹Ñ… Ğ°Ñ€Ğ³ÑƒĞ¼ĞµĞ½Ñ‚Ğ¾Ğ².  

**kwargs â€” ĞºĞ¾Ğ³Ğ´Ğ° Ğ½ĞµĞ¸Ğ·Ğ²ĞµÑÑ‚Ğ½Ğ¾ ĞºĞ¾Ğ»Ğ¸Ñ‡ĞµÑÑ‚Ğ²Ğ¾ Ğ¸Ğ¼ĞµĞ½Ğ¾Ğ²Ğ°Ğ½Ğ½Ñ‹Ñ… Ğ°Ñ€Ğ³ÑƒĞ¼ĞµĞ½Ñ‚Ğ¾Ğ².  

### ĞŸÑ€Ğ¸Ğ¼ĞµÑ€:

def make_request(method: str, url: str, **kwargs: object) -> None:  
    print(method)  
    print(url)  
    print(kwargs)  

make_request(  
    "GET",  
    "https://example.com/users",  
    timeout=5,  
    headers={"Authorization": "token"},  
)  

## 13. ĞĞ±Ğ»Ğ°ÑÑ‚Ğ¸ Ğ²Ğ¸Ğ´Ğ¸Ğ¼Ğ¾ÑÑ‚Ğ¸ LEGB

Python Ğ¸Ñ‰ĞµÑ‚ Ğ¿ĞµÑ€ĞµĞ¼ĞµĞ½Ğ½Ñ‹Ğµ Ğ² Ğ¿Ğ¾Ñ€ÑĞ´ĞºĞµ:  

Local â€” Ğ»Ğ¾ĞºĞ°Ğ»ÑŒĞ½Ğ°Ñ Ğ¾Ğ±Ğ»Ğ°ÑÑ‚ÑŒ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ğ¸.  
Enclosing â€” Ğ¾Ğ±Ğ»Ğ°ÑÑ‚ÑŒ Ğ²Ğ½ĞµÑˆĞ½ĞµĞ¹ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ğ¸.  
Global â€” Ğ³Ğ»Ğ¾Ğ±Ğ°Ğ»ÑŒĞ½Ğ°Ñ Ğ¾Ğ±Ğ»Ğ°ÑÑ‚ÑŒ Ğ¼Ğ¾Ğ´ÑƒĞ»Ñ.  
Built-in â€” Ğ²ÑÑ‚Ñ€Ğ¾ĞµĞ½Ğ½Ñ‹Ğµ Ğ¸Ğ¼ĞµĞ½Ğ°.  

### ĞŸÑ€Ğ¸Ğ¼ĞµÑ€:

name = "global"  

def outer() -> None:  
    name = "outer"  

    def inner() -> None:  
        name = "inner"  
        print(name)  

    inner()  

outer()  # inner  
global  
counter = 0  

def increment() -> None:  
    global counter  
    counter += 1  
nonlocal  
def make_counter() -> callable:  
    count = 0  

    def increment() -> int:  
        nonlocal count  
        count += 1  
        return count  

    return increment  

counter = make_counter()  

print(counter())  # 1  
print(counter())  # 2  

## 14. Lambda

users = [  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
]  

users.sort(key=lambda user: user["age"])  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

lambda â€” ÑÑ‚Ğ¾ ĞºĞ¾Ñ€Ğ¾Ñ‚ĞºĞ°Ñ Ğ°Ğ½Ğ¾Ğ½Ğ¸Ğ¼Ğ½Ğ°Ñ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ. Ğ§Ğ°ÑÑ‚Ğ¾ Ğ¸ÑĞ¿Ğ¾Ğ»ÑŒĞ·ÑƒĞµÑ‚ÑÑ ĞºĞ°Ğº key Ğ´Ğ»Ñ ÑĞ¾Ñ€Ñ‚Ğ¸Ñ€Ğ¾Ğ²ĞºĞ¸, Ñ„Ğ¸Ğ»ÑŒÑ‚Ñ€Ğ°Ñ†Ğ¸Ğ¸ Ğ¸Ğ»Ğ¸ Ğ¼Ğ°Ğ¿Ğ¿Ğ¸Ğ½Ğ³Ğ°.  

## 15. Ğ¡Ğ¾Ñ€Ñ‚Ğ¸Ñ€Ğ¾Ğ²ĞºĞ°

sorted()  

Ğ’Ğ¾Ğ·Ğ²Ñ€Ğ°Ñ‰Ğ°ĞµÑ‚ Ğ½Ğ¾Ğ²Ñ‹Ğ¹ ÑĞ¿Ğ¸ÑĞ¾Ğº.  

numbers = [3, 1, 2]  

result = sorted(numbers)  

print(result)   # [1, 2, 3]  
print(numbers)  # [3, 1, 2]  
.sort()  

ĞœĞµĞ½ÑĞµÑ‚ ÑĞ¿Ğ¸ÑĞ¾Ğº Ğ½Ğ° Ğ¼ĞµÑÑ‚Ğµ.  

numbers = [3, 1, 2]  
numbers.sort()  

print(numbers)  # [1, 2, 3]  
Ğ¡Ğ¾Ñ€Ñ‚Ğ¸Ñ€Ğ¾Ğ²ĞºĞ° ÑĞ¿Ğ¸ÑĞºĞ° ÑĞ»Ğ¾Ğ²Ğ°Ñ€ĞµĞ¹  
users = [  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
    {"name": "Petr", "age": 35},  
]  

users_sorted = sorted(users, key=lambda user: user["age"])  
Ğ¡Ğ¾Ñ€Ñ‚Ğ¸Ñ€Ğ¾Ğ²ĞºĞ° Ğ¿Ğ¾ Ğ½ĞµÑĞºĞ¾Ğ»ÑŒĞºĞ¸Ğ¼ Ğ¿Ğ¾Ğ»ÑĞ¼  
users = [  
    {"name": "Bob", "age": 30},  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
]  

result = sorted(users, key=lambda user: (user["age"], user["name"]))  

## 16. ĞšĞ¾Ğ¿Ğ¸Ñ€Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğµ Ğ¾Ğ±ÑŠĞµĞºÑ‚Ğ¾Ğ²

ĞŸĞ¾Ğ²ĞµÑ€Ñ…Ğ½Ğ¾ÑÑ‚Ğ½Ğ°Ñ ĞºĞ¾Ğ¿Ğ¸Ñ  
from copy import copy  

a = [[1, 2], [3, 4]]  
b = copy(a)  

b[0].append(100)  

print(a)  # [[1, 2, 100], [3, 4]]  

Ğ¡ĞºĞ¾Ğ¿Ğ¸Ñ€Ğ¾Ğ²Ğ°Ğ»ÑÑ Ğ²Ğ½ĞµÑˆĞ½Ğ¸Ğ¹ ÑĞ¿Ğ¸ÑĞ¾Ğº, Ğ½Ğ¾ Ğ²Ğ»Ğ¾Ğ¶ĞµĞ½Ğ½Ñ‹Ğµ ÑĞ¿Ğ¸ÑĞºĞ¸ Ğ¾ÑÑ‚Ğ°Ğ»Ğ¸ÑÑŒ Ğ¾Ğ±Ñ‰Ğ¸Ğ¼Ğ¸.  

Ğ“Ğ»ÑƒĞ±Ğ¾ĞºĞ°Ñ ĞºĞ¾Ğ¿Ğ¸Ñ  
from copy import deepcopy  

a = [[1, 2], [3, 4]]  
b = deepcopy(a)  

b[0].append(100)  

print(a)  # [[1, 2], [3, 4]]  
print(b)  # [[1, 2, 100], [3, 4]]  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

copy ĞºĞ¾Ğ¿Ğ¸Ñ€ÑƒĞµÑ‚ Ñ‚Ğ¾Ğ»ÑŒĞºĞ¾ Ğ²Ğ½ĞµÑˆĞ½Ğ¸Ğ¹ Ğ¾Ğ±ÑŠĞµĞºÑ‚, deepcopy Ñ€ĞµĞºÑƒÑ€ÑĞ¸Ğ²Ğ½Ğ¾ ĞºĞ¾Ğ¿Ğ¸Ñ€ÑƒĞµÑ‚ Ğ²Ğ»Ğ¾Ğ¶ĞµĞ½Ğ½Ñ‹Ğµ Ğ¾Ğ±ÑŠĞµĞºÑ‚Ñ‹.  

## 17. Ğ˜ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ñ

Ğ‘Ğ°Ğ·Ğ¾Ğ²Ñ‹Ğ¹ Ğ¿Ñ€Ğ¸Ğ¼ĞµÑ€  
try:  
    result = 10 / 0  
except ZeroDivisionError:  
    print("Ğ”ĞµĞ»ĞµĞ½Ğ¸Ğµ Ğ½Ğ° Ğ½Ğ¾Ğ»ÑŒ")  
ĞĞµÑĞºĞ¾Ğ»ÑŒĞºĞ¾ Ğ¸ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ğ¹  
try:  
    value = int("abc")  
except ValueError:  
    print("ĞÑˆĞ¸Ğ±ĞºĞ° Ğ¿Ñ€ĞµĞ¾Ğ±Ñ€Ğ°Ğ·Ğ¾Ğ²Ğ°Ğ½Ğ¸Ñ")  
except TypeError:  
    print("ĞĞµĞ²ĞµÑ€Ğ½Ñ‹Ğ¹ Ñ‚Ğ¸Ğ¿")  
else  

Ğ’Ñ‹Ğ¿Ğ¾Ğ»Ğ½ÑĞµÑ‚ÑÑ, ĞµÑĞ»Ğ¸ Ğ¸ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ñ Ğ½Ğµ Ğ±Ñ‹Ğ»Ğ¾.  

try:  
    value = int("123")  
except ValueError:  
    print("ĞÑˆĞ¸Ğ±ĞºĞ°")  
else:  
    print("Ğ£ÑĞ¿ĞµÑˆĞ½Ğ¾")  
finally  

Ğ’Ñ‹Ğ¿Ğ¾Ğ»Ğ½ÑĞµÑ‚ÑÑ Ğ²ÑĞµĞ³Ğ´Ğ°.  

try:  
    file = open("data.txt")  
except FileNotFoundError:  
    print("Ğ¤Ğ°Ğ¹Ğ» Ğ½Ğµ Ğ½Ğ°Ğ¹Ğ´ĞµĞ½")  
finally:  
    print("Ğ—Ğ°Ğ²ĞµÑ€ÑˆĞµĞ½Ğ¸Ğµ")  
Ğ¡Ğ¾Ğ·Ğ´Ğ°Ğ½Ğ¸Ğµ ÑĞ²Ğ¾ĞµĞ³Ğ¾ Ğ¸ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ñ  
class UserNotFoundError(Exception):  
    pass  

def get_user(user_id: int) -> dict[str, object]:  
    if user_id <= 0:  
        raise UserNotFoundError(f"User with id={user_id} not found")  

    return {"id": user_id}  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

Ğ˜ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ñ Ğ½ÑƒĞ¶Ğ½Ñ‹ Ğ´Ğ»Ñ Ğ¾Ğ±Ñ€Ğ°Ğ±Ğ¾Ñ‚ĞºĞ¸ Ğ¾ÑˆĞ¸Ğ±Ğ¾Ñ‡Ğ½Ñ‹Ñ… ÑÑ†ĞµĞ½Ğ°Ñ€Ğ¸ĞµĞ². Ğ’ Ñ‚ĞµÑÑ‚Ğ¾Ğ²Ğ¾Ğ¼ Ñ„Ñ€ĞµĞ¹Ğ¼Ğ²Ğ¾Ñ€ĞºĞµ Ñ Ğ±Ñ‹ ÑĞ¾Ğ·Ğ´Ğ°Ğ²Ğ°Ğ» ÑĞ¾Ğ±ÑÑ‚Ğ²ĞµĞ½Ğ½Ñ‹Ğµ Ğ¸ÑĞºĞ»ÑÑ‡ĞµĞ½Ğ¸Ñ Ğ´Ğ»Ñ Ğ¿Ğ¾Ğ½ÑÑ‚Ğ½Ñ‹Ñ… Ğ¾ÑˆĞ¸Ğ±Ğ¾Ğº: Ğ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ñ‚ĞµĞ»ÑŒ Ğ½Ğµ ÑĞ¾Ğ·Ğ´Ğ°Ğ½, ÑÑ‚ĞµĞ½Ğ´ Ğ½ĞµĞ´Ğ¾ÑÑ‚ÑƒĞ¿ĞµĞ½, Ğ½ĞµĞºĞ¾Ñ€Ñ€ĞµĞºÑ‚Ğ½Ñ‹Ğ¹ Ğ¾Ñ‚Ğ²ĞµÑ‚ API.  

## 18. ĞšĞ¾Ğ½Ñ‚ĞµĞºÑÑ‚Ğ½Ñ‹Ğ¹ Ğ¼ĞµĞ½ĞµĞ´Ğ¶ĞµÑ€ with

ĞšĞ¾Ğ½Ñ‚ĞµĞºÑÑ‚Ğ½Ñ‹Ğ¹ Ğ¼ĞµĞ½ĞµĞ´Ğ¶ĞµÑ€ ÑƒĞ¿Ñ€Ğ°Ğ²Ğ»ÑĞµÑ‚ Ñ€ĞµÑÑƒÑ€ÑĞ¾Ğ¼: Ğ¾Ñ‚ĞºÑ€Ñ‹Ñ‚ÑŒ/Ğ·Ğ°ĞºÑ€Ñ‹Ñ‚ÑŒ Ñ„Ğ°Ğ¹Ğ», ÑĞ¾ĞµĞ´Ğ¸Ğ½ĞµĞ½Ğ¸Ğµ, lock, ÑĞµÑÑĞ¸Ñ.  

with open("data.txt", "r", encoding="utf-8") as file:  
    content = file.read()  

Ğ¤Ğ°Ğ¹Ğ» Ğ·Ğ°ĞºÑ€Ğ¾ĞµÑ‚ÑÑ Ğ°Ğ²Ñ‚Ğ¾Ğ¼Ğ°Ñ‚Ğ¸Ñ‡ĞµÑĞºĞ¸.  

Ğ¡Ğ²Ğ¾Ğ¹ ĞºĞ¾Ğ½Ñ‚ĞµĞºÑÑ‚Ğ½Ñ‹Ğ¹ Ğ¼ĞµĞ½ĞµĞ´Ğ¶ĞµÑ€ Ñ‡ĞµÑ€ĞµĞ· ĞºĞ»Ğ°ÑÑ  
class FileManager:  
    def __init__(self, path: str) -> None:  
        self.path = path  
        self.file = None  

    def __enter__(self):  
        self.file = open(self.path, "r", encoding="utf-8")  
        return self.file  

    def __exit__(self, exc_type, exc_value, traceback) -> None:  
        if self.file:  
            self.file.close()  

with FileManager("data.txt") as file:  
    print(file.read())  
Ğ§ĞµÑ€ĞµĞ· contextmanager  
from collections.abc import Generator  
from contextlib import contextmanager  

@contextmanager  
def open_file(path: str) -> Generator:  
    file = open(path, "r", encoding="utf-8")  
    try:  
        yield file  
    finally:  
        file.close()  

Ğ˜ÑĞ¿Ğ¾Ğ»ÑŒĞ·Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğµ:  

with open_file("data.txt") as file:  
    print(file.read())  

### ĞĞ° ÑĞ¾Ğ±ĞµÑĞµĞ´Ğ¾Ğ²Ğ°Ğ½Ğ¸Ğ¸:

with Ğ³Ğ°Ñ€Ğ°Ğ½Ñ‚Ğ¸Ñ€ÑƒĞµÑ‚ ĞºĞ¾Ñ€Ñ€ĞµĞºÑ‚Ğ½Ğ¾Ğµ Ğ¾ÑĞ²Ğ¾Ğ±Ğ¾Ğ¶Ğ´ĞµĞ½Ğ¸Ğµ Ñ€ĞµÑÑƒÑ€ÑĞ° Ğ´Ğ°Ğ¶Ğµ Ğ¿Ñ€Ğ¸ Ğ¾ÑˆĞ¸Ğ±ĞºĞµ Ğ²Ğ½ÑƒÑ‚Ñ€Ğ¸ Ğ±Ğ»Ğ¾ĞºĞ°.  

## 19. Ğ”ĞµĞºĞ¾Ñ€Ğ°Ñ‚Ğ¾Ñ€Ñ‹

Ğ”ĞµĞºĞ¾Ñ€Ğ°Ñ‚Ğ¾Ñ€ â€” Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ, ĞºĞ¾Ñ‚Ğ¾Ñ€Ğ°Ñ Ğ¿Ñ€Ğ¸Ğ½Ğ¸Ğ¼Ğ°ĞµÑ‚ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ Ğ¸ Ğ²Ğ¾Ğ·Ğ²Ñ€Ğ°Ñ‰Ğ°ĞµÑ‚ Ğ½Ğ¾Ğ²ÑƒÑ Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ñ.  

ĞŸÑ€Ğ¾ÑÑ‚Ğ¾Ğ¹ Ğ¿Ñ€Ğ¸Ğ¼ĞµÑ€  
from collections.abc import Callable  
from functools import wraps  

def log_call(func: Callable) -> Callable:  
    @wraps(func)  
    def wrapper(*args, **kwargs):  
        print(f"Ğ’Ñ‹Ğ·Ğ¾Ğ² Ñ„ÑƒĞ½ĞºÑ†Ğ¸Ğ¸: {func.__name__}")  
        return func(*args, **kwargs)  

    return wrapper  

@log_call  
def add(a: int, b: int) -> int:  
    return a + b  

print(add(2, 3))  
Ğ§Ñ‚Ğ¾ Ğ¿Ñ€Ğ¾Ğ¸ÑÑ…Ğ¾Ğ´Ğ¸Ñ‚ Ğ½Ğ° ÑĞ°Ğ¼Ğ¾Ğ¼ Ğ´ĞµĞ»Ğµ  
@log_call  
def add(a: int, b: int) -> int:  ÷Ï:¶‰ËkºwµçkBHƒFFBÓF#B×BğƒFBïFFBÃBÔƒBûBÿB×FBÃFBãBàƒFBøƒFBïBûBËBÃFFGBğƒBóBûBÏFFƒBÓB×BÏFBÃBÓBãFBûBËBÃFF0°ƒB÷BøƒBÈƒB÷BûFBóBÃBïF3B÷F/FƒFFBïBûBËBãF?FƒBÓBûFFFBüƒBÿBøƒBëBïF;FFƒFFBãFBÃB×FFF<< Ä¤¸€€4(4(ŒŒ€ĞÜ¸ƒBŸBÃFFF/BÔƒBËBûBÿFBûFF,ƒBàƒFBûFBûF#BãBÔƒBûFBËB×FF,4(4+BŸFBøƒFBÃBëBûBÔAåÑ¡½¸ü€€4(4)AåÑ¡½¸ƒŠPƒBËF/FBûBëBûFFBûBËB÷B×BËF/BäƒBãB÷FB×FBÿFB×FBãFFB×BóF/BäƒF?BßF/BèƒFƒBÓBãB÷BÃBóBãFB×FBëBûBäƒFBãBÿBãBßBÃFBãB×Bä¸ƒB{BôƒBÿBûBÓBÓB×FBÛBãBËBÃB×FƒB{B{B|°ƒFFB÷BëFBãBûB÷BÃBïF3B÷F/BäƒFFBãBïF0°ƒBãBóB×B×FƒBÇBûBÏBÃFFF8ƒFFBÃB÷BÓBÃFFB÷FF8ƒBÇBãBÇBïBãBûFB×BëFƒBàƒFBÃFFBøƒBãFBÿBûBïF3BßFB×FFF<ƒBÓBïF<ƒBÃBËFBûBóBÃFBãBßBÃFBãBà°‰…­•¹°‘…Ñ„ÁÉ½•ÍÍ¥¹œ°ƒFB×FFBãFBûBËBÃB÷BãF<ƒBàƒFBëFBãBÿFBûBÈ¸€€4(4)AåÑ¡½¸ƒBëBûBóBÿBãBïBãFFB×BóF/BäƒBãBïBàƒBãB÷FB×FBÿFB×FBãFFB×BóF/Bäü€€4(4+B{BÇF/FB÷BøƒBÏBûBËBûFF?F°ƒFFBøAåÑ¡½¸ƒBãB÷FB×FBÿFB×FBãFFB×BóF/Bä°ƒB÷BøƒFBûFB÷B×BÔèƒBãFFBûBÓB÷F/BäƒBëBûBĞƒFB÷BÃFBÃBïBÀƒBëBûBóBÿBãBïBãFFB×FFF<ƒBÈƒBÇBÃBçFBëBûBĞ€¹ÁåŒ°ƒBÀƒBßBÃFB×BğƒBËF/BÿBûBïB÷F?B×FFF<ƒBËBãFFFBÃBïF3B÷BûBäƒBóBÃF#BãB÷BûBäAåÑ¡½¸¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔƒBÓBãB÷BÃBóBãFB×FBëBÃF<ƒFBãBÿBãBßBÃFBãF<ü4(4+B‹BãBüƒBÿFBãB÷BÃBÓBïB×BÛBãFƒBûBÇF+B×BëFF°ƒBÀƒB÷BÔƒBÿB×FB×BóB×B÷B÷BûBä¸ƒB{BÓB÷BÀƒBàƒFBÀƒBÛBÔƒBÿB×FB×BóB×B÷B÷BÃF<ƒBóBûBÛB×FƒFFF/BïBÃFF3FF<ƒB÷BÀƒBûBÇF+B×BëFF,ƒFBÃBßB÷F/FƒFBãBÿBûBÈ¸€€4(4)Ù…±Õ”€ô€ÄÀ€€4)Ù…±Õ”€ô€‰¡•±±¼ˆ€€4+BŸFBøƒFBÃBëBûBÔƒFFFBûBÏBÃF<ƒFBãBÿBãBßBÃFBãF<ü€€4(4)AåÑ¡½¸ƒB÷BÔƒBËF/BÿBûBïB÷F?B×FƒB÷B×F?BËB÷F/BÔƒBûBÿBÃFB÷F/BÔƒBÿFB×BûBÇFBÃBßBûBËBÃB÷BãF<ƒFBãBÿBûBÈ¸€€4(4)ÁÉ¥¹Ğ ˆÄˆ€¬€Ä¤€€4(4+BGFBÓB×FƒBûF#BãBÇBëBÀè€€4(4)QåÁ•ÉÉ½È€€4+BŸB×Bğ±¥ÍĞƒBûFBïBãFBÃB×FFF<ƒBûFÑÕÁ±”ü€€4(4)±¥ÍĞƒBãBßBóB×B÷F?B×BóF/Bä°ÑÕÁ±”ƒB÷B×BãBßBóB×B÷F?B×BóF/Bä¸ƒB‡BÿBãFBûBèƒBãFBÿBûBïF3BßFF;FƒBÓBïF<ƒBÓBãB÷BÃBóBãFB×FBëBãFƒB÷BÃBÇBûFBûBÈƒBÓBÃB÷B÷F/F°ƒBëBûFFB×BØƒŠPƒBÓBïF<ƒFBãBëFBãFBûBËBÃB÷B÷BûBäƒFFFFBëFFFF,¸€€4(4+BŸB×Bğ±¥ÍĞƒBûFBïBãFBÃB×FFF<ƒBûFÍ•Ğü€€4(4)±¥ÍĞƒFFBÃB÷BãFƒBÿBûFF?BÓBûBèƒBàƒBÓBûBÿFFBëBÃB×FƒBÓFBÇBïBãBëBÃFF,¸Í•ĞƒFFBÃB÷BãFƒFBûBïF3BëBøƒFB÷BãBëBÃBïF3B÷F/BÔƒF7BïB×BóB×B÷FF,ƒBàƒBÇF/FFFB×BÔƒBÓBïF<ƒBÿFBûBËB×FBëBàƒBËFBûBÛBÓB×B÷BãF<¸€€4(4+BŸB×Bğ‘¥ĞƒBûFBïBãFBÃB×FFF<ƒBûF±¥ÍĞü€€4(4)±¥ÍĞƒŠPƒBãB÷BÓB×BëFBãFBûBËBÃB÷B÷BÃF<ƒBÿBûFBïB×BÓBûBËBÃFB×BïF3B÷BûFFF0¸‘¥ĞƒŠPƒFFFFBëFFFBÀƒBëBïF;F·BßB÷BÃFB×B÷BãBÔ°ƒBÏBÓBÔƒBÓBûFFFBüƒBãBÓFGFƒBÿBøƒBëBïF;FF¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔÍ¡…±±½Ü½ÁäƒBà‘••À½Áäü4(4)M¡…±±½Ü½ÁäƒBëBûBÿBãFFB×FƒFBûBïF3BëBøƒBËB÷B×F#B÷BãBäƒBûBÇF+B×BëF°ƒBÀƒBËBïBûBÛB×B÷B÷F/BÔƒBûBÇF+B×BëFF,ƒBûFFBÃF;FFF<ƒBûBÇF'BãBóBà¸••À½ÁäƒFB×BëFFFBãBËB÷BøƒBëBûBÿBãFFB×FƒBËBïBûBÛB×B÷B÷F/BÔƒBûBÇF+B×BëFF,¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔƒBÏB×B÷B×FBÃFBûF ü4(4+BOB×B÷B×FBÃFBûF ƒŠPƒBûBÇF+B×BëF°ƒBëBûFBûFF/BäƒBïB×B÷BãBËBøƒBËF/BÓBÃFGFƒBßB÷BÃFB×B÷BãF<ƒBÿBøƒBûBÓB÷BûBóF¸ƒB{BôƒFBûBßBÓBÃFGFFF<ƒFFB÷BëFBãB×BäƒFå¥•±ƒBàƒF7BëBûB÷BûBóBãFƒBÿBÃBóF?FF0¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔƒBÓB×BëBûFBÃFBûF ü4(4+BSB×BëBûFBÃFBûF ƒŠPƒFFB÷BëFBãF<°ƒBëBûFBûFBÃF<ƒBûBÇBûFBÃFBãBËBÃB×FƒBÓFFBÏFF8ƒFFB÷BëFBãF8ƒBàƒFBÃFF#BãFF?B×FƒB×FDƒBÿBûBËB×BÓB×B÷BãBÔƒBÇB×BÜƒBãBßBóB×B÷B×B÷BãF<ƒBãFFBûBÓB÷BûBÏBøƒBëBûBÓBÀ¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔƒBëBûB÷FB×BëFFB÷F/BäƒBóB×B÷B×BÓBÛB×F ü4(4+B{BÇF+B×BëF°ƒBëBûFBûFF/BäƒFBÿFBÃBËBïF?B×FƒBËFBûBÓBûBğƒBàƒBËF/FBûBÓBûBğƒBãBÜƒBÇBïBûBëBÀİ¥Ñ ¸ƒB{BÇF/FB÷BøƒBãFBÿBûBïF3BßFB×FFF<ƒBÓBïF<ƒBÇB×BßBûBÿBÃFB÷BûBäƒFBÃBÇBûFF,ƒFƒFB×FFFFBÃBóBà¸€€4(4(ŒŒŒƒBŸFBøƒFBÃBëBûBÔ%0ü4(4)%0ƒŠPƒBÇBïBûBëBãFBûBËBëBÀƒBÈAåÑ¡½¸°ƒBãBÜ·BßBÀƒBëBûFBûFBûBäƒFBûBïF3BëBøƒBûBÓBãBôƒBÿBûFBûBèƒBûBÓB÷BûBËFB×BóB×B÷B÷BøƒBËF/BÿBûBïB÷F?B×FAåÑ¡½¸‰åÑ•½‘”¸ƒB{BôƒBûBÏFBÃB÷BãFBãBËBÃB×FATµ‰½Õ¹ƒBóB÷BûBÏBûBÿBûFBûFB÷BûFFF0°ƒB÷BøƒB÷BÔƒBóB×F#BÃB×FƒBãFBÿBûBïF3BßBûBËBÃFF0ƒBÿBûFBûBëBàƒBÓBïF<%<µ‰½Õ¹ƒBßBÃBÓBÃF¸€€4(4(ŒŒŒƒBŸFBøƒBïFFF#BÔèÑ¡É•…‘¥¹œ°µÕ±Ñ¥ÁÉ½•ÍÍ¥¹œƒBãBïBà…Íå¹¥¼ü4(4+BSBïF<%<µ‰½Õ¹ƒBßBÃBÓBÃFƒŠPÑ¡É•…‘¥¹œƒBãBïBà…Íå¹¥¼¸ƒBSBïF<ATµ‰½Õ¹ƒBßBÃBÓBÃFƒŠPµÕ±Ñ¥ÁÉ½•ÍÍ¥¹œ¸Íå¹¥¼ƒFBûFBûF#BøƒBÿBûBÓFBûBÓBãFƒBÓBïF<ƒBÇBûBïF3F#BûBÏBøƒBëBûBïBãFB×FFBËBÀƒFB×FB×BËF/FƒBûBÿB×FBÃFBãBä°ƒB×FBïBàƒBãFBÿBûBïF3BßFB×BóF/BÔƒBÇBãBÇBïBãBûFB×BëBàƒBÃFBãB÷FFBûB÷B÷F/BÔ¸€€4(4(ŒŒ€Ğà¸ƒBŸFBøƒBóBûBÏFFƒFBÿFBûFBãFF0ƒFEÕÑ½µ…Ñ¥½¸4(4+BkBÃBèƒBÇF,ƒFF,ƒBÿBûFFFBûBãBìA$·BëBïBãB×B÷Fü€€4(4+B{FBËB×Fè€€4(4+B¼ƒBÇF,ƒBËF/B÷B×FƒFBÃBÇBûFFƒF!QQ@ƒBÈƒBûFBÓB×BïF3B÷F/BäƒBëBïBãB×B÷Fè‰…Í•}ÕÉ°°Ñ¥µ•½ÕĞ°¡•…‘•ÉÌ°ƒBÃBËFBûFBãBßBÃFBãF<°ƒBóB×FBûBÓF,•Ğ½Á½ÍĞ½ÁÕĞ½‘•±•Ñ”°ƒBûBÇFBÃBÇBûFBëBÀƒBûF#BãBÇBûBèƒBàƒBïBûBÏBãFBûBËBÃB÷BãBÔ¸ƒBHƒFB×FFBÃFƒBûFFBÃBËBãBìƒBÇF,ƒFBûBïF3BëBøƒBÇBãBßB÷B×F·BÓB×BçFFBËBãF<ƒBàƒBÿFBûBËB×FBëBà¸€€4(4(ŒŒŒƒBBûFB×BóFƒBÿBïBûFBøƒBÿBãFBÃFF0É•ÅÕ•ÍÑÌƒBÿFF?BóBøƒBÈƒFB×FFBÃFü4(4+BBûFBûBóFƒFFBøƒBÿBûF?BËBïF?B×FFF<ƒBÓFBÇBïBãFBûBËBÃB÷BãBÔ°ƒFB×FFF,ƒFFBÃB÷BûBËF?FFF<ƒBóB×B÷B×BÔƒFBãFBÃB×BóF/BóBà°ƒFBïBûBÛB÷B×BÔƒBóB×B÷F?FF0ƒBÃBËFBûFBãBßBÃFBãF8°‰…Í•}ÕÉ°°ƒFBÃBçBóBÃFFF,ƒBàƒBûBÇFBÃBÇBûFBëFƒBûF#BãBÇBûBè¸ƒBoFFF#BÔƒBãBóB×FF0ƒFBïBûBäƒBëBïBãB×B÷FBÀ¸€€4(4(ŒŒŒƒBkBÃBèƒBÇF,ƒFF,ƒFB×FFBãFBûBËBÃBìƒFFB÷BëFBãF8ü4(4(ŒŒŒƒBFBãBóB×F è4(4)‘•˜¥Í}Ù…±¥‘}Á…ÍÍİ½É¡Á…ÍÍİ½ÉèÍÑÈ¤€´ø‰½½°è€€4(€€€€¸¸¸€€4(4+B‹B×FFBûBËF/BÔƒBÓBÃB÷B÷F/BÔè€€4(4+BBÃFBûBïF0'B{BÛBãBÓBÃB×Bğ€€4)A…ÍÍİ½ÉÄ%QÉÕ”€€4)Á…ÍÌ%…±Í”€€4)Á…ÍÍİ½ÉÄ%…±Í”€€4)AMM]=I%…±Í”€€4)A…ÍÍİ½É%…±Í”€€4(ÄÈÌĞÔØÜà%…±Í”€€4)A…ÍÌÄÈÌĞ%QÉÕ”€€4+BkBÃBèƒBÇF,ƒFF,ƒBûBÇFBÃBÇBÃFF/BËBÃBìƒB÷B×FFBÃBÇBãBïF3B÷F/BÔƒFB×FFF,ü€€4(4+B‡B÷BÃFBÃBïBÀƒBËF/F?FB÷BãBìƒBÇF,ƒBÿFBãFBãB÷FèƒBûBÛBãBÓBÃB÷BãF<°ƒBÓBÃB÷B÷F/BÔ°ƒBûBëFFBÛB×B÷BãBÔ°ƒFB×FB×BËF/BÔƒBÿFBûBÇBïB×BóF,°É…”½¹‘¥Ñ¥½¸¸ƒBwBÔƒFFBÃBìƒBÇF,ƒBÿFBûFFBøƒBÓBûBÇBÃBËBïF?FF0Í±••À¸ƒBoFFF#BÔƒBãFBÿBûBïF3BßBûBËBÃFF0ƒF?BËB÷F/BÔƒBûBÛBãBÓBÃB÷BãF<°ƒFB×FFBÃBàƒFBûBïF3BëBøƒB÷BÀƒBãB÷FFBÃFFFFBëFFFB÷F/BÔƒBûF#BãBÇBëBà°ƒBãBßBûBïF?FBãF8ƒBÓBÃB÷B÷F/FƒBàƒB÷BûFBóBÃBïF3B÷FF8ƒBÓBãBÃBÏB÷BûFFBãBëF¸€€4(4(ŒŒŒƒBkBÃBèƒBÇF,ƒFF,ƒFFBëBûFF?BìƒBÃBËFBûFB×FFF,ü4(4+BBÃFBÃBïBïB×BïF3B÷F/BäƒBßBÃBÿFFBè°ƒFBÃBßBÓB×BïB×B÷BãBÔƒFB×FFBûBÈƒBÿBøƒFFBûBËB÷F?Bğ°ƒFBóB×B÷F3F#B×B÷BãBÔU$·FB×FFBûBÈ°ƒBÿB×FB×BãFBÿBûBïF3BßBûBËBÃB÷BãBÔƒFBãBëFFFF °ƒBÿBûBÓBÏBûFBûBËBëBÀƒBÓBÃB÷B÷F/FƒFB×FB×BÜA$¿BGBP°ƒBóBÃFBëBãFBûBËBëBÀƒFB×FFBûBÈ°ƒBßBÃBÿFFBèƒFBûBïF3BëBøƒBßBÃFFBûB÷FFF/FƒBûBÇBïBÃFFB×Bä°ƒBÃB÷BÃBïBãBÜƒFBÃBóF/FƒBÓBûBïBÏBãFƒFB×FFBûBÈ¸€€4(4(ŒŒ€Ğä¸ƒB‹BãBÿBãFB÷F/BÔƒBûF#BãBÇBëBàƒB÷BÀƒFBûBÇB×FB×BÓBûBËBÃB÷BãBà4(4(Ä¸ƒBcFBÿBûBïF3BßBûBËBÃFF0µÕÑ…‰±”‘•™…Õ±Ğ…ÉÕµ•¹Ğ€€4(4(ŒŒŒƒBBïBûFBøè4(4)‘•˜™Õ¹Œ¡¥Ñ•µÌõmt¤è€€4(€€€€¸¸¸€€4(4(ŒŒŒƒB—BûFBûF#Bøè4(4)‘•˜™Õ¹Œ¡¥Ñ•µÌè±¥ÍÑm¥¹Ñtğ9½¹”€ô9½¹”¤€´ø9½¹”è€€4(€€€¥˜¥Ñ•µÌ¥Ì9½¹”è€€4(€€€€€€€¥Ñ•µÌ€ômt€€4(È¸ƒBFFBÃFF0¥ÌƒBà€ôô€€4(4(ŒŒŒƒBBïBûFBøè4(4)¥˜Ù…±Õ”€ôô9½¹”è€€4(€€€€¸¸¸€€4(4(ŒŒŒƒB—BûFBûF#Bøè4(4)¥˜Ù…±Õ”¥Ì9½¹”è€€4(€€€€¸¸¸€€4(Ì¸ƒBSFBóBÃFF0°ƒFFBø…ÁÁ•¹ƒBËBûBßBËFBÃF'BÃB×FƒFBÿBãFBûBè€€4(4(ŒŒŒƒBBïBûFBøè4(4)¥Ñ•µÌ€ôlÄ°€Ét€€4)¥Ñ•µÌ€ô¥Ñ•µÌ¹…ÁÁ•¹ Ì¤€€4(4+BBûFBïBÔƒF7FBûBÏBøè€€4(4)¥Ñ•µÌ¥Ì9½¹”€€4(Ğ¸ƒBcBßBóB×B÷F?FF0ƒFBÿBãFBûBèƒBËBøƒBËFB×BóF<ƒBûBÇFBûBÓBÀ€€4(4(ŒŒŒƒBBïBûFBøè4(4)¥Ñ•µÌ€ôlÄ°€È°€Ì°€Ñt€€4(4)™½È¥Ñ•´¥¸¥Ñ•µÌè€€4(€€€¥˜¥Ñ•´€”€È€ôô€Àè€€4(€€€€€€€¥Ñ•µÌ¹É•µ½Ù”¡¥Ñ•´¤€€4(4+BoFFF#BÔè€€4(4)¥Ñ•µÌ€ôm¥Ñ•´™½È¥Ñ•´¥¸¥Ñ•µÌ¥˜¥Ñ•´€”€È€„ô€Át€€4(Ô¸ƒBoBûBËBãFF0ƒBËFBÔƒBãFBëBïF;FB×B÷BãF<ƒBÇB×BÜƒBÿFBãFBãB÷F,€€4(4(ŒŒŒƒBBïBûFBøè4(4)ÑÉäè€€4(€€€€¸¸¸€€4)•á•ÁĞá•ÁÑ¥½¸è€€4(€€€Á…ÍÌ€€4(4+BoFFF#BÔè€€4(4)ÑÉäè€€4(€€€€¸¸¸€€4)•á•ÁĞY…±Õ•ÉÉ½È…Ì•ÉÉ½Èè€€4(€€€ÁÉ¥¹Ğ¡•ÉÉ½È¤€€4(4(ŒŒ€ÔÀ¸ƒBsBãB÷Bà·F#BÿBÃFBÏBÃBïBëBÀƒBÿBøƒFBãB÷FBÃBëFBãFF4(4(ŒƒBFBïBûBËBãBÔ€€4)¥˜Ù…±Õ”€ø€ÄÀè€€4(€€€ÁÉ¥¹Ğ ‰‰¥œˆ¤€€4)•±¥˜Ù…±Õ”€ôô€ÄÀè€€4(€€€ÁÉ¥¹Ğ ‰Ñ•¸ˆ¤€€4)•±Í”è€€4(€€€ÁÉ¥¹Ğ ‰Íµ…±°ˆ¤€€4(4(ŒƒB›BãBëBì™½È€€4)™½È¥Ñ•´¥¸¥Ñ•µÌè€€4(€€€ÁÉ¥¹Ğ¡¥Ñ•´¤€€4(4(ŒƒB›BãBëBìİ¡¥±”€€4)İ¡¥±”½¹‘¥Ñ¥½¸è€€4(€€€€¸¸¸€€4(4(ŒƒB“FB÷BëFBãF<€€4)‘•˜™Õ¹Œ¡„è¥¹Ğ°ˆè¥¹Ğ¤€´ø¥¹Ğè€€4(€€€É•ÑÕÉ¸„€¬ˆ€€4(4(ŒƒBkBïBÃFF€€4)±…ÍÌUÍ•Èè€€4(€€€‘•˜}}¥¹¥Ñ}|¡Í•±˜°¹…µ”èÍÑÈ¤€´ø9½¹”è€€4(€€€€€€€Í•±˜¹¹…µ”€ô¹…µ”€€4(4(ŒƒBcFBëBïF;FB×B÷BãF<€€4)ÑÉäè€€4(€€€€¸¸¸€€4)•á•ÁĞY…±Õ•ÉÉ½Èè€€4(€€€€¸¸¸€€4)™¥¹…±±äè€€4(€€€€¸¸¸€€4(4(ŒƒBkBûB÷FB×BëFFB÷F/BäƒBóB×B÷B×BÓBÛB×F €€4)İ¥Ñ ½Á•¸ ‰™¥±”¹ÑáĞˆ°€‰Èˆ°•¹½‘¥¹œô‰ÕÑ˜´àˆ¤…Ì™¥±”è€€4(€€€½¹Ñ•¹Ğ€ô™¥±”¹É•… ¤€€4(4(Œ1¥ÍĞ½µÁÉ•¡•¹Í¥½¸€€4)ÍÅÕ…É•Ì€ômà€¨à™½Èà¥¸É…¹” ÄÀ¥t€€4(4(Œ¥Ğ½µÁÉ•¡•¹Í¥½¸€€4)‘…Ñ„€ôíàèà€¨à™½Èà¥¸É…¹” ÄÀ¥ô€€4(4(Œ1…µ‰‘„€€4)¥Ñ•µÌ¹Í½ÉĞ¡­•äõ±…µ‰‘„¥Ñ•´è¥Ñ•µl‰¥‰t¤€€4(4(ŒŒ€ÔÄ¸ƒBŸFBøƒBÿBûBËFBûFBãFF0ƒBÿB×FB×BĞƒFBûBÇB×FB×BÓBûBËBÃB÷BãB×BğƒBÈƒBÿB×FBËFF8ƒBûFB×FB×BÓF04(4+B‡BÃBóBûBÔƒBËBÃBÛB÷BûBÔè€€4(4)±¥ÍĞ°‘¥Ğ°Í•Ğ°ÑÕÁ±”¸€€4)¥ÌÙÌ€ôô¸€€4+BcBßBóB×B÷F?B×BóF/BÔƒBàƒB÷B×BãBßBóB×B÷F?B×BóF/BÔƒFBãBÿF,¸€€4+B“FB÷BëFBãBà°€©…ÉÌ°€¨©­İ…ÉÌ¸€€4)5ÕÑ…‰±”‘•™…Õ±Ğ…ÉÕµ•¹Ğ¸€€4+BcFBëBïF;FB×B÷BãF<¸€€4+BkBûB÷FB×BëFFB÷F/BÔƒBóB×B÷B×BÓBÛB×FF,¸€€4+BSB×BëBûFBÃFBûFF,¸€€4+BOB×B÷B×FBÃFBûFF,¸€€4+B{B{B|èÍ•±˜°ƒB÷BÃFBïB×BÓBûBËBÃB÷BãBÔ°ÍÕÁ•È°ÍÑ…Ñ¥µ•Ñ¡½°±…ÍÍµ•Ñ¡½°ÁÉ½Á•ÉÑä¸€€4)QåÁ¥¹œ¸€€4)½Õ¹Ñ•È°‘•™…Õ±Ñ‘¥Ğ¸€€4)Ñ¡É•…‘¥¹œ°µÕ±Ñ¥ÁÉ½•ÍÍ¥¹œ°…Íå¹¥¼°%0¸€€4+BFBûFFF/BÔƒBßBÃBÓBÃFBàƒB÷BÀƒFFFBûBëBà°ƒFBÿBãFBëBàƒBàƒFBïBûBËBÃFBà¸€€4+B‡BïBûBÛB÷BûFFF0<¡¸¤ƒBà<¡»
È¤¸€€4(4(ŒŒ€ÔÈ¸ƒBkBÃBèƒBûFBËB×FBÃFF0°ƒB×FBïBàƒB÷BÔƒBßB÷BÃB×F#F0ƒBÏBïFBÇBûBëBø4(4(ŒŒŒƒB—BûFBûF#BÃF<ƒFBûFBóFBïBãFBûBËBëBÀè4(4+B¼ƒB÷BÔƒBÇFBÓFƒBÿFBãBÓFBóF/BËBÃFF0¸ƒBHƒFBÃBÇBûFBÔƒF<ƒFƒF7FBãBğƒFFBÃBïBëBãBËBÃBïFF<ƒB÷BÀƒBÇBÃBßBûBËBûBğƒFFBûBËB÷BÔ¸ƒBBûB÷BãBóBÃF8ƒBûBÇF'FF8ƒBãBÓB×F8èƒB÷BÃBÿFBãBóB×F °%0ƒBûBÏFBÃB÷BãFBãBËBÃB×FƒBËF/BÿBûBïB÷B×B÷BãBÔAåÑ¡½¸‰åÑ•½‘”ƒBÈƒB÷B×FBëBûBïF3BëBãFƒBÿBûFBûBëBÃF°ƒBÿBûF7FBûBóFƒBÓBïF<ATµ‰½Õ¹ƒBïFFF#BÔµÕ±Ñ¥ÁÉ½•ÍÍ¥¹œ°ƒBÀƒBÓBïF<%<µ‰½Õ¹ƒBóBûBÛB÷BøƒBãFBÿBûBïF3BßBûBËBÃFF0Ñ¡É•…‘¥¹œƒBãBïBà…Íå¹¥¼¸ƒBHƒBÓB×FBÃBïF?FƒFB×BÃBïBãBßBÃFBãBàƒBóBûBÏFƒBÿBûBÓFBóBûFFB×FF0ƒBÓBûBëFBóB×B÷FBÃFBãF8°ƒB÷BøƒBÿFBÃBëFBãFB×FBëBãBäƒFBóF/FBìƒBÿBûB÷BãBóBÃF8¸€€4(4+B·FBøƒBßBËFFBãFƒBïFFF#BÔ°ƒFB×BğƒBÿF/FBÃFF3FF<ƒFBËB×FB×B÷B÷BøƒFBëBÃBßBÃFF0ƒB×FFB÷BÓF¸€€4(4(ŒŒ€ÔÌ¸ƒB{FB×B÷F0ƒBëBûFBûFBëBÃF<ƒBËB×FFBãF<ƒBÓBïF<ƒBÿBûBËFBûFB×B÷BãF<ƒBßBÀ€ÔƒBóBãB÷FF4(4)AåÑ¡½¸ƒŠPƒBÓBãB÷BÃBóBãFB×FBëBàƒBàƒFFFBûBÏBøƒFBãBÿBãBßBãFBûBËBÃB÷B÷F/BäƒF?BßF/Bè¸ƒBB×FB×BóB×B÷B÷F/BÔƒFFBÃB÷F?FƒFFF/BïBëBàƒB÷BÀƒBûBÇF+B×BëFF,¸€€4(4)±¥ÍĞƒŠPƒBãBßBóB×B÷F?B×BóF/Bä°ƒFFBÃB÷BãFƒBÿBûFF?BÓBûBèƒBàƒBÓFBÇBïBãBëBÃFF,¸€€4)ÑÕÁ±”ƒŠPƒB÷B×BãBßBóB×B÷F?B×BóF/Bä¸€€4)Í•ĞƒŠPƒFB÷BãBëBÃBïF3B÷F/BÔƒF7BïB×BóB×B÷FF,°ƒBÇF/FFFF/Bä¥¸¸€€4)‘¥ĞƒŠPƒBëBïF;F·BßB÷BÃFB×B÷BãBÔ°ƒBÓBûFFFBüƒBÿBøƒBëBïF;FFƒBÈƒFFB×BÓB÷B×Bğ< Ä¤¸€€4(4(ôôƒFFBÃBËB÷BãBËBÃB×FƒBßB÷BÃFB×B÷BãF<¸€€4)¥ÌƒFFBÃBËB÷BãBËBÃB×FƒBãBÓB×B÷FBãFB÷BûFFF0ƒBûBÇF+B×BëFBûBÈ¸€€4)9½¹”ƒBÿFBûBËB×FF?B×BğƒFB×FB×BÜ¥Ì9½¹”¸€€4(4)5ÕÑ…‰±”‘•™…Õ±Ğ…ÉÕµ•¹ĞƒŠPƒFBÃFFBÃF<ƒBûF#BãBÇBëBÀè€€4(4)‘•˜™Õ¹Œ¡¥Ñ•µÌõmt¤è€€4(€€€€¸¸¸€€4(4+BFBÃBËBãBïF3B÷Bøè€€4(4)‘•˜™Õ¹Œ¡¥Ñ•µÌõ9½¹”¤è€€4(€€€¥˜¥Ñ•µÌ¥Ì9½¹”è€€4(€€€€€€€¥Ñ•µÌ€ômt€€4(4+BOB×B÷B×FBÃFBûF ƒBãFBÿBûBïF3BßFB×Få¥•±ƒBàƒBïB×B÷BãBËBøƒBûFBÓBÃFGFƒBßB÷BÃFB×B÷BãF<¸€€4(4+BSB×BëBûFBÃFBûF ƒBûBÇBûFBÃFBãBËBÃB×FƒFFB÷BëFBãF8¸€€4(4+BkBûB÷FB×BëFFB÷F/BäƒBóB×B÷B×BÓBÛB×F ƒFBÿFBÃBËBïF?B×FƒFB×FFFFBûBğƒFB×FB×BÜİ¥Ñ ¸€€4(4)%0ƒBóB×F#BÃB×FƒBÿBûFBûBëBÃBğƒBÿBÃFBÃBïBïB×BïF3B÷BøƒBËF/BÿBûBïB÷F?FF0AåÑ¡½¸‰åÑ•½‘”°ƒBÿBûF7FBûBóFè€€4(4)%<µ‰½Õ¹ƒŠPÑ¡É•…‘¥¹œ€¼…Íå¹¥¼ì€€4)ATµ‰½Õ¹ƒŠPµÕ±Ñ¥ÁÉ½•ÍÍ¥¹œ¸€€4(4+B{B{B|è€€4(4)Í•±˜ƒŠPƒFB×BëFF'BãBäƒBûBÇF+B×BëFì€€4)±…ÍÍµ•Ñ¡½ƒBÿBûBïFFBÃB×F±Ìì€€4)ÍÑ…Ñ¥µ•Ñ¡½ƒB÷BÔƒBÿBûBïFFBÃB×FƒB÷BàÍ•±˜°ƒB÷Bà±Ìì€€4)ÁÉ½Á•ÉÑäƒFBÿFBÃBËBïF?B×FƒBÓBûFFFBÿBûBğƒBèƒBÃFFBãBÇFFFì€€4)ÍÕÁ•È ¤ƒBËF/BßF/BËBÃB×FƒFBûBÓBãFB×BïF3FBëFF8ƒFB×BÃBïBãBßBÃFBãF8¸€€4(4(ŒŒ€ÔĞ¸ƒBsBãB÷Bà·B÷BÃBÇBûF ƒBßBÃBÓBÃF°ƒBëBûFBûFF/BÔƒFFBûBãFƒFBóB×FF0ƒBÿBãFBÃFF0ƒBÇF/FFFBø4(4)™É½´½±±•Ñ¥½¹Ì¥µÁ½ÉĞ½Õ¹Ñ•È°‘•™…Õ±Ñ‘¥Ğ€€4)™É½´ÑåÁ¥¹œ¥µÁ½ÉĞ¹ä€€4(4)‘•˜É•µ½Ù•}‘ÕÁ±¥…Ñ•Ì¡¥Ñ•µÌè±¥ÍÑm¥¹Ñt¤€´ø±¥ÍÑm¥¹Ñtè€€4(€€€Í••¸€ôÍ•Ğ ¤€€4(€€€É•ÍÕ±Ğ€ômt€€4(4(€€€™½È¥Ñ•´¥¸¥Ñ•µÌè€€4(€€€€€€€¥˜¥Ñ•´¹½Ğ¥¸Í••¸è€€4(€€€€€€€€€€€Í••¸¹…‘¡¥Ñ•´¤€€4(€€€€€€€€€€€É•ÍÕ±Ğ¹…ÁÁ•¹¡¥Ñ•´¤€€4(4(€€€É•ÑÕÉ¸É•ÍÕ±Ğ€€4(4)‘•˜™¥ÉÍÑ}Õ¹¥ÅÕ•}¡…È¡Ñ•áĞèÍÑÈ¤€´øÍÑÈğ9½¹”è€€4(€€€½Õ¹Ñ•È€ô½Õ¹Ñ•È¡Ñ•áĞ¤€€4(4(€€€™½È¡…È¥¸Ñ•áĞè€€4(€€€€€€€¥˜½Õ¹Ñ•Ém¡…Ét€ôô€Äè€€4(€€€€€€€€€€€É•ÑÕÉ¸¡…È€€4(4(€€€É•ÑÕÉ¸9½¹”€€4(4)‘•˜É½ÕÁ}‰å}ÕÍ•È¡½É‘•ÉÌè±¥ÍÑm‘¥ÑmÍÑÈ°¥¹Ñut¤€´ø‘¥Ñm¥¹Ğ°¥¹Ñtè€€4(€€€É•ÍÕ±Ğ€ô‘•™…Õ±Ñ‘¥Ğ¡¥¹Ğ¤€€4(4(€€€™½È½É‘•È¥¸½É‘•ÉÌè€€4(€€€€€€€É•ÍÕ±Ñm½É‘•Él‰ÕÍ•É}¥‰ut€¬ô½É‘•Él‰…µ½Õ¹Ğ‰t€€4(4(€€€É•ÑÕÉ¸‘¥Ğ¡É•ÍÕ±Ğ¤€€4(4)‘•˜‘¥Ñ}‘¥™˜ €€4(€€€‰•™½É”è‘¥ÑmÍÑÈ°¹åt°€€4(€€€…™Ñ•Èè‘¥ÑmÍÑÈ°¹åt°€€4(¤€´ø‘¥ÑmÍÑÈ°ÑÕÁ±•m¹ä°¹åutè€€4(€€€É•ÍÕ±Ğ€ôíô€€4(4(€€€™½È­•ä¥¸‰•™½É”¹­•åÌ ¤ğ…™Ñ•È¹­•åÌ ¤è€€4(€€€€€€€‰•™½É•}Ù…±Õ”€ô‰•™½É”¹•Ğ¡­•ä¤€€4(€€€€€€€…™Ñ•É}Ù…±Õ”€ô…™Ñ•È¹•Ğ¡­•ä¤€€4(4(€€€€€€€¥˜‰•™½É•}Ù…±Õ”€„ô…™Ñ•É}Ù…±Õ”è€€4(€€€€€€€€€€€É•ÍÕ±Ñm­•åt€ô€¡‰•™½É•}Ù…±Õ”°…™Ñ•É}Ù…±Õ”¤€€4(4(€€€É•ÑÕÉ¸É•ÍÕ±Ğ€€4(4)‘•˜¥Í}Ù…±¥‘}Á…ÍÍİ½É¡Á…ÍÍİ½ÉèÍÑÈ¤€´ø‰½½°è€€4(€€€É•ÑÕÉ¸€ €€4(€€€€€€€±•¸¡Á…ÍÍİ½É¤€øô€à€€4(€€€€€€€…¹…¹ä¡¡…È¹¥Í‘¥¥Ğ ¤™½È¡…È¥¸Á…ÍÍİ½É¤€€4(€€€€€€€…¹…¹ä¡¡…È¹¥ÍÕÁÁ•È ¤™½È¡…È¥¸Á…ÍÍİ½É¤€€4(€€€€¤€€4(4(ŒŒ€ÔÔ¸ƒB‡BÃBóBûBÔƒBËBÃBÛB÷BûBÔƒBÓBïF<M•¹¥½ÈEÕÑ½µ…Ñ¥½¸4(4+B‹B×BÇF<ƒBóBûBÏFFƒBûFB×B÷BãBËBÃFF0ƒB÷BÔƒFBûBïF3BëBøƒBÿBøƒBßB÷BÃB÷BãF8ƒFBãB÷FBÃBëFBãFBÀ°ƒBÀƒBÿBøƒFBûBóF°ƒBëBÃBèƒFF,ƒFBÃFFFBÛBÓBÃB×F#F0¸€€4(4(ŒŒŒƒB—BûFBûF#BøƒBÏBûBËBûFBãFF0ƒFBÃBèè4(4+B¼ƒFFBÃFBÃF;FF0ƒB÷BÔƒBÿBãFBÃFF0ƒBËFF8ƒBïBûBÏBãBëFƒBÿFF?BóBøƒBÈƒFB×FFBÔ¸ƒB{BÇF/FB÷BøƒFBÃBßBÓB×BïF?F8ƒBëBûBĞƒB÷BÀA$·BëBïBãB×B÷FF,°ƒFBãBëFFFFF,°ƒBóBûBÓB×BïBàƒBÓBÃB÷B÷F/F°‰Õ¥±‘•ÉÌ½™…Ñ½É¥•Ì°¡•±Á•ÉÌƒBà…ÍÍ•ÉÑ¥½¹Ì¸ƒBHƒFB×FFBÔƒBÓBûBïBÛB÷BÀƒBÇF/FF0ƒBËBãBÓB÷BÀƒBÇBãBßB÷B×F·FFFF0èƒBÿBûBÓBÏBûFBûBËBëBÀƒBÓBÃB÷B÷F/F°ƒBÓB×BçFFBËBãBÔ°ƒBÿFBûBËB×FBëBÀƒFB×BßFBïF3FBÃFBÀ¸ƒBwBãBßBëBûFFBûBËB÷B×BËF/BÔƒBÓB×FBÃBïBàƒBïFFF#BÔƒBÓB×FBÛBÃFF0ƒBûFBÓB×BïF3B÷Bø¸€€4(4(ŒŒŒƒBFBãBóB×F ƒFBûFBûF#B×BÏBøƒFB×FFBûBËBûBÏBøƒBÿBûBÓFBûBÓBÀè4(4)‘•˜Ñ•ÍÑ}ÕÍ•É}…¹}‰•}É•…Ñ•¡ÕÍ•É}±¥•¹Ğ°ÕÍ•É}™…Ñ½Éä¤€´ø9½¹”è€€4(€€€Á…å±½…€ôÕÍ•É}™…Ñ½Éä¹‰Õ¥± ¤€€4(4(€€€É•…Ñ•‘}ÕÍ•È€ôÕÍ•É}±¥•¹Ğ¹É•…Ñ•}ÕÍ•È¡Á…å±½…¤€€4(4(€€€…ÍÍ•ÉĞÉ•…Ñ•‘}ÕÍ•Él‰¥‰t¥Ì¹½Ğ9½¹”€€4(€€€…ÍÍ•ÉĞÉ•…Ñ•‘}ÕÍ•Él‰ÕÍ•É¹…µ”‰t€ôôÁ…å±½…‘l‰ÕÍ•É¹…µ”‰t€€4(4(ŒŒŒƒBBïBûFBûBäƒBÿBûBÓFBûBĞè4(4)‘•˜Ñ•ÍÑ}ÕÍ•É}…¹}‰•}É•…Ñ• ¤€´ø9½¹”è€€4(€€€É•ÍÁ½¹Í”€ôÉ•ÅÕ•ÍÑÌ¹Á½ÍĞ €€4(€€€€€€€€‰¡ÑÑÀè¼½¡½ÍĞ½…Á¤½ÕÍ•ÉÌˆ°€€4(€€€€€€€©Í½¸õì‰ÕÍ•É¹…µ”ˆè€‰…±•àˆ°€‰Á…ÍÍİ½Éˆè€‰A…ÍÍİ½ÉÄ‰ô°€€4(€€€€¤€€4(4(€€€…ÍÍ•ÉĞÉ•ÍÁ½¹Í”¹ÍÑ…ÑÕÍ}½‘”€ôô€ÈÀÄ€€4(4(ŒŒŒƒBBûFB×BóFƒBÿB×FBËF/BäƒBïFFF#BÔè4(4+FB×FFƒFBãFBÃB×FFF<ƒBëBÃBèƒFFB×B÷BÃFBãBäì€€4+BóB×B÷F3F#BÔƒBÓFBÇBïBãFBûBËBÃB÷BãF<ì€€4+BÿFBûF'BÔƒBóB×B÷F?FF0A$ì€€4+BÿFBûF'BÔƒBÿB×FB×BãFBÿBûBïF3BßBûBËBÃFF0ƒBÿBûBÓBÏBûFBûBËBëFƒBÓBÃB÷B÷F/Fì€€4+BÿFBûF'BÔƒBÿBûBÓBÓB×FBÛBãBËBÃFF0ƒBÇBûBïF3F#BûBäƒBÿFBûB×BëF¸€€4(4+B·FFƒF#BÿBÃFBÏBÃBïBëFƒBóBûBÛB÷BøƒBÓBÃBïF3F#BÔƒFBÃBßBËB×FB÷FFF0ƒBÈƒFBûFBóBÃFƒŠsBËBûBÿFBûFƒŠHƒBëBûFBûFBëBãBäƒBûFBËB×FƒBëBÃBèƒB÷BÀƒFBûBÇB×FB×BÓBûBËBÃB÷BãBàƒŠHƒBÿFBãBóB×F ƒBëBûBÓBÃŠt¸ƒB·FBøƒBÇFBÓB×FƒFBÓBûBÇB÷B×BÔƒBãBóB×B÷B÷BøƒBÓBïF<ƒFFB×B÷BãFBûBËBëBàƒBÿB×FB×BĞƒBãB÷FB×FBËF3F8¸€€4