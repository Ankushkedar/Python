# Object-Oriented Programming (OOP) in Python — Concepts & Examples

Notes on the four pillars of OOP plus supporting concepts, each with a plain
explanation and a small runnable Python example.

---

## What is OOP?

OOP is a way of organizing code around **objects** — bundles of data (attributes)
and behavior (methods) — instead of loose functions and variables. It helps model
real-world things and keeps related code together.

The four pillars:
1. Encapsulation
2. Abstraction
3. Inheritance
4. Polymorphism

---

## Building blocks: Class, Object, Attribute, Method

- **Class** = blueprint/template.
- **Object (instance)** = a real thing built from the class.
- **Attribute** = data stored on an object.
- **Method** = a function that belongs to an object.

```python
class bike: # class
  def __init__(self,model,speed,milage): #special method
      self.model=model #attributes
      self.speed=speed #attributes
      self.milage=milage #attributes

  def explain(self): # Regular method
    return f"{self.model} having speed {self.speed} & milage {self.milage}"

my_bike=bike("Unicorn", 80, 60) # object or instance of class
print(my_bike.model)
print(my_bike.explain())
```

---

## 1. Encapsulation

**Idea:** Encapsulation means bundling data (variables) and methods (functions) into a single unit (class) and controlling access to that data.

- In Python, a leading underscore (`_name`) signals "internal, don't touch."
- A double underscore (`__name`) triggers name-mangling for stronger hiding.
- Access is typically controlled through methods (getters/setters) or properties.

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance          # "private" — hidden from outside

    def deposit(self, amount):            # controlled way to change data
        if amount > 0:
            self.__balance += amount

    def get_balance(self):                # controlled way to read data
        return self.__balance

acc = BankAccount(100)
acc.deposit(50)
print(acc.get_balance())     # -> 150
# print(acc.__balance)       # ERROR: cannot access the private attribute directly
```

**Why it matters:** callers cannot put the object into a bad state (e.g. negative
deposits), because all changes go through validated methods.

---

## 2. Abstraction

**Idea:** Show only the essential features and hide the complex implementation. The
user of a class knows *what* it does, not *how*.

- Often done with **abstract base classes** (`abc` module) that define a required
  interface without providing the full implementation.

```python
from abc import ABC, abstractmethod

class Shape(ABC):                 # abstract base class
    @abstractmethod
    def area(self):               # must be implemented by subclasses
        ...

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    def area(self):
        return 3.14159 * self.radius ** 2

class Square(Shape):
    def __init__(self, side):
        self.side = side
    def area(self):
        return self.side ** 2

# shape = Shape()   # ERROR: cannot instantiate an abstract class
for s in [Circle(2), Square(3)]:
    print(s.area())               # caller just calls area(), ignores the math inside
```

**Why it matters:** you can work with any `Shape` through the same simple interface
(`area()`) without caring about each shape's internal formula.

---

## 3. Inheritance

**Idea:** A class can reuse and extend another class. The child (subclass) gets the
parent's (superclass's) attributes and methods, and can add or override its own.

- `class Child(Parent):` declares inheritance.
- `super().__init__(...)` runs the parent's constructor.

```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        return "Some sound"

class Dog(Animal):                       # Dog inherits from Animal
    def __init__(self, name, breed):
        super().__init__(name)           # reuse Animal's setup
        self.breed = breed
    def speak(self):                     # override the parent's method
        return "Woof"

d = Dog("Rex", "Labrador")
print(d.name)        # -> "Rex"   (inherited attribute)
print(d.speak())     # -> "Woof"  (overridden method)
```

**Why it matters:** shared behavior lives in one place (the parent), avoiding
duplicated code across related classes.

---

## 4. Polymorphism

**Idea:** "Many forms." The same method name behaves differently depending on the
object. You can treat different objects uniformly as long as they share a method.

```python
class Cat:
    def speak(self):
        return "Meow"

class Dog:
    def speak(self):
        return "Woof"

class Cow:
    def speak(self):
        return "Moo"

for animal in [Cat(), Dog(), Cow()]:
    print(animal.speak())    # same call, different result per object
# -> Meow / Woof / Moo
```

**Why it matters:** code that loops over "things that can speak" does not need to know
or check each exact type — it just calls `speak()`.

### Related: method overriding vs overloading

- **Overriding** = a subclass redefines a parent method (shown in Inheritance above).
- **Overloading** (same method, different argument counts) is not built into Python the
  way it is in Java/C++. Python uses default arguments or `*args` instead:

```python
class Greeter:
    def hello(self, name=None):
        if name is None:
            return "Hello!"
        return f"Hello, {name}!"

g = Greeter()
print(g.hello())          # -> "Hello!"
print(g.hello("Alice"))   # -> "Hello, Alice!"
```

---

## Supporting concept: Composition

**Idea:** Build complex objects by **combining** other objects, instead of inheriting.
Often summarized as "has-a" (composition) vs "is-a" (inheritance).

```python
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()        # a Car HAS an Engine
    def start(self):
        return self.engine.start()    # delegate to the contained object

print(Car().start())    # -> "Engine started"
```

**When to use which:**
- Inheritance ("is-a"): a `Dog` *is an* `Animal`.
- Composition ("has-a"): a `Car` *has an* `Engine`.
- Prefer composition when the relationship is "has/uses," not "is a kind of."

---

## Supporting concept: `__init__`, `self`, and dunder methods

- `__init__` — the constructor; runs automatically when an object is created.
- `self` — the current object; supplied automatically, used to access its own data.
- Other common **dunder** ("double underscore") methods customize built-in behavior:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __str__(self):                  # controls print() / str()
        return f"Point({self.x}, {self.y})"
    def __eq__(self, other):            # controls ==
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
print(p1)            # -> "Point(1, 2)"   (uses __str__)
print(p1 == p2)      # -> True            (uses __eq__)
```

---

## Quick reference

| Concept | One-line meaning | Keyword / example |
|---------|------------------|-------------------|
| Class | Blueprint for objects. | `class Car:` |
| Object | Instance built from a class. | `c = Car()` |
| Attribute | Data on an object. | `self.brand` |
| Method | Behavior on an object. | `def describe(self):` |
| Encapsulation | Hide data, expose safe methods. | `self.__balance` + getters |
| Abstraction | Show what, hide how. | `ABC`, `@abstractmethod` |
| Inheritance | Reuse/extend another class. | `class Dog(Animal):` |
| Polymorphism | Same call, different behavior. | `animal.speak()` |
| Composition | Build objects from other objects. | `self.engine = Engine()` |
| `__init__` | Constructor, runs on creation. | `def __init__(self):` |
| `self` | The current object. | `self.x` |
| `super()` | Call the parent class. | `super().__init__()` |
| Dunder methods | Customize built-in behavior. | `__str__`, `__eq__` |
