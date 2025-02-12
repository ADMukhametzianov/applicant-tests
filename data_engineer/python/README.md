# Задания

## Python

Ниже даны 3 варианта заданий. Решить нужно какое-то одно.
Если успеваете в срок решить несколько — пожалуйста, такое будет плюсом :)

### 1 (парсинг, агрегация данных)

Необходимо:
1. Получить данные по комментариям и постам с ресурса http://jsonplaceholder.typicode.com/
    * http://jsonplaceholder.typicode.com/posts
    * http://jsonplaceholder.typicode.com/comments

2. Посчитать среднее количество комментариев к посту каждого
   пользователя, результатом должен быть словарь формата:
    * user_id
    * average_comments_per_post
   
3. Результат вывести в stdout (например `print`).

**Ответ:**
```python
import requests
import json
# Забираем посты и комментарии
posts_url = 'http://jsonplaceholder.typicode.com/posts'
response = requests.get(posts_url)
posts = json.loads(response.text)
comments_url = 'http://jsonplaceholder.typicode.com/comments'
response = requests.get(comments_url)
comments = json.loads(response.text)

user_stats = {}
# Проходимся по постам и считаем количество комментариев для каждого пользователя
for post in posts:
    user_id = post['userId']
    if user_id not in user_stats:
        user_stats[user_id] = {'total_posts': 0, 'total_comments': 0}
    user_stats[user_id]['total_posts'] += 1
    post_comments = [comment for comment in comments if comment['postId'] == post['id']]
    user_stats[user_id]['total_comments'] += len(post_comments)

#Считаем среднее количество комментариев на пост для каждого пользователя
average_comments_per_post = {}
for user_id, stats in user_stats.items():
    if stats['total_posts'] != 0:
        average_comments_per_post[user_id] = int(stats['total_comments'] / stats['total_posts'])
    else:
        average_comments_per_post[user_id] = 0

print(average_comments_per_post)
```

### 2 (качество кода, Dependency Injection, Dependency Inversion)

Код ниже работает без ошибок, но, возможно, требует определенного рефакторинга.
Внесите изменения в код так, чтобы добавлять новые **LampSwitcher'ы** было проще.

```python
from typing import Union


class GlowLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def on(self) -> None:
        print('Лампа накаливания включена...')
        self.on_state = True

    def turn_off(self) -> None:
        print('Лампа накаливания выключена...')
        self.on_state = False


class HalogenLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def turn_on(self) -> None:
        print('Галогенная лампа включена...')
        self.on_state = True

    def off(self) -> None:
        print('Галогенная лампа выключена...')
        self.on_state = False


class AnotherLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def lamp_on(self) -> None:
        print('Ещё лампа включена...')
        self.on_state = True

    def lamp_off(self) -> None:
        print('Ещё лампа выключена...')
        self.on_state = False


class ElectricLightSwitchManager:

    def __init__(
        self, switcher: Union[GlowLampSwitcher, HalogenLampSwitcher, AnotherLampSwitcher],
    ) -> None:
        self.switcher = switcher

    def press(self) -> None:
        if isinstance(self.switcher, GlowLampSwitcher):
            if self.switcher.on_state:
                self.switcher.turn_off()
            else:
                self.switcher.on()
        elif isinstance(self.switcher, HalogenLampSwitcher):
            if self.switcher.on_state:
                self.switcher.off()
            else:
                self.switcher.turn_on()
        elif isinstance(self.switcher, AnotherLampSwitcher):
            if self.switcher.on_state:
                self.switcher.lamp_off()
            else:
                self.switcher.lamp_on()


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

### 3 (качество кода, SOLID, принцип единой ответственности, Dependency Injection)

Выполните рефакторинг кода ниже.<br>
*Подсказка: руководствуйтесь принципом единой ответственности. Также можно применить Dependency Injection.*

```python
class Order:

   def __init__(self):
      self.items = []
      self.quantities = []
      self.prices = []
      self.status = 'open'

   def add_item(self, name, quantity, price):
      self.items.append(name)
      self.quantities.append(quantity)
      self.prices.append(price)

   def total_price(self):
      total = 0
      for i in range(len(self.prices)):
         total += self.quantities[i] * self.prices[i]
      return total

   def pay(self, payment_type, security_code):
      if payment_type == 'debit':
         print('Какая-то логика реализации debit...')
         print(f'Верифицируем код: {security_code}')
         self.status = 'paid'
      elif payment_type == 'credit':
         print('Какая-то логика реализации credit...')
         print(f'Верифицируем код: {security_code}')
         self.status = 'paid'
      else:
         raise Exception(f'Неизвестный тип платежа: {payment_type}')


def main() -> None:
   order = Order()
   order.add_item('Keyboard', 1, 50)
   order.add_item('SSD', 1, 150)
   order.add_item('USB cable', 2, 5)
   print(order.total_price())
   order.pay('debit', '0372846')


if __name__ == "__main__":
   main()
```
