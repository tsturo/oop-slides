# Live Coding: Coffee Shop Order System

## Goal
Build a coffee shop order system demonstrating **4 design patterns** — Singleton, Builder, Strategy, and Decorator.

---

## Part A: Singleton — DatabaseConnection

## Step 1: Create a singleton database connection

```python
class DatabaseConnection:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.connection = "PostgreSQL@localhost:5432"
        return cls._instance

    def query(self, sql):
        print(f"[{self.connection}] Executing: {sql}")
```

```python
db1 = DatabaseConnection()
db2 = DatabaseConnection()

db1.query("SELECT * FROM orders")
db2.query("INSERT INTO orders VALUES ('latte')")
```

### Output:
```
[PostgreSQL@localhost:5432] Executing: SELECT * FROM orders
[PostgreSQL@localhost:5432] Executing: INSERT INTO orders VALUES ('latte')
```

## Step 2: Prove it's the same object

```python
print(f"db1 id: {id(db1)}")
print(f"db2 id: {id(db2)}")
print(f"Same object? {db1 is db2}")
```

### Output:
```
db1 id: 4391227280
db2 id: 4391227280
Same object? True
```

> **Key point:** No matter how many times you call `DatabaseConnection()`, you always get the same instance.
> This prevents multiple database connections from being opened unnecessarily.

---

## Part B: Builder — CoffeeOrder

## Step 3: Create the product class

```python
class CoffeeOrder:
    def __init__(self):
        self.size = "medium"
        self.milk = None
        self.sugar = 0
        self.extras = []
        self.name = "Anonymous"

    def __str__(self):
        parts = [f"{self.size} coffee for {self.name}"]
        if self.milk:
            parts.append(f"milk: {self.milk}")
        if self.sugar:
            parts.append(f"sugar: {self.sugar}")
        if self.extras:
            parts.append(f"extras: {', '.join(self.extras)}")
        return " | ".join(parts)
```

## Step 4: Create the builder with method chaining

```python
class CoffeeOrderBuilder:
    def __init__(self):
        self.order = CoffeeOrder()

    def set_size(self, size):
        self.order.size = size
        return self

    def set_milk(self, milk):
        self.order.milk = milk
        return self

    def add_sugar(self, amount=1):
        self.order.sugar += amount
        return self

    def add_extra(self, extra):
        self.order.extras.append(extra)
        return self

    def set_name(self, name):
        self.order.name = name
        return self

    def build(self):
        return self.order
```

## Step 5: Build different orders using method chaining

```python
latte = (CoffeeOrderBuilder()
    .set_size("large")
    .set_milk("oat")
    .add_sugar(2)
    .set_name("Alice")
    .build())

espresso = (CoffeeOrderBuilder()
    .set_size("small")
    .set_name("Bob")
    .build())

custom = (CoffeeOrderBuilder()
    .set_size("large")
    .set_milk("almond")
    .add_sugar(1)
    .add_extra("whipped cream")
    .add_extra("caramel drizzle")
    .set_name("Charlie")
    .build())

print(latte)
print(espresso)
print(custom)
```

### Output:
```
large coffee for Alice | milk: oat | sugar: 2
small coffee for Bob
large coffee for Charlie | milk: almond | sugar: 1 | extras: whipped cream, caramel drizzle
```

> **Key point:** The builder lets you construct complex objects step-by-step.
> Each method returns `self`, enabling readable method chaining. No need for a constructor with 10 parameters.

---

## Part C: Strategy — PaymentStrategy

## Step 6: Create the strategy abstraction

```python
from abc import ABC, abstractmethod


class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount):
        pass
```

## Step 7: Create concrete payment strategies

```python
class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number):
        self.card_number = card_number

    def pay(self, amount):
        masked = self.card_number[-4:]
        print(f"Paid ${amount:.2f} with credit card ending in {masked}")


class CashPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"Paid ${amount:.2f} in cash")


class MobilePayment(PaymentStrategy):
    def __init__(self, phone_number):
        self.phone_number = phone_number

    def pay(self, amount):
        print(f"Paid ${amount:.2f} via mobile ({self.phone_number})")
```

## Step 8: Use strategies in a coffee shop

```python
class CoffeeShop:
    def __init__(self, payment_strategy: PaymentStrategy):
        self.payment_strategy = payment_strategy

    def process_order(self, order, price):
        print(f"Order: {order}")
        self.payment_strategy.pay(price)
        print()
```

```python
shop_card = CoffeeShop(CreditCardPayment("4111222233334444"))
shop_card.process_order(latte, 5.50)

shop_cash = CoffeeShop(CashPayment())
shop_cash.process_order(espresso, 3.00)

shop_mobile = CoffeeShop(MobilePayment("+370 612 34567"))
shop_mobile.process_order(custom, 7.25)
```

### Output:
```
Order: large coffee for Alice | milk: oat | sugar: 2
Paid $5.50 with credit card ending in 4444

Order: small coffee for Bob
Paid $3.00 in cash

Order: large coffee for Charlie | milk: almond | sugar: 1 | extras: whipped cream, caramel drizzle
Paid $7.25 via mobile (+370 612 34567)
```

> **Key point:** The `CoffeeShop` doesn't know or care how payment works — it delegates to whatever strategy it receives.
> Adding Apple Pay? Just create a new strategy class. `CoffeeShop` never changes.

---

## Part D: Decorator — CoffeeDecorator

## Step 9: Create the base coffee class

```python
from abc import ABC, abstractmethod


class Coffee(ABC):
    @abstractmethod
    def cost(self):
        pass

    @abstractmethod
    def description(self):
        pass


class SimpleCoffee(Coffee):
    def cost(self):
        return 2.00

    def description(self):
        return "Simple coffee"
```

## Step 10: Create decorators that wrap coffee objects

```python
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee

    def cost(self):
        return self._coffee.cost()

    def description(self):
        return self._coffee.description()


class MilkDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.50

    def description(self):
        return self._coffee.description() + ", milk"


class SugarDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.25

    def description(self):
        return self._coffee.description() + ", sugar"


class WhipCreamDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.75

    def description(self):
        return self._coffee.description() + ", whip cream"
```

## Step 11: Stack decorators to build up a complex coffee

```python
coffee = SimpleCoffee()
print(f"{coffee.description()} = ${coffee.cost():.2f}")

coffee = MilkDecorator(coffee)
print(f"{coffee.description()} = ${coffee.cost():.2f}")

coffee = SugarDecorator(coffee)
print(f"{coffee.description()} = ${coffee.cost():.2f}")

coffee = WhipCreamDecorator(coffee)
print(f"{coffee.description()} = ${coffee.cost():.2f}")
```

### Output:
```
Simple coffee = $2.00
Simple coffee, milk = $2.50
Simple coffee, milk, sugar = $2.75
Simple coffee, milk, sugar, whip cream = $3.50
```

> **Key point:** Each decorator wraps the previous object, adding behavior without modifying the original class.
> You can combine decorators in any order and any quantity — try double milk or sugar + sugar + whip cream.

---

## Comparison

| Pattern | Problem | Solution |
|---|---|---|
| **Singleton** | Multiple instances waste resources or cause conflicts | Ensure only one instance exists, reuse it everywhere |
| **Builder** | Complex constructors with many optional parameters | Step-by-step construction with method chaining |
| **Strategy** | Hardcoded algorithm makes swapping behavior difficult | Encapsulate algorithms behind an interface, swap at runtime |
| **Decorator** | Subclass explosion when combining optional behaviors | Wrap objects dynamically to add responsibilities |

## Discussion Points
- When is Singleton a problem? (Global mutable state, hard to test, hides dependencies — some call it an anti-pattern)
- How does Builder differ from just using keyword arguments in Python? (Builder enforces step-by-step construction, can validate intermediate state, and reads clearly when construction is complex)
- How does Strategy relate to DIP from SOLID? (Strategy is DIP in action — the high-level class depends on an abstraction, concrete strategies are injected)
- Could you solve the Decorator problem with inheritance instead? (You could, but you'd need a class for every combination — `MilkSugarCoffee`, `MilkWhipCoffee`, etc. That's a combinatorial explosion)
- Where have you seen these patterns in real frameworks or libraries? (Singleton: logging module. Builder: ORM query builders. Strategy: sorting key functions. Decorator: Python's `@decorator` syntax, middleware)
