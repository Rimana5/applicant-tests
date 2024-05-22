# Задания

## Python

Ниже даны 3 варианта заданий. Решить нужно какое-то одно.
Если успеваете в срок решить несколько — пожалуйста, такое будет плюсом 😀 

### 1 (качество кода, SOLID, принцип единой ответственности, Dependency Injection)

Выполните рефакторинг кода ниже. Реализуйте новый *payment_type* (скажем, *bank*). Убедитесь, что добавление новых *payment_type* осуществляется легко.<br>
*Подсказка: руководствуйтесь принципом единой ответственности. Также можно применить Dependency Injection.*

```python
class Order:
    def __init__(self):
        self.items = []
        self.quantities = []
        self.prices = []
        self.status = 'open'

    def add_item(self, name: str, quantity: int, price: float) -> None:
        self.items.append(name)
        self.quantities.append(quantity)
        self.prices.append(price)

    def total_price(self) -> float:
        total = 0
        for i in range(len(self.prices)):
            total += self.quantities[i] * self.prices[i]
        return total

    def pay(self, payment_processor, security_code: str) -> None:
        payment_processor.pay(security_code)
        self.status = 'paid'

class DebitPaymentProcessor:
    def pay(self, security_code: str) -> None:
        print('Какая-то логика реализации debit...')
        print(f'Верифицируем код: {security_code}')

class CreditPaymentProcessor:
    def pay(self, security_code: str) -> None:
        print('Какая-то логика реализации credit...')
        print(f'Верифицируем код: {security_code}')

class BankPaymentProcessor:
    def pay(self, security_code: str) -> None:
        print('Какая-то логика реализации bank...')
        print(f'Верифицируем код: {security_code}')

def main() -> None:
    order = Order()
    order.add_item('Keyboard', 1, 50)
    order.add_item('SSD', 1, 150)
    order.add_item('USB cable', 2, 5)
    print(order.total_price())

    payment_method = DebitPaymentProcessor()
    order.pay(payment_method, '0372846')

if __name__ == "__main__":
    main()

```

### 2 (качество кода, Dependency Injection, Dependency Inversion)

Код ниже работает без ошибок, но, возможно, требует определенного рефакторинга.
Внесите изменения в код так, чтобы добавлять новые **LampSwitcher'ы** было проще.

```python
from typing import Protocol

class LampSwitcher(Protocol):
    def turn_on(self) -> None:
        ...
    
    def turn_off(self) -> None:
        ...
    
    @property
    def on_state(self) -> bool:
        ...

class GlowLampSwitcher:
    def __init__(self) -> None:
        self._on_state = False

    def turn_on(self) -> None:
        print('Лампа накаливания включена...')
        print('Логика реализации включения лампы накаливания...')
        self._on_state = True

    def turn_off(self) -> None:
        print('Лампа накаливания выключена...')
        print('Логика реализации выключения лампы накаливания...')
        self._on_state = False

    @property
    def on_state(self) -> bool:
        return self._on_state

class HalogenLampSwitcher:
    def __init__(self) -> None:
        self._on_state = False

    def turn_on(self) -> None:
        print('Галогенная лампа включена...')
        print('Логика реализации включения галогена...')
        self._on_state = True

    def turn_off(self) -> None:
        print('Галогенная лампа выключена...')
        print('Логика реализации выключения галогена...')
        self._on_state = False

    @property
    def on_state(self) -> bool:
        return self._on_state

class AnotherLampSwitcher:
    def __init__(self) -> None:
        self._on_state = False

    def turn_on(self) -> None:
        print('Ещё лампа включена...')
        print('Специфическая логика реализации включения лампы...')
        self._on_state = True

    def turn_off(self) -> None:
        print('Ещё лампа выключена...')
        print('Специфическая логика реализации выключения лампы...')
        self._on_state = False

    @property
    def on_state(self) -> bool:
        return self._on_state

class ElectricLightSwitchManager:
    def __init__(self, switcher: LampSwitcher) -> None:
        self.switcher = switcher

    def press(self) -> None:
        if self.switcher.on_state:
            self.switcher.turn_off()
        else:
            self.switcher.turn_on()

def main() -> None:
    switch = ElectricLightSwitchManager(GlowLampSwitcher())
    switch.press()
    switch.press()
    
    switch = ElectricLightSwitchManager(HalogenLampSwitcher())
    switch.press()
    switch.press()
    
    switch = ElectricLightSwitchManager(AnotherLampSwitcher())
    switch.press()
    switch.press()

if __name__ == '__main__':
    main()

```

### 3 (парсинг, агрегация данных)

Необходимо:
1. Получить данные по комментариям и постам с ресурса http://jsonplaceholder.typicode.com/
    * http://jsonplaceholder.typicode.com/posts
    * http://jsonplaceholder.typicode.com/comments

2. Посчитать среднее количество комментариев к посту каждого
   пользователя, результатом должен быть словарь формата:
    * user_id
    * average_comments_per_post
   
3. Результат вывести в stdout (например `print`).


```python
#В данном коде не уверен, не успел нормально протестировать, но должен работать
import requests

def fetch_data(url):
    response = requests.get(url)
    response.raise_for_status()
    return response.json()

def calculate_average_comments(posts, comments):
    # Создаем словарь для хранения количества постов и комментариев для каждого пользователя
    user_post_comments = {}
    
    for post in posts:
        user_id = post['userId']
        if user_id not in user_post_comments:
            user_post_comments[user_id] = {'post_count': 0, 'comment_count': 0}
        user_post_comments[user_id]['post_count'] += 1
    
    for comment in comments:
        post_id = comment['postId']
        post = next((p for p in posts if p['id'] == post_id), None)
        if post:
            user_id = post['userId']
            user_post_comments[user_id]['comment_count'] += 1
    
    # Создаем словарь для хранения среднего количества комментариев на пост для каждого пользователя
    user_average_comments = {}
    
    for user_id, counts in user_post_comments.items():
        average_comments = counts['comment_count'] / counts['post_count']
        user_average_comments[user_id] = average_comments
    
    return user_average_comments

def main():
    posts_url = 'http://jsonplaceholder.typicode.com/posts'
    comments_url = 'http://jsonplaceholder.typicode.com/comments'
    
    posts = fetch_data(posts_url)
    comments = fetch_data(comments_url)
    
    average_comments = calculate_average_comments(posts, comments)
    
    print(average_comments)

if __name__ == '__main__':
    main()
```
