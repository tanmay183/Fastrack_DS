# Topic: OOPs in Python (Object-Oriented Programming)

**All concepts + 15 standard problems from easy to hard, every line of code explained in comments, with output**

> **Goal of this note:** after this, any OOP question (design a class, use inheritance, explain polymorphism, build a small system like a library or a parking lot) should feel familiar. You will know the **concept**, the **Python syntax**, and the **interview answer**.

How to use this note: read Sections 1 to 12 first (concepts, about 60 minutes). Type every example yourself and predict the output before reading it. Then solve the 15 problems in Section 13 in order, trying each for 10 minutes **before** reading the solution.

---

## Table of Contents

1. [What is OOP and why do we use it?](#1-what-is-oop-and-why-do-we-use-it)
2. [Classes and objects](#2-classes-and-objects)
3. [Encapsulation](#3-encapsulation)
4. [Abstraction](#4-abstraction)
5. [Inheritance](#5-inheritance)
6. [Polymorphism](#6-polymorphism)
7. [Class methods and static methods](#7-class-methods-and-static-methods)
8. [Magic (dunder) methods](#8-magic-dunder-methods)
9. [Composition vs inheritance](#9-composition-vs-inheritance)
10. [Dataclasses and enums](#10-dataclasses-and-enums)
11. [Design principles and patterns (SOLID in simple words)](#11-design-principles-and-patterns)
12. [Common interview questions and mistakes](#12-common-interview-questions-and-mistakes)
13. [The 15 problems](#13-the-15-problems)
14. [Cheat sheet and extra practice](#14-cheat-sheet-and-extra-practice)

---

## 1. What is OOP and why do we use it?

### 1.1 The idea in plain words

In normal (procedural) programming you write **functions** and keep **data** separately. In **Object-Oriented Programming (OOP)** you bundle **data and the functions that work on that data** together into one unit called an **object**.

**Analogy:** A **class** is the **blueprint of a house**. An **object** is an **actual house** built from that blueprint. One blueprint can build many houses. Each house has its own colour and owner (its **data**), and all houses can do the same things, like open the door (their **methods**).

| Word | Meaning | Example |
|---|---|---|
| **Class** | blueprint / template | `Car` |
| **Object (instance)** | one real thing made from the class | `my_car = Car()` |
| **Attribute** | data stored in the object | `color`, `speed` |
| **Method** | function that belongs to the class | `start()`, `stop()` |

### 1.2 The 4 pillars of OOP (memorize these)

| Pillar | One-line meaning | Python tool |
|---|---|---|
| **Encapsulation** | keep data and methods together, and protect data from careless access | `_name`, `__name`, `@property` |
| **Abstraction** | show only **what** an object does, hide **how** | `ABC`, `@abstractmethod` |
| **Inheritance** | a new class reuses and extends an existing class | `class Child(Parent)` |
| **Polymorphism** | the same method name behaves differently for different objects | method overriding, duck typing, operator overloading |

### 1.3 Why OOP is useful

- **Reuse:** write once in a parent class, use in many child classes.
- **Organization:** big programs become small, named pieces.
- **Safety:** data can be protected and validated inside the class.
- **Modeling:** real-world things (account, student, vehicle) map naturally to classes.

---

## 2. Classes and objects

### 2.1 Your first class

```python
class Dog:                                      # define a class named Dog (the blueprint)
    species = "Canis familiaris"                # class attribute: shared by ALL dogs

    def __init__(self, name, age):              # constructor: runs automatically when a Dog is created
        self.name = name                        # instance attribute: belongs to this one dog
        self.age = age                          # instance attribute: belongs to this one dog

    def bark(self):                             # instance method: first parameter is always self
        return f"{self.name} says Woof!"        # self.name reaches THIS dog's name

    def birthday(self):                         # another method
        self.age += 1                           # change this dog's own data

d1 = Dog("Bruno", 3)                            # create the first object (calls __init__)
d2 = Dog("Luna", 5)                             # create the second object
print(d1.name, d1.age)                          # read attributes of d1
print(d2.bark())                                # call a method on d2
d1.birthday()                                   # d1 gets one year older
print(d1.age, d2.age)                           # only d1 changed, d2 is separate
print(d1.species, d2.species)                   # class attribute is shared
print(type(d1))                                 # the type of the object
print(isinstance(d1, Dog))                      # is d1 an object of class Dog?
```

**Output:**
```text
Bruno 3
Luna says Woof!
4 5
Canis familiaris Canis familiaris
<class '__main__.Dog'>
True
```

### 2.2 What is `self`?

`self` is the object **itself**. When you write `d1.bark()`, Python secretly calls `Dog.bark(d1)`, so `self` becomes `d1`. You must write `self` as the first parameter of every instance method, but you do **not** pass it when calling.

```python
class Greeter:                                  # a tiny class to understand self
    def __init__(self, name):                   # constructor
        self.name = name                        # store the name on the object

    def hello(self):                            # instance method
        return "Hello, " + self.name            # self gives access to this object's data

g = Greeter("Asha")                             # create an object
print(g.hello())                                # normal way
print(Greeter.hello(g))                         # what Python really does: pass the object as self
```

**Output:**
```text
Hello, Asha
Hello, Asha
```

### 2.3 Instance attributes vs class attributes

| | Instance attribute | Class attribute |
|---|---|---|
| Defined | inside `__init__` using `self.x` | directly inside the class body |
| Belongs to | one object | the class (shared by all objects) |
| Use for | data that differs per object (`name`) | constants, counters (`species`) |

```python
class Counter:                                  # counts how many objects were created
    count = 0                                   # class attribute: one copy shared by everyone

    def __init__(self):                         # constructor
        Counter.count += 1                      # change the shared counter through the class name
        self.id = Counter.count                 # instance attribute: this object's own number

a = Counter()                                   # first object
b = Counter()                                   # second object
c = Counter()                                   # third object
print(Counter.count, a.id, b.id, c.id)         # shared count and each object's id
a.count = 100                                   # creates a NEW instance attribute on a only
print(a.count, b.count, Counter.count)          # a hides the class attribute, others still see 3
```

**Output:**
```text
3 1 2 3
100 3 3
```

### 2.4 The mutable class attribute trap (classic bug)

```python
class Bad:                                      # wrong way
    items = []                                  # ONE list shared by every object

    def add(self, x):                           # add an item
        self.items.append(x)                    # modifies the shared list

class Good:                                     # right way
    def __init__(self):                         # constructor
        self.items = []                         # every object gets its OWN list

    def add(self, x):                           # add an item
        self.items.append(x)                    # modifies this object's list only

p, q = Bad(), Bad()                             # two Bad objects
p.add(1)                                        # add to p only
print(q.items)                                  # q sees it too! (shared list)

r, s = Good(), Good()                           # two Good objects
r.add(1)                                        # add to r only
print(s.items)                                  # s is unaffected
```

**Output:**
```text
[1]
[]
```

**Rule:** put **mutable** things (lists, dicts, sets) in `__init__` using `self.`, never directly in the class body.

### 2.5 `__str__` and `__repr__`

| Method | Purpose | Used by |
|---|---|---|
| `__str__` | readable text for users | `print(obj)`, `str(obj)` |
| `__repr__` | unambiguous text for developers (ideally looks like code) | the shell, `repr(obj)`, printing a list of objects |

```python
class Point:                                    # a 2D point
    def __init__(self, x, y):                   # constructor
        self.x = x                              # store x
        self.y = y                              # store y

    def __str__(self):                          # friendly text
        return f"({self.x}, {self.y})"          # looks like (2, 3)

    def __repr__(self):                         # developer text
        return f"Point({self.x}, {self.y})"     # looks like the code to recreate it

p = Point(2, 3)                                 # create a point
print(p)                                        # uses __str__
print(repr(p))                                  # uses __repr__
print([p])                                      # containers use __repr__ for their items
```

**Output:**
```text
(2, 3)
Point(2, 3)
[Point(2, 3)]
```

If you define only one, define `__repr__`. Python uses it as a fallback for `str()`.

---

## 3. Encapsulation

**Meaning:** keep the data **inside** the class and give controlled ways (methods) to read or change it. For example, nobody should set a bank balance to a negative number directly.

### 3.1 Public, protected, private (by convention)

Python has **no strict private**. It uses naming rules:

| Name | Meaning | Enforced? |
|---|---|---|
| `name` | public: anyone can use it | no restriction |
| `_name` | **protected**: "internal use, please do not touch" | convention only |
| `__name` | **private**: Python renames it to `_ClassName__name` (name mangling) | makes accidental access fail |

```python
class Account:                                  # a bank account
    def __init__(self, owner, balance):         # constructor
        self.owner = owner                      # public attribute
        self._bank = "ABC Bank"                 # protected by convention
        self.__balance = balance                # private (name mangled)

    def get_balance(self):                      # controlled way to read the balance
        return self.__balance                   # inside the class we can use the private name

a = Account("Asha", 1000)                       # create an account
print(a.owner)                                  # public: fine
print(a._bank)                                  # protected: works, but you should not
try:                                            # try the private attribute from outside
    print(a.__balance)                          # fails: the real name is different
except AttributeError:                          # Python raises AttributeError
    print("AttributeError: no direct access")   # show that it was blocked
print(a.get_balance())                          # the proper way
print(a._Account__balance)                      # name mangling revealed (never do this in real code)
print(sorted(a.__dict__))                       # all stored attribute names
```

**Output:**
```text
Asha
ABC Bank
AttributeError: no direct access
1000
1000
['_Account__balance', '_bank', 'owner']
```

### 3.2 `@property`: getters and setters the Pythonic way

`@property` lets you write `obj.celsius` like a normal attribute while Python runs a method behind the scenes. This allows **validation** without changing how other code uses your class.

```python
class Temperature:                              # stores temperature in Celsius
    def __init__(self, celsius):                # constructor
        self.celsius = celsius                  # this line goes through the setter below

    @property                                   # makes celsius readable like an attribute
    def celsius(self):                          # getter
        return self._celsius                    # return the hidden value

    @celsius.setter                             # runs when we write t.celsius = value
    def celsius(self, value):                   # setter with validation
        if value < -273.15:                     # physically impossible temperature
            raise ValueError("below absolute zero")   # reject it
        self._celsius = value                   # store the valid value

    @property                                   # a computed, read-only attribute
    def fahrenheit(self):                       # no setter, so it cannot be assigned
        return self._celsius * 9 / 5 + 32       # compute from celsius every time

t = Temperature(25)                             # create with 25 C
print(t.celsius, t.fahrenheit)                  # 25 C and the converted value
t.celsius = 100                                 # setter runs and validates
print(t.fahrenheit)                             # fahrenheit follows automatically
try:                                            # try an invalid value
    t.celsius = -300                            # setter will reject this
except ValueError as e:                         # catch the error
    print("Error:", e)                          # show the message
```

**Output:**
```text
25 77.0
212.0
Error: below absolute zero
```

---

## 4. Abstraction

**Meaning:** show **what** an object can do, hide **how** it does it. You drive a car using the steering wheel and pedals (the interface) without knowing how the engine works.

In Python, we create an **abstract class** using `ABC`. It can contain **abstract methods** (no body) that every child class **must** implement. You cannot create an object of an abstract class.

```python
from abc import ABC, abstractmethod             # tools for abstract classes

class Shape(ABC):                               # abstract base class (a contract)
    @abstractmethod                             # marks the method as required
    def area(self):                             # no real code here, only the promise
        pass                                    # children must provide the body

class Square(Shape):                            # a concrete class that follows the contract
    def __init__(self, side):                   # constructor
        self.side = side                        # store the side

    def area(self):                             # we MUST implement area
        return self.side ** 2                   # square area

class Broken(Shape):                            # a child that forgot to implement area
    pass                                        # nothing implemented

try:                                            # abstract classes cannot be instantiated
    Shape()                                     # not allowed
except TypeError:                               # Python raises TypeError
    print("TypeError: cannot create an object of an abstract class")   # explain the failure
try:                                            # a child with missing methods is also abstract
    Broken()                                    # not allowed either
except TypeError:                               # Python raises TypeError
    print("TypeError: Broken did not implement area")   # explain the failure
print(Square(4).area())                         # a complete child works fine
```

**Output:**
```text
TypeError: cannot create an object of an abstract class
TypeError: Broken did not implement area
16
```

**Abstraction vs encapsulation (a very common question):**

| Abstraction | Encapsulation |
|---|---|
| hides **complexity** (design level) | hides **data** (implementation level) |
| "what it does" | "how data is protected" |
| achieved with abstract classes / interfaces | achieved with private attributes and properties |

---

## 5. Inheritance

**Meaning:** a **child** class gets all attributes and methods of a **parent** class and can add or change things. It models an **"is-a"** relationship: a `Dog` **is an** `Animal`.

```python
class Animal:                                   # parent (base) class
    def __init__(self, name):                   # parent constructor
        self.name = name                        # store the name

    def speak(self):                            # general behaviour
        return f"{self.name} makes a sound"     # default text

class Dog(Animal):                              # child class: Dog inherits from Animal
    def __init__(self, name, breed):            # child constructor with an extra value
        super().__init__(name)                  # call the PARENT constructor to set name
        self.breed = breed                      # extra attribute only dogs have

    def speak(self):                            # method overriding: replace the parent's version
        return f"{self.name} barks"             # dog-specific text

class Puppy(Dog):                               # multilevel: Puppy inherits from Dog, which inherits from Animal
    def speak(self):                            # override again
        return super().speak() + " softly"      # reuse the parent's result and extend it

a = Animal("Generic")                           # a plain animal
d = Dog("Bruno", "Labrador")                    # a dog
p = Puppy("Tiny", "Pug")                        # a puppy
print(a.speak())                                # parent version
print(d.speak())                                # Dog version
print(p.speak())                                # Puppy version built on Dog's
print(isinstance(p, Dog), isinstance(p, Animal), isinstance(d, Puppy))   # is-a checks
print(issubclass(Puppy, Animal))                # class relationship check
print([c.__name__ for c in Puppy.__mro__])      # method resolution order: where Python searches
```

**Output:**
```text
Generic makes a sound
Bruno barks
Tiny barks softly
True True False
True
['Puppy', 'Dog', 'Animal', 'object']
```

### 5.1 Types of inheritance

| Type | Shape | Example |
|---|---|---|
| Single | `B(A)` | `Dog(Animal)` |
| Multilevel | `C(B)`, `B(A)` | `Puppy(Dog)`, `Dog(Animal)` |
| Hierarchical | `B(A)`, `C(A)` | `Dog(Animal)`, `Cat(Animal)` |
| Multiple | `D(B, C)` | a class with two parents |
| Hybrid | mix of the above | diamond shape |

### 5.2 Multiple inheritance and MRO (the diamond problem)

```
      A
     / \
    B   C
     \ /
      D
```

If `B` and `C` both override a method of `A`, which one does `D` use? Python uses the **MRO (Method Resolution Order)**, computed with the **C3 linearization** rule: children before parents, and the order of the parents as you wrote them. `super()` follows the MRO, so every class in the diamond runs **exactly once**.

```python
class A:                                        # top of the diamond
    def hello(self):                            # base method
        return "A"                              # ends the chain

class B(A):                                     # left side
    def hello(self):                            # override
        return "B" + super().hello()            # super() goes to the NEXT class in the MRO

class C(A):                                     # right side
    def hello(self):                            # override
        return "C" + super().hello()            # super() goes to the NEXT class in the MRO

class D(B, C):                                  # bottom of the diamond, parents listed as B then C
    def hello(self):                            # override
        return "D" + super().hello()            # next in the MRO is B

print(D().hello())                              # D, then B, then C, then A (each once)
print([c.__name__ for c in D.__mro__])          # the search order
```

**Output:**
```text
DBCA
['D', 'B', 'C', 'A', 'object']
```

Note that `B`'s `super()` called `C`, not `A`. That is the power of the MRO: `super()` means "the next class in the MRO", not "the parent".

### 5.3 What does `super()` do?

`super()` gives access to the **next class in the MRO**. Use it to call the parent's `__init__` and to extend (not replace) parent methods. Always call `super().__init__(...)` in a child constructor, unless you purposely want to skip the parent's setup.

---

## 6. Polymorphism

**Meaning:** "many forms". The **same method name** works on different objects, and each object responds in its own way.

### 6.1 Method overriding (runtime polymorphism)

```python
class Bird:                                     # base class
    def fly(self):                              # general behaviour
        return "Some birds can fly"             # default text

class Eagle(Bird):                              # child class
    def fly(self):                              # override
        return "Eagle soars high"               # eagle-specific

class Penguin(Bird):                            # child class
    def fly(self):                              # override
        return "Penguin cannot fly"             # penguin-specific

for bird in [Bird(), Eagle(), Penguin()]:       # one loop, three different behaviours
    print(bird.fly())                           # Python picks the right fly() at runtime
```

**Output:**
```text
Some birds can fly
Eagle soars high
Penguin cannot fly
```

### 6.2 Duck typing: "if it quacks like a duck..."

Python does not care about the **type** of an object, only whether it has the **method** you call. Unrelated classes can be used interchangeably.

```python
class Duck:                                     # a class with speak()
    def speak(self):                            # method
        return "Quack"                          # duck sound

class Robot:                                    # an unrelated class, also with speak()
    def speak(self):                            # same method name
        return "Beep"                           # robot sound

for thing in [Duck(), Robot()]:                 # no common parent needed
    print(thing.speak())                        # works because both have speak()
print(len("abc"), len([1, 2]), len({"a": 1}))   # built-in len() is polymorphic too
```

**Output:**
```text
Quack
Beep
3 2 1
```

### 6.3 Method overloading: Python does not have it

In Java you can define `add(a)` and `add(a, b)` separately. In Python, a second `def add` **replaces** the first. Use **default arguments**, `*args` or `None` checks instead.

```python
class Calc:                                     # a calculator
    def add(self, a, b=0, c=0):                 # default values make b and c optional
        return a + b + c                        # works for 1, 2 or 3 numbers

    def total(self, *numbers):                  # *numbers accepts any count of arguments
        return sum(numbers)                     # add them all

c = Calc()                                      # create the calculator
print(c.add(1))                                 # one argument
print(c.add(1, 2))                              # two arguments
print(c.add(1, 2, 3))                           # three arguments
print(c.total(4, 5, 6, 7))                      # any number of arguments
```

**Output:**
```text
1
3
6
22
```

### 6.4 Operator overloading

You can make `+`, `==`, `<`, `len()` and more work on **your own objects** by defining magic methods (see Section 8 and Problem 6).

---

## 7. Class methods and static methods

Python has **three kinds of methods**:

| Kind | Decorator | First parameter | Can access | Typical use |
|---|---|---|---|---|
| **Instance method** | none | `self` (the object) | object data and class data | normal behaviour |
| **Class method** | `@classmethod` | `cls` (the class) | class data only | alternative constructors, change class-level settings |
| **Static method** | `@staticmethod` | nothing | neither | helper/utility that belongs logically to the class |

```python
class Employee:                                 # an employee record
    company = "TechCorp"                        # class attribute shared by all employees
    count = 0                                   # how many employees were created

    def __init__(self, name, salary):           # normal constructor
        self.name = name                        # store the name
        self.salary = salary                    # store the salary
        Employee.count += 1                     # one more employee exists

    def raise_salary(self, percent):            # instance method: needs one specific employee
        self.salary += self.salary * percent // 100   # increase this employee's salary

    @classmethod                                # works with the class, not one object
    def from_string(cls, text):                 # alternative constructor: build from "name-salary"
        name, salary = text.split("-")          # break the text into two parts
        return cls(name, int(salary))           # cls(...) creates an object (works for child classes too)

    @classmethod                                # another class method
    def change_company(cls, new_name):          # changes the shared class attribute
        cls.company = new_name                  # affects every employee

    @staticmethod                               # no self, no cls: just a helper function
    def is_valid_salary(amount):                # checks a number
        return amount > 0                       # valid if positive

e = Employee.from_string("Asha-50000")          # build an object from a string
print(e.name, e.salary)                         # the fields were parsed
e.raise_salary(10)                              # instance method: 10 percent raise
print(e.salary)                                 # new salary
print(Employee.is_valid_salary(-5))             # static method called through the class
Employee.change_company("NewCorp")             # class method changes shared data
print(e.company)                                # the object sees the change
print(Employee.count)                           # number of objects created so far
```

**Output:**
```text
Asha 50000
55000
False
NewCorp
1
```

**Why `cls(...)` instead of `Employee(...)`?** If a child class `Manager(Employee)` calls `Manager.from_string(...)`, `cls` is `Manager`, so a `Manager` object is created. Hard-coding `Employee(...)` would wrongly create an `Employee`.

---

## 8. Magic (dunder) methods

Methods with **double underscores** on both sides (`__init__`, `__str__`) are **magic methods**. Python calls them automatically when you use certain syntax. They let your objects behave like built-in types.

| Syntax you write | Method Python calls |
|---|---|
| `Cls(...)` | `__init__` (after `__new__`) |
| `print(obj)` / `str(obj)` | `__str__` |
| `repr(obj)` | `__repr__` |
| `len(obj)` | `__len__` |
| `obj[i]` | `__getitem__` |
| `x in obj` | `__contains__` |
| `for x in obj` | `__iter__` (and `__next__`) |
| `obj()` | `__call__` |
| `a + b`, `a - b`, `a * b`, `a / b` | `__add__`, `__sub__`, `__mul__`, `__truediv__` |
| `a == b`, `a != b` | `__eq__`, `__ne__` |
| `a < b`, `a <= b`, `a > b`, `a >= b` | `__lt__`, `__le__`, `__gt__`, `__ge__` |
| `bool(obj)` / `if obj:` | `__bool__` (or `__len__`) |
| `hash(obj)` / use as dict key | `__hash__` |
| `with obj:` | `__enter__`, `__exit__` |
| `del obj` | `__del__` |

```python
class Playlist:                                 # a collection of songs
    def __init__(self, songs):                  # constructor
        self.songs = list(songs)                # keep our own copy of the songs

    def __len__(self):                          # makes len(playlist) work
        return len(self.songs)                  # number of songs

    def __getitem__(self, index):               # makes playlist[i] work
        return self.songs[index]                # return the song at that position

    def __contains__(self, song):               # makes "x in playlist" work
        return song in self.songs               # reuse the list's check

    def __iter__(self):                         # makes "for song in playlist" work
        return iter(self.songs)                 # hand out an iterator over the songs

    def __eq__(self, other):                    # makes playlist1 == playlist2 work
        return isinstance(other, Playlist) and self.songs == other.songs   # same type and same songs

    def __call__(self, index):                  # makes playlist(0) work like a function
        return f"Now playing: {self.songs[index]}"   # build a message

p = Playlist(["a", "b", "c"])                   # create a playlist
print(len(p))                                   # __len__
print(p[1])                                     # __getitem__
print("c" in p)                                 # __contains__
print([s.upper() for s in p])                   # __iter__
print(p == Playlist(["a", "b", "c"]))           # __eq__
print(p(0))                                     # __call__
```

**Output:**
```text
3
b
True
['A', 'B', 'C']
True
Now playing: a
```

> **Important:** if you define `__eq__` but not `__hash__`, Python sets `__hash__` to `None`, so the objects become **unhashable** (cannot be dict keys or set items). If you need both, define `__hash__` yourself, for example `return hash(tuple(self.songs))`, but only for **immutable** data.

### 8.1 Context manager (`with` statement)

```python
class Session:                                  # a resource that must be opened and closed
    def __enter__(self):                        # runs when the with-block starts
        print("open")                           # acquire the resource
        return self                             # this value goes to "as s"

    def __exit__(self, exc_type, exc, tb):      # runs when the with-block ends, even after an error
        print("close")                          # release the resource
        return False                            # False means: do not hide exceptions

with Session() as s:                            # __enter__ is called here
    print("working")                            # the body of the with-block
```

**Output:**
```text
open
working
close
```

---

## 9. Composition vs inheritance

| | Inheritance | Composition |
|---|---|---|
| Relationship | **is-a** (`Dog` is an `Animal`) | **has-a** (`Car` has an `Engine`) |
| How | `class Child(Parent)` | an attribute that holds another object |
| Coupling | tight (child depends on parent's details) | loose (parts can be swapped) |
| Use when | true "is-a" and shared behaviour | building something out of parts |

**Advice used in industry:** *"Prefer composition over inheritance."* Use inheritance only for a real is-a relationship.

```python
class Engine:                                   # a part
    def start(self):                            # behaviour of the part
        return "Engine started"                 # message

class Wheels:                                   # another part
    def __init__(self, count=4):                # constructor with a default
        self.count = count                      # number of wheels

class Car:                                      # the whole is built from parts
    def __init__(self):                         # constructor
        self.engine = Engine()                  # Car HAS-A Engine (composition)
        self.wheels = Wheels()                  # Car HAS-A Wheels

    def start(self):                            # Car delegates the work to its engine
        return f"{self.engine.start()}, rolling on {self.wheels.count} wheels"   # combine the parts

print(Car().start())                            # one call uses several objects
```

**Output:**
```text
Engine started, rolling on 4 wheels
```

---

## 10. Dataclasses and enums

### 10.1 `@dataclass`: less boilerplate

For classes that mostly **store data**, `@dataclass` writes `__init__`, `__repr__` and `__eq__` for you.

```python
from dataclasses import dataclass, field        # dataclass tools

@dataclass                                      # auto-generate __init__, __repr__, __eq__
class Item:                                     # a simple data holder
    name: str                                   # required field with a type hint
    price: float                                # required field
    qty: int = 1                                # field with a default value
    tags: list = field(default_factory=list)    # mutable default must use default_factory

i = Item("pen", 10.5)                           # create an item (qty and tags use defaults)
print(i)                                        # auto-generated repr
print(i == Item("pen", 10.5))                   # auto-generated equality compares all fields

@dataclass(frozen=True)                         # frozen means immutable (and hashable)
class Pt:                                       # an immutable point
    x: int                                      # field x
    y: int                                      # field y

p = Pt(1, 2)                                    # create a frozen point
try:                                            # try to change it
    p.x = 5                                     # not allowed
except Exception as e:                          # dataclasses raise FrozenInstanceError
    print(type(e).__name__)                     # show the error type
print({p: "ok"})                                # frozen dataclasses can be dict keys
```

**Output:**
```text
Item(name='pen', price=10.5, qty=1, tags=[])
True
FrozenInstanceError
{Pt(x=1, y=2): 'ok'}
```

### 10.2 `Enum`: named constants

```python
from enum import Enum                           # enum support

class Color(Enum):                              # a fixed set of named values
    RED = 1                                     # member RED with value 1
    GREEN = 2                                   # member GREEN with value 2

print(Color.RED)                                # the member
print(Color.RED.name, Color.RED.value)          # its name and value
print(Color(2))                                 # look up a member by value
print(Color.GREEN == Color.GREEN, Color.RED == Color.GREEN)   # compare members
print([c.name for c in Color])                  # loop over all members
```

**Output:**
```text
Color.RED
RED 1
Color.GREEN
True False
['RED', 'GREEN']
```

---

## 11. Design principles and patterns

### 11.1 SOLID in simple words

| Letter | Principle | Simple meaning | Example |
|---|---|---|---|
| **S** | Single Responsibility | a class should have **one reason to change** | `Invoice` calculates totals; `InvoicePrinter` prints them |
| **O** | Open / Closed | open for **extension**, closed for **modification** | add a new `Shape` subclass without editing old code |
| **L** | Liskov Substitution | a child must be usable **wherever the parent is expected** | if `Bird` has `fly()`, a `Penguin` child breaks it, so redesign |
| **I** | Interface Segregation | many small interfaces beat one big one | `Printable`, `Scannable` instead of one `Machine` with everything |
| **D** | Dependency Inversion | depend on **abstractions**, not concrete classes | `Notifier` takes any `Sender`, not only `EmailSender` |

### 11.2 Patterns you must recognize

| Pattern | Problem it solves | Problem number |
|---|---|---|
| **Singleton** | exactly **one** object of a class (logger, config) | 12 |
| **Factory** | create objects without writing `if kind == ...` everywhere | 12 |
| **Observer** | when one object changes, many others are told (events, notifications) | 13 |
| **Strategy** | swap algorithms at runtime (different discount rules) | Section 12 |
| **State** | object's behaviour depends on its current state (vending machine, traffic light) | extra practice |
| **Iterator** | walk through a collection without exposing its structure | 9 |

---

## 12. Common interview questions and mistakes

### 12.1 Questions with short answers

| Question | Short answer |
|---|---|
| What is the difference between a class and an object? | class = blueprint, object = instance built from it |
| What is `self`? | the current object, passed automatically as the first argument of instance methods |
| `__init__` vs `__new__`? | `__new__` **creates** the object, `__init__` **initializes** it. Rarely override `__new__` (used for Singleton and immutable types) |
| Is there real private in Python? | No. `__x` is name-mangled to `_Class__x`; it prevents accidents, not determined access |
| Method overloading in Python? | Not supported. Use default arguments, `*args`, or `functools.singledispatch` |
| Method overriding? | child defines a method with the same name as the parent's to change behaviour |
| What is MRO? | the order Python searches classes for a method; see `Class.__mro__` |
| `super()` does what? | calls the **next class in the MRO** |
| Abstract class vs interface? | Python has no `interface` keyword; an abstract class with only abstract methods plays that role |
| `@staticmethod` vs `@classmethod`? | static gets nothing automatically; class method gets `cls` |
| What is duck typing? | what matters is that the object has the needed methods, not its type |
| Shallow vs deep copy of objects? | `copy.copy` copies the outer object only; `copy.deepcopy` copies nested objects too |
| What is `__slots__`? | declares fixed attributes to save memory and block new attributes |
| What are `==` and `is`? | `==` compares values (`__eq__`), `is` compares identity (same object) |

### 12.2 Strategy pattern in a few lines

```python
class Order:                                    # an order that can use different discount rules
    def __init__(self, amount, discount_rule):  # we INJECT the rule (strategy) from outside
        self.amount = amount                    # order amount
        self.discount_rule = discount_rule      # a function that returns the discounted amount

    def final_amount(self):                     # compute the payable amount
        return self.discount_rule(self.amount)  # delegate the work to the chosen strategy

no_discount = lambda amt: amt                   # strategy 1: no discount
festival = lambda amt: amt * 0.8                # strategy 2: 20 percent off
print(Order(1000, no_discount).final_amount())  # uses strategy 1
print(Order(1000, festival).final_amount())     # uses strategy 2, no change to the Order class
```

**Output:**
```text
1000
800.0
```

### 12.3 Top mistakes

1. Forgetting `self` in a method definition (`def bark():` gives `TypeError` when called).
2. Forgetting `self.` before attribute names (`name = name` instead of `self.name = name`).
3. Mutable class attributes shared by all objects (Section 2.4).
4. Not calling `super().__init__()` in the child constructor, so parent attributes are missing.
5. Defining `__eq__` without `__hash__` and then using the object in a set or as a dict key.
6. Using inheritance for "has-a" relationships (a `Car` is **not** an `Engine`).
7. Using a mutable default argument: `def __init__(self, items=[])`. Use `None` and create the list inside.
8. Confusing `__str__` and `__repr__`.
9. Returning a private list directly (`return self.__items`) so callers can mutate it. Return a copy.
10. Giant classes that do everything. Split by responsibility.

---

## 13. The 15 problems

| # | Problem | Level | Main OOP idea |
|---|---|---|---|
| 1 | Student Report Card | Easy | class, object, methods |
| 2 | Rectangle with Validation | Easy | class attribute, `@property`, `__str__` |
| 3 | Bank Account | Easy | encapsulation, custom exception |
| 4 | Employee and Manager | Medium | inheritance, `super()`, overriding |
| 5 | Shapes (Abstract Class) | Medium | abstraction, polymorphism |
| 6 | Fraction Calculator | Medium | operator overloading, `total_ordering` |
| 7 | Product with Tax Rules | Medium | `@classmethod`, `@staticmethod` |
| 8 | Min Stack | Medium | data structure as a class |
| 9 | Custom Iterator and Generator | Medium | `__iter__`, `__next__`, `yield` |
| 10 | Library Management System | Medium | composition, several cooperating classes |
| 11 | LRU Cache | Hard | doubly linked list + hash map as a class |
| 12 | Singleton and Factory | Hard | design patterns |
| 13 | Observer (News Agency) | Hard | design pattern, loose coupling |
| 14 | Parking Lot | Hard | enum, abstract design, system design |
| 15 | Tic-Tac-Toe | Hard | game design, O(1) win check, custom errors |

For each problem you get: the question, the idea, the code **with every line explained in a comment**, the output, and a note on the concepts used.

---

### Problem 1: Student Report Card (Easy)

**Question:** Create a `Student` class with a name and roll number. It should store marks per subject, compute the **average**, and return a **grade** (A for 90 and above, B for 75 and above, C for 50 and above, F otherwise).

**Concepts:** class, `__init__`, instance attributes, methods calling other methods, `__str__`.

```python
class Student:                                        # blueprint for a student
    def __init__(self, name, roll_no):                # constructor receives name and roll number
        self.name = name                              # store the name on this object
        self.roll_no = roll_no                        # store the roll number
        self.marks = {}                               # dictionary: subject -> marks

    def add_marks(self, subject, score):              # record marks for one subject
        self.marks[subject] = score                   # save (or overwrite) the score

    def average(self):                                # compute the average marks
        if not self.marks:                            # no subjects recorded yet
            return 0                                  # avoid dividing by zero
        return sum(self.marks.values()) / len(self.marks)   # total marks divided by number of subjects

    def grade(self):                                  # convert the average into a grade letter
        avg = self.average()                          # reuse average() instead of repeating its logic
        if avg >= 90:                                 # top band
            return "A"                                # grade A
        if avg >= 75:                                 # second band
            return "B"                                # grade B
        if avg >= 50:                                 # third band
            return "C"                                # grade C
        return "F"                                    # everything else fails

    def __str__(self):                                # readable text for print()
        return f"Student({self.name}, roll {self.roll_no})"   # name and roll number

s = Student("Asha", 101)                              # create a student object
s.add_marks("Maths", 95)                              # add first subject
s.add_marks("Physics", 88)                            # add second subject
s.add_marks("Chemistry", 92)                          # add third subject
print(s)                                              # uses __str__
print(f"{s.average():.2f}")                           # average rounded to 2 decimals
print(s.grade())                                      # grade letter
s2 = Student("Ravi", 102)                             # a student with no marks yet
print(s2.average(), s2.grade())                       # average 0 gives grade F
```

**Output:**
```text
Student(Asha, roll 101)
91.67
A
0 F
```

**Takeaway:** each object keeps its own `marks` dictionary, and methods use `self` to reach it.

---

### Problem 2: Rectangle with Validation (Easy)

**Question:** Create a `Rectangle` class with `length` and `width`. Reject non-positive values. Provide `area()`, `perimeter()`, `is_square()` and a readable string. All rectangles have `sides = 4`.

**Concepts:** class attribute, `@property` with a setter for validation, `__str__`.

```python
class Rectangle:                                      # a rectangle shape
    sides = 4                                         # class attribute: true for every rectangle

    def __init__(self, length, width):                # constructor
        self.length = length                          # goes through the length setter (validates)
        self.width = width                            # goes through the width setter (validates)

    @property                                         # lets us read r.length like a normal attribute
    def length(self):                                 # getter for length
        return self._length                           # return the hidden value

    @length.setter                                    # runs on r.length = value
    def length(self, value):                          # setter for length
        if value <= 0:                                # invalid input
            raise ValueError("length must be positive")   # reject it
        self._length = value                          # store the valid value

    @property                                         # lets us read r.width like a normal attribute
    def width(self):                                  # getter for width
        return self._width                            # return the hidden value

    @width.setter                                     # runs on r.width = value
    def width(self, value):                           # setter for width
        if value <= 0:                                # invalid input
            raise ValueError("width must be positive")    # reject it
        self._width = value                           # store the valid value

    def area(self):                                   # area of the rectangle
        return self.length * self.width               # length times width

    def perimeter(self):                              # perimeter of the rectangle
        return 2 * (self.length + self.width)         # twice the sum of the sides

    def is_square(self):                              # check whether it is a square
        return self.length == self.width              # equal sides

    def __str__(self):                                # readable text
        return f"Rectangle({self.length} x {self.width})"   # show the dimensions

r = Rectangle(5, 3)                                   # a 5 x 3 rectangle
sq = Rectangle(4, 4)                                  # a square
print(r)                                              # uses __str__
print(r.area(), r.perimeter())                        # 15 and 16
print(r.is_square(), sq.is_square())                  # False and True
print(Rectangle.sides)                                # class attribute through the class
r.width = 10                                          # the setter validates and updates
print(r.area())                                       # area uses the new width
try:                                                  # try to create an invalid rectangle
    Rectangle(-1, 2)                                  # negative length
except ValueError as e:                               # catch the validation error
    print("Error:", e)                                # show the message
```

**Output:**
```text
Rectangle(5 x 3)
15 16
False True
4
50
Error: length must be positive
```

**Takeaway:** validating in the setter protects the object even when the value is changed later, not only at creation.

---

### Problem 3: Bank Account (Easy)

**Question:** Create a `BankAccount` where the balance cannot be changed directly. Support `deposit`, `withdraw` (error if funds are insufficient) and a transaction history.

**Concepts:** private attributes, read-only property, custom exception, returning a copy.

```python
class InsufficientFundsError(Exception):              # our own exception type
    pass                                              # no extra code needed

class BankAccount:                                    # the account class
    def __init__(self, owner, balance=0):             # constructor with a default balance
        self.owner = owner                            # public: the owner's name
        self.__balance = balance                      # private: hidden from outside code
        self.__history = []                           # private: list of (type, amount)

    @property                                         # read-only access to the balance
    def balance(self):                                # getter only, no setter
        return self.__balance                         # return the private value

    def deposit(self, amount):                        # add money
        if amount <= 0:                               # invalid amount
            raise ValueError("Deposit must be positive")   # reject it
        self.__balance += amount                      # increase the balance
        self.__history.append(("deposit", amount))    # record the transaction

    def withdraw(self, amount):                       # take money out
        if amount <= 0:                               # invalid amount
            raise ValueError("Withdrawal must be positive")    # reject it
        if amount > self.__balance:                   # not enough money
            raise InsufficientFundsError(f"Balance {self.__balance}, requested {amount}")   # explain
        self.__balance -= amount                      # decrease the balance
        self.__history.append(("withdraw", amount))   # record the transaction

    def statement(self):                              # give the history to the caller
        return list(self.__history)                   # return a COPY so callers cannot edit our list

acc = BankAccount("Asha", 1000)                       # open an account with 1000
acc.deposit(500)                                      # balance becomes 1500
acc.withdraw(300)                                     # balance becomes 1200
print(acc.balance)                                    # read through the property
try:                                                  # try to overdraw
    acc.withdraw(5000)                                # more than the balance
except InsufficientFundsError as e:                   # catch our custom error
    print("Error:", e)                                # show the message
try:                                                  # try a negative deposit
    acc.deposit(-5)                                   # invalid
except ValueError as e:                               # catch the error
    print("Error:", e)                                # show the message
try:                                                  # try to set the balance directly
    acc.balance = 0                                   # property has no setter
except AttributeError:                                # Python blocks it
    print("Cannot set balance directly")              # explain
print(acc.statement())                                # the transaction history
```

**Output:**
```text
1200
Error: Balance 1200, requested 5000
Error: Deposit must be positive
Cannot set balance directly
[('deposit', 500), ('withdraw', 300)]
```

**Takeaway:** encapsulation means **all changes go through methods that enforce the rules**.

---

### Problem 4: Employee and Manager (Medium)

**Question:** Create `Employee` with a name and base salary. A `Manager` gets a bonus and manages a team. An `Intern` earns half the base salary. Every class must have `salary()` and `describe()`.

**Concepts:** inheritance, `super().__init__`, method overriding, polymorphism (the parent's `describe()` calls the child's `salary()`).

```python
class Employee:                                       # parent class
    def __init__(self, name, base_salary):            # constructor
        self.name = name                              # store the name
        self.base_salary = base_salary                # store the base salary

    def salary(self):                                 # how much this person earns
        return self.base_salary                       # a normal employee earns the base salary

    def describe(self):                               # text for reports
        return f"{self.name} earns {self.salary()}"   # self.salary() runs the CHILD's version if it exists

class Manager(Employee):                              # a Manager is an Employee
    def __init__(self, name, base_salary, bonus):     # extra value: bonus
        super().__init__(name, base_salary)           # let the parent set name and base_salary
        self.bonus = bonus                            # manager-only attribute
        self.team = []                                # people reporting to this manager

    def add_member(self, employee):                   # manager-only method
        self.team.append(employee)                    # add a person to the team

    def salary(self):                                 # override: managers earn base + bonus
        return super().salary() + self.bonus          # reuse the parent's value and add the bonus

class Intern(Employee):                               # an Intern is an Employee
    def salary(self):                                 # override: interns earn half
        return self.base_salary // 2                  # integer half of the base salary

e = Employee("Ravi", 50000)                           # a normal employee
m = Manager("Asha", 80000, 20000)                     # a manager with a bonus
i = Intern("Tiny", 30000)                             # an intern
m.add_member(e)                                       # Ravi reports to Asha
m.add_member(i)                                       # Tiny reports to Asha
for person in [e, m, i]:                              # one loop for three different classes
    print(person.describe())                          # each object uses its own salary()
print(len(m.team))                                    # team size
print(sum(p.salary() for p in [e, m, i]))             # total payroll
print(isinstance(m, Employee), isinstance(e, Manager))    # is-a checks
```

**Output:**
```text
Ravi earns 50000
Asha earns 100000
Tiny earns 15000
2
165000
True False
```

**Takeaway:** `describe()` is written **once** in the parent but works correctly for every child, because `self.salary()` finds the right version at runtime.

---

### Problem 5: Shapes with an Abstract Class (Medium)

**Question:** Design an abstract `Shape` with `area()` and `perimeter()`. Implement `Circle`, `Rectangle` and `Triangle` (from three sides). Print the details of a list of shapes and find the one with the largest area.

**Concepts:** `ABC`, `@abstractmethod`, polymorphism, inheritance of a shared method (`describe`).

```python
from abc import ABC, abstractmethod                   # tools for abstract classes
import math                                           # for pi and square root

class Shape(ABC):                                     # abstract base class
    @abstractmethod                                   # every shape MUST provide area
    def area(self):                                   # no body here
        pass                                          # the promise only

    @abstractmethod                                   # every shape MUST provide perimeter
    def perimeter(self):                              # no body here
        pass                                          # the promise only

    def describe(self):                               # shared, ready-made method for all shapes
        name = self.__class__.__name__                # the real class name, e.g. "Circle"
        return f"{name}: area={self.area():.2f}, perimeter={self.perimeter():.2f}"   # formatted text

class Circle(Shape):                                  # concrete shape 1
    def __init__(self, radius):                       # constructor
        self.radius = radius                          # store the radius

    def area(self):                                   # implement area
        return math.pi * self.radius ** 2             # pi r squared

    def perimeter(self):                              # implement perimeter
        return 2 * math.pi * self.radius              # circumference

class Rectangle(Shape):                               # concrete shape 2
    def __init__(self, length, width):                # constructor
        self.length = length                          # store the length
        self.width = width                            # store the width

    def area(self):                                   # implement area
        return self.length * self.width               # length times width

    def perimeter(self):                              # implement perimeter
        return 2 * (self.length + self.width)         # twice the sum

class Triangle(Shape):                                # concrete shape 3
    def __init__(self, a, b, c):                      # constructor with three sides
        self.a, self.b, self.c = a, b, c              # store the sides

    def area(self):                                   # implement area using Heron's formula
        s = (self.a + self.b + self.c) / 2            # semi-perimeter
        return math.sqrt(s * (s - self.a) * (s - self.b) * (s - self.c))   # Heron's formula

    def perimeter(self):                              # implement perimeter
        return self.a + self.b + self.c               # sum of the sides

shapes = [Circle(3), Rectangle(4, 5), Triangle(3, 4, 5)]   # three different shapes in one list
for shape in shapes:                                  # polymorphism: same call, different behaviour
    print(shape.describe())                           # describe() is inherited from Shape
biggest = max(shapes, key=lambda s: s.area())         # pick the shape with the largest area
print("Largest:", biggest.__class__.__name__)         # show its class name
try:                                                  # abstract classes cannot be created
    Shape()                                           # not allowed
except TypeError:                                     # Python raises TypeError
    print("Shape is abstract")                        # confirm
```

**Output:**
```text
Circle: area=28.27, perimeter=18.85
Rectangle: area=20.00, perimeter=18.00
Triangle: area=6.00, perimeter=12.00
Largest: Circle
Shape is abstract
```

**Takeaway:** adding a new shape later needs **no change** to existing code (Open/Closed principle).

---

### Problem 6: Fraction Calculator (Medium)

**Question:** Create a `Fraction` class that is always stored in lowest terms. Support `+ - * /`, comparison operators, printing and sorting.

**Concepts:** operator overloading (`__add__`, `__sub__`, `__mul__`, `__truediv__`, `__eq__`, `__lt__`), `functools.total_ordering`, `__str__` and `__repr__`.

```python
from math import gcd                                  # greatest common divisor for simplifying
from functools import total_ordering                  # fills in <=, >, >= from __eq__ and __lt__

@total_ordering                                       # we only write __eq__ and __lt__
class Fraction:                                       # a fraction num/den
    def __init__(self, num, den=1):                   # constructor, denominator defaults to 1
        if den == 0:                                  # a fraction cannot have a zero denominator
            raise ZeroDivisionError("denominator cannot be zero")   # reject it
        if den < 0:                                   # keep the sign on the numerator
            num, den = -num, -den                     # move the minus sign up
        g = gcd(abs(num), den)                        # largest number dividing both parts
        self.num = num // g                           # reduce the numerator
        self.den = den // g                           # reduce the denominator

    def __add__(self, other):                         # makes a + b work
        return Fraction(self.num * other.den + other.num * self.den, self.den * other.den)   # cross-multiply

    def __sub__(self, other):                         # makes a - b work
        return Fraction(self.num * other.den - other.num * self.den, self.den * other.den)   # cross-multiply

    def __mul__(self, other):                         # makes a * b work
        return Fraction(self.num * other.num, self.den * other.den)   # multiply straight across

    def __truediv__(self, other):                     # makes a / b work
        return Fraction(self.num * other.den, self.den * other.num)   # multiply by the reciprocal

    def __eq__(self, other):                          # makes a == b work
        return self.num == other.num and self.den == other.den   # equal because both are reduced

    def __lt__(self, other):                          # makes a < b work
        return self.num * other.den < other.num * self.den   # compare by cross-multiplying

    def __str__(self):                                # text for print()
        return f"{self.num}/{self.den}" if self.den != 1 else str(self.num)   # hide /1

    def __repr__(self):                               # text for lists and debugging
        return f"Fraction({self.num}, {self.den})"    # looks like the constructor call

a = Fraction(1, 2)                                    # one half
b = Fraction(1, 3)                                    # one third
print(a + b)                                          # 1/2 + 1/3
print(a - b)                                          # 1/2 - 1/3
print(a * b)                                          # 1/2 * 1/3
print(a / b)                                          # 1/2 divided by 1/3
print(a > b, a == Fraction(2, 4), a <= b)             # comparisons (2/4 reduces to 1/2)
print(Fraction(4, 2))                                 # 4/2 reduces to 2
print(sorted([Fraction(3, 4), Fraction(1, 2), Fraction(2, 3)]))   # sorting uses __lt__
try:                                                  # division by a zero fraction
    a / Fraction(0, 1)                                # becomes denominator 0
except ZeroDivisionError as e:                        # catch the error
    print("Error:", e)                                # show the message
```

**Output:**
```text
5/6
1/6
1/6
3/2
True True False
2
[Fraction(1, 2), Fraction(2, 3), Fraction(3, 4)]
Error: denominator cannot be zero
```

**Takeaway:** magic methods let your own class feel like a built-in type.

---

### Problem 7: Product with Tax Rules (Medium)

**Question:** Create a `Product` class with a shared tax rate. Support building a product from a string like `"Pen, 100"`, changing the tax rate for **all** products, and checking whether a price is valid.

**Concepts:** `@classmethod` (alternative constructor, shared settings), `@staticmethod` (utility), class attributes.

```python
class Product:                                        # a product on sale
    tax_rate = 0.18                                   # class attribute: tax shared by every product
    count = 0                                         # class attribute: how many products exist

    def __init__(self, name, price):                  # normal constructor
        self.name = name                              # store the name
        self.price = price                            # store the price
        Product.count += 1                            # one more product was created

    @classmethod                                      # receives the class, not an object
    def from_string(cls, text):                       # alternative constructor
        name, price = text.split(",")                 # split "Pen, 100" into two parts
        return cls(name.strip(), float(price))        # clean the name, convert price, build the object

    @classmethod                                      # another class-level operation
    def set_tax_rate(cls, rate):                      # change the shared tax rate
        cls.tax_rate = rate                           # affects every product

    @staticmethod                                     # no self, no cls needed
    def is_valid_price(price):                        # a pure helper function
        return price > 0                              # prices must be positive

    def price_with_tax(self):                         # instance method using shared data
        return round(self.price * (1 + Product.tax_rate), 2)   # add tax, round to 2 decimals

p = Product.from_string("Pen, 100")                   # build from text
print(p.name, p.price)                                # parsed values
print(p.price_with_tax())                             # 100 plus 18 percent tax
Product.set_tax_rate(0.05)                            # tax changes for everyone
print(p.price_with_tax())                             # 100 plus 5 percent tax
print(Product.is_valid_price(-1), Product.is_valid_price(10))   # static helper
book = Product("Book", 250)                           # build normally
print(Product.count)                                  # two products exist now
```

**Output:**
```text
Pen 100.0
118.0
105.0
False True
2
```

**Takeaway:** use `@classmethod` for **alternative constructors** and shared settings, and `@staticmethod` for helpers that need neither the object nor the class.

---

### Problem 8: Min Stack (Medium)

**Question:** Design a stack with `push`, `pop`, `top` and `get_min` that all run in **O(1)** time.

**Idea:** Store each element together with **the minimum so far**. When you pop, the previous minimum is still stored below.

**Concepts:** data structure as a class, `__len__`, raising exceptions for invalid operations.

```python
class MinStack:                                       # stack that knows its minimum
    def __init__(self):                               # constructor
        self.stack = []                               # list of (value, minimum so far) pairs

    def push(self, x):                                # add a value on top
        if self.stack:                                # the stack already has items
            current_min = min(x, self.stack[-1][1])   # smaller of x and the previous minimum
        else:                                         # first item
            current_min = x                           # it is the minimum by itself
        self.stack.append((x, current_min))           # store the value with the running minimum

    def pop(self):                                    # remove and return the top value
        if not self.stack:                            # nothing to pop
            raise IndexError("pop from empty stack")  # signal the error
        return self.stack.pop()[0]                    # remove the pair, return only the value

    def top(self):                                    # look at the top value without removing
        if not self.stack:                            # nothing to look at
            raise IndexError("stack is empty")        # signal the error
        return self.stack[-1][0]                      # value part of the top pair

    def get_min(self):                                # smallest value currently in the stack
        if not self.stack:                            # empty stack has no minimum
            raise IndexError("stack is empty")        # signal the error
        return self.stack[-1][1]                      # minimum part of the top pair

    def __len__(self):                                # makes len(stack) work
        return len(self.stack)                        # number of stored pairs

ms = MinStack()                                       # create the stack
for x in [5, 3, 7, 2]:                                # push four values
    ms.push(x)                                        # each push is O(1)
print("min:", ms.get_min())                           # smallest so far is 2
print("pop:", ms.pop())                               # remove 2
print("min:", ms.get_min())                           # minimum falls back to 3
print("top:", ms.top())                               # top is 7
print("size:", len(ms))                               # three items left
try:                                                  # test the error case
    MinStack().pop()                                  # popping an empty stack
except IndexError as e:                               # catch the error
    print("Error:", e)                                # show the message
```

**Output:**
```text
min: 2
pop: 2
min: 3
top: 7
size: 3
Error: pop from empty stack
```

**Complexity:** every operation is O(1) time, and the stack uses O(n) space.

---

### Problem 9: Custom Iterator and Generator (Medium)

**Question:** Build a `Countdown` iterator that counts down to 1, and a `Fibonacci` iterable that produces the first `n` Fibonacci numbers and can be looped over **many times**.

**Idea:** An **iterator** has `__iter__` (returns itself) and `__next__` (returns the next value, raises `StopIteration` at the end). An **iterable** only needs `__iter__`, and a **generator** (`yield`) is the easiest way to write it.

**Concepts:** iterator protocol, `yield`, one-shot vs re-usable iteration.

```python
class Countdown:                                      # an ITERATOR: remembers where it is
    def __init__(self, start):                        # constructor
        self.current = start                          # the next number to give out

    def __iter__(self):                               # called by "for" and list()
        return self                                   # an iterator returns itself

    def __next__(self):                               # called to get each value
        if self.current <= 0:                         # nothing left to give
            raise StopIteration                       # tells the loop to stop
        value = self.current                          # remember the value to return
        self.current -= 1                             # move to the next number
        return value                                  # hand out the value

class Fibonacci:                                      # an ITERABLE: creates a fresh iterator each time
    def __init__(self, n):                            # constructor
        self.n = n                                    # how many numbers to produce

    def __iter__(self):                               # generator method: contains yield
        a, b = 0, 1                                   # first two Fibonacci numbers
        for _ in range(self.n):                       # repeat n times
            yield a                                   # hand out a, then pause here until the next request
            a, b = b, a + b                           # move to the next pair

cd = Countdown(5)                                     # create the iterator
print(list(cd))                                       # consumes it completely
cd2 = Countdown(3)                                    # a new iterator
print(list(cd2))                                      # first pass gives the numbers
print(list(cd2))                                      # second pass is empty: the iterator is used up
fib = Fibonacci(8)                                    # an iterable
print(list(fib))                                      # first pass
print(list(fib))                                      # second pass works: a new generator is created each time
print(sum(Fibonacci(10)))                             # sum() can loop over any iterable
for i, f in enumerate(Fibonacci(3)):                  # works with enumerate too
    print(i, f)                                       # index and value
```

**Output:**
```text
[5, 4, 3, 2, 1]
[3, 2, 1]
[]
[0, 1, 1, 2, 3, 5, 8, 13]
[0, 1, 1, 2, 3, 5, 8, 13]
88
0 0
1 1
2 1
```

**Takeaway:** an iterator is **used up** after one pass, while an iterable can start fresh every time. Generators save memory because they produce values **one at a time**.

---

### Problem 10: Library Management System (Medium)

**Question:** Design a small library system. `Book`, `Member` and `Library` classes. A member can borrow at most 2 books. A book can be issued to one member at a time. Support checkout, return, and listing the available titles.

**Concepts:** composition (the library **has** books and members), responsibility split between classes, returning messages instead of printing inside the logic.

```python
class Book:                                           # one book in the library
    def __init__(self, isbn, title):                  # constructor
        self.isbn = isbn                              # unique id of the book
        self.title = title                            # the title
        self.available = True                         # a new book is on the shelf

class Member:                                         # one library member
    def __init__(self, name, limit=2):                # constructor with a default borrowing limit
        self.name = name                              # member's name
        self.limit = limit                            # how many books they can hold at once
        self.borrowed = []                            # books currently with this member

class Library:                                        # the system that connects books and members
    def __init__(self):                               # constructor
        self.books = {}                               # isbn -> Book
        self.members = {}                             # name -> Member

    def add_book(self, book):                         # register a book
        self.books[book.isbn] = book                  # store it by its isbn

    def add_member(self, member):                     # register a member
        self.members[member.name] = member            # store it by name

    def checkout(self, name, isbn):                   # member borrows a book
        member = self.members[name]                   # find the member
        book = self.books[isbn]                       # find the book
        if not book.available:                        # somebody already has it
            return f"'{book.title}' is already issued"    # refuse
        if len(member.borrowed) >= member.limit:      # member is at the limit
            return f"{name} reached the limit of {member.limit} books"    # refuse
        book.available = False                        # the book leaves the shelf
        member.borrowed.append(book)                  # the member now holds it
        return f"{name} borrowed '{book.title}'"      # confirmation message

    def return_book(self, name, isbn):                # member returns a book
        member = self.members[name]                   # find the member
        book = self.books[isbn]                       # find the book
        if book not in member.borrowed:               # they never had it
            return f"{name} does not have '{book.title}'"     # refuse
        member.borrowed.remove(book)                  # the member gives it back
        book.available = True                         # it is on the shelf again
        return f"{name} returned '{book.title}'"      # confirmation message

    def available_titles(self):                       # list titles that can be borrowed
        return sorted(b.title for b in self.books.values() if b.available)   # sorted for neat output

lib = Library()                                       # create the library
lib.add_book(Book(1, "Clean Code"))                   # add three books
lib.add_book(Book(2, "Python Tricks"))                # second book
lib.add_book(Book(3, "Algorithms"))                   # third book
lib.add_member(Member("Asha"))                        # add two members
lib.add_member(Member("Ravi"))                        # second member
print(lib.checkout("Asha", 1))                        # Asha borrows book 1
print(lib.checkout("Ravi", 1))                        # Ravi tries the same book
print(lib.checkout("Asha", 2))                        # Asha borrows a second book
print(lib.checkout("Asha", 3))                        # Asha is at her limit
print(lib.available_titles())                         # only one book is still on the shelf
print(lib.return_book("Asha", 1))                     # Asha returns book 1
print(lib.return_book("Ravi", 2))                     # Ravi never borrowed book 2
print(lib.available_titles())                         # shelf after the return
```

**Output:**
```text
Asha borrowed 'Clean Code'
'Clean Code' is already issued
Asha borrowed 'Python Tricks'
Asha reached the limit of 2 books
['Algorithms']
Asha returned 'Clean Code'
Ravi does not have 'Python Tricks'
['Algorithms', 'Clean Code']
```

**Design notes (say these in an interview):** each class has **one job** (Single Responsibility). `Library` coordinates, `Book` and `Member` only hold state. To extend (fines, due dates, reservations), add new classes or fields without rewriting the old ones.

---

### Problem 11: LRU Cache (Hard)

**Question:** Design a cache with a fixed capacity. `get(key)` returns the value or `-1`. `put(key, value)` inserts or updates. When the cache is full, remove the **Least Recently Used** item. Both operations must be **O(1)**.

**Idea:** Use a **hash map** (key to node) for O(1) lookup and a **doubly linked list** to keep the order of use. The **front** is the most recently used, the **back** is the least recently used. Moving a node or removing the last node takes O(1) because we hold direct pointers.

**Concepts:** a helper class (`Node`), private helper methods, combining two data structures in one class.

```python
class Node:                                           # one entry of the doubly linked list
    def __init__(self, key=0, value=0):               # constructor with defaults (used for dummy nodes)
        self.key = key                                # the key (needed when we evict the node)
        self.value = value                            # the stored value
        self.prev = None                              # pointer to the previous node
        self.next = None                              # pointer to the next node

class LRUCache:                                       # the cache itself
    def __init__(self, capacity):                     # constructor
        self.capacity = capacity                      # maximum number of items
        self.map = {}                                 # key -> Node, for O(1) lookup
        self.head = Node()                            # dummy head: the node after it is the most recent
        self.tail = Node()                            # dummy tail: the node before it is the least recent
        self.head.next = self.tail                    # empty list: head points to tail
        self.tail.prev = self.head                    # and tail points back to head

    def _remove(self, node):                          # unlink a node from the list
        node.prev.next = node.next                    # the previous node skips over this one
        node.next.prev = node.prev                    # the next node skips back over this one

    def _add_front(self, node):                       # put a node right after the head (most recent)
        node.next = self.head.next                    # new node points to the old first node
        node.prev = self.head                         # and back to the head
        self.head.next.prev = node                    # the old first node points back to the new node
        self.head.next = node                         # the head points to the new node

    def get(self, key):                               # read a value
        if key not in self.map:                       # unknown key
            return -1                                 # the required "not found" answer
        node = self.map[key]                          # find its node in O(1)
        self._remove(node)                            # take it out of its position
        self._add_front(node)                         # reading makes it the most recently used
        return node.value                             # return the value

    def put(self, key, value):                        # insert or update
        if key in self.map:                           # the key already exists
            self._remove(self.map[key])               # remove the old node first
        node = Node(key, value)                       # create a fresh node
        self.map[key] = node                          # register it in the map
        self._add_front(node)                         # it is now the most recently used
        if len(self.map) > self.capacity:             # we are over capacity
            lru = self.tail.prev                      # the node before the tail is the least recently used
            self._remove(lru)                         # unlink it from the list
            del self.map[lru.key]                     # and forget it in the map (this is why Node stores key)

cache = LRUCache(2)                                   # capacity 2
cache.put(1, 1)                                       # cache: 1
cache.put(2, 2)                                       # cache: 2, 1 (2 is most recent)
print(cache.get(1))                                   # returns 1, and 1 becomes most recent
cache.put(3, 3)                                       # full: evicts key 2 (least recent)
print(cache.get(2))                                   # 2 was evicted
cache.put(4, 4)                                       # full: evicts key 1
print(cache.get(1))                                   # 1 was evicted
print(cache.get(3))                                   # 3 is still there
print(cache.get(4))                                   # 4 is still there
```

**Output:**
```text
1
-1
-1
3
4
```

**Shorter version with `OrderedDict`** (a dictionary that remembers order and can move keys to the end):

```python
from collections import OrderedDict                   # dict that keeps insertion order and supports reordering

class LRUCacheShort:                                  # same behaviour, much less code
    def __init__(self, capacity):                     # constructor
        self.capacity = capacity                      # maximum number of items
        self.data = OrderedDict()                     # last item = most recently used

    def get(self, key):                               # read a value
        if key not in self.data:                      # unknown key
            return -1                                 # not found
        self.data.move_to_end(key)                    # mark it as most recently used
        return self.data[key]                         # return the value

    def put(self, key, value):                        # insert or update
        if key in self.data:                          # existing key
            self.data.move_to_end(key)                # refresh its position
        self.data[key] = value                        # store the value
        if len(self.data) > self.capacity:            # over capacity
            self.data.popitem(last=False)             # remove the FIRST item = least recently used

c = LRUCacheShort(2)                                  # capacity 2
c.put(1, 1)                                           # cache: 1
c.put(2, 2)                                           # cache: 1, 2
print(c.get(1))                                       # 1, and key 1 moves to the end
c.put(3, 3)                                           # evicts key 2
print(c.get(2))                                       # -1
c.put(4, 4)                                           # evicts key 1
print(c.get(1), c.get(3), c.get(4))                   # -1 3 4
```

**Output:**
```text
1
-1
-1 3 4
```

**Complexity:** `get` and `put` are O(1), space O(capacity). In an interview, write the linked-list version if they ask you to "implement from scratch", and mention `OrderedDict` as the practical shortcut.

---

### Problem 12: Singleton and Factory (Hard)

**Question A (Singleton):** Create a `Logger` class so that **only one** instance can ever exist, no matter how many times you call `Logger()`.

**Question B (Factory):** Create notifications (`Email`, `SMS`, `Push`) through a `NotificationFactory.create(kind)` so that callers never need `if kind == ...` chains.

**Concepts:** `__new__`, class attribute as shared state, abstract base class, registry dictionary.

```python
class Logger:                                         # Singleton: only one object is ever created
    _instance = None                                  # class attribute that will hold the single object

    def __new__(cls):                                 # __new__ CREATES the object (runs before __init__)
        if cls._instance is None:                     # first time: nothing created yet
            cls._instance = super().__new__(cls)      # really create the object
            cls._instance.logs = []                   # give it its message list (done once)
        return cls._instance                          # always return the same object

    def log(self, message):                           # record a message
        self.logs.append(message)                     # add to the shared list

a = Logger()                                          # first call creates the object
b = Logger()                                          # second call returns the SAME object
a.log("first")                                        # log through a
b.log("second")                                       # log through b
print(a is b)                                         # they are one object
print(a.logs)                                         # both messages are in the same list
```

**Output:**
```text
True
['first', 'second']
```

```python
from abc import ABC, abstractmethod                   # abstract class tools

class Notification(ABC):                              # the common interface
    @abstractmethod                                   # every notification must be able to send
    def send(self, message):                          # no body here
        pass                                          # the promise only

class Email(Notification):                            # concrete type 1
    def send(self, message):                          # implement send
        return f"Email: {message}"                    # pretend to send an email

class SMS(Notification):                              # concrete type 2
    def send(self, message):                          # implement send
        return f"SMS: {message}"                      # pretend to send an SMS

class Push(Notification):                             # concrete type 3
    def send(self, message):                          # implement send
        return f"Push: {message}"                     # pretend to send a push notification

class NotificationFactory:                            # creates the right object from a name
    _registry = {"email": Email, "sms": SMS, "push": Push}    # name -> class

    @staticmethod                                     # no object needed to call it
    def create(kind):                                 # build a notification of the requested kind
        cls = NotificationFactory._registry.get(kind.lower())     # look up the class (case-insensitive)
        if cls is None:                               # unknown kind
            raise ValueError(f"Unknown notification type: {kind}")    # reject it
        return cls()                                  # create and return the object

for kind in ["email", "SMS", "push"]:                 # the caller only knows names
    print(NotificationFactory.create(kind).send("Hello"))     # same call, different behaviour
try:                                                  # an unsupported kind
    NotificationFactory.create("fax")                 # not registered
except ValueError as e:                               # catch the error
    print("Error:", e)                                # show the message
```

**Output:**
```text
Email: Hello
SMS: Hello
Push: Hello
Error: Unknown notification type: fax
```

**Takeaway:** to add a new notification type, write one new class and add one line to the registry. No other code changes (Open/Closed principle).

**Singleton warning:** it is global state in disguise. Mention in interviews that it can make testing harder, and that in Python a **module-level object** is often the simplest singleton.

---

### Problem 13: Observer Pattern, News Agency (Hard)

**Question:** A `NewsAgency` publishes headlines. Any number of subscribers can **subscribe** or **unsubscribe**, and all current subscribers must be notified on every new headline.

**Idea:** The publisher keeps a list of observers and calls `update(message)` on each one. The publisher does not know **what** the subscribers do with the message (loose coupling).

**Concepts:** inheritance (`NewsAgency` extends `Subject`), polymorphic callback, loose coupling.

```python
class Subject:                                        # base publisher: manages the subscriber list
    def __init__(self):                               # constructor
        self._observers = []                          # everyone who wants to be notified

    def subscribe(self, observer):                    # add a subscriber
        self._observers.append(observer)              # remember it

    def unsubscribe(self, observer):                  # remove a subscriber
        self._observers.remove(observer)              # forget it

    def notify(self, message):                        # tell everyone
        for observer in self._observers:              # go through the current subscribers
            observer.update(message)                  # each one reacts in its own way

class NewsAgency(Subject):                            # a concrete publisher
    def publish(self, headline):                      # publish a new headline
        self.notify(headline)                         # push it to all subscribers

class Subscriber:                                     # a concrete observer
    def __init__(self, name):                         # constructor
        self.name = name                              # subscriber's name
        self.inbox = []                               # headlines received so far

    def update(self, message):                        # called by the publisher
        self.inbox.append(message)                    # keep the headline
        print(f"{self.name} received: {message}")     # show that it arrived

agency = NewsAgency()                                 # create the publisher
asha = Subscriber("Asha")                             # first subscriber
ravi = Subscriber("Ravi")                             # second subscriber
agency.subscribe(asha)                                # Asha subscribes
agency.subscribe(ravi)                                # Ravi subscribes
agency.publish("Markets up")                          # both are notified
agency.unsubscribe(ravi)                              # Ravi leaves
agency.publish("Rain alert")                          # only Asha is notified
print(len(asha.inbox), len(ravi.inbox))               # inbox sizes
```

**Output:**
```text
Asha received: Markets up
Ravi received: Markets up
Asha received: Rain alert
2 1
```

**Takeaway:** the publisher works with **any** object that has an `update` method (duck typing), so new kinds of subscribers (email sender, logger, dashboard) need no change in the publisher. Real-life examples: YouTube subscriptions, stock price alerts, GUI button click events.

---

### Problem 14: Parking Lot (Hard)

**Question:** Design a parking lot with slots for cars, bikes and trucks. Support `park(vehicle)`, `leave(plate, hours)` which returns the fee (per-hour rates: car 20, bike 10, truck 50), and `free_slots()`.

**Concepts:** `Enum`, inheritance, composition (`ParkingLot` has `ParkingSlot`s), a lookup dictionary for O(1) leave.

```python
from enum import Enum                                 # for the fixed vehicle categories

class VehicleType(Enum):                              # the kinds of vehicles we support
    CAR = 1                                           # car
    BIKE = 2                                          # bike
    TRUCK = 3                                         # truck

class Vehicle:                                        # base class for all vehicles
    vehicle_type = None                               # each child sets its own type

    def __init__(self, plate):                        # constructor
        self.plate = plate                            # the number plate identifies the vehicle

class Car(Vehicle):                                   # a car is a vehicle
    vehicle_type = VehicleType.CAR                    # tell the system what it is

class Bike(Vehicle):                                  # a bike is a vehicle
    vehicle_type = VehicleType.BIKE                   # tell the system what it is

class Truck(Vehicle):                                 # a truck is a vehicle
    vehicle_type = VehicleType.TRUCK                  # tell the system what it is

class ParkingSlot:                                    # one parking space
    def __init__(self, number, vehicle_type):         # constructor
        self.number = number                          # slot number
        self.vehicle_type = vehicle_type              # which kind of vehicle fits here
        self.vehicle = None                           # nobody is parked yet

class ParkingLot:                                     # the whole system
    RATES = {VehicleType.CAR: 20, VehicleType.BIKE: 10, VehicleType.TRUCK: 50}    # price per hour

    def __init__(self, layout):                       # layout: {VehicleType: number of slots}
        self.slots = []                               # all slots in order
        number = 1                                    # slot numbers start from 1
        for vtype, count in layout.items():           # go through each category
            for _ in range(count):                    # create that many slots
                self.slots.append(ParkingSlot(number, vtype))     # add one slot
                number += 1                           # next slot gets the next number
        self.where = {}                               # plate -> slot, so leaving is O(1)

    def park(self, vehicle):                          # try to park a vehicle
        if vehicle.plate in self.where:               # already inside
            return f"{vehicle.plate} is already parked"      # refuse duplicates
        for slot in self.slots:                       # look for the first suitable free slot
            if slot.vehicle_type == vehicle.vehicle_type and slot.vehicle is None:    # right type and empty
                slot.vehicle = vehicle                # park here
                self.where[vehicle.plate] = slot      # remember where it is
                return f"{vehicle.plate} parked at slot {slot.number}"    # confirmation
        return f"No free slot for {vehicle.plate}"    # nothing available

    def leave(self, plate, hours):                    # vehicle exits, we compute the fee
        slot = self.where.pop(plate, None)            # remove from the lookup (None if unknown)
        if slot is None:                              # it was never parked here
            return f"{plate} not found"               # report it
        fee = self.RATES[slot.vehicle_type] * hours   # hourly rate times hours
        slot.vehicle = None                           # the slot becomes free
        return f"{plate} left slot {slot.number}, fee = {fee}"    # receipt text

    def free_slots(self):                             # count free slots per category
        return {vt.name: sum(1 for s in self.slots if s.vehicle_type == vt and s.vehicle is None)
                for vt in VehicleType}                # dictionary: "CAR" -> free count

lot = ParkingLot({VehicleType.CAR: 2, VehicleType.BIKE: 1, VehicleType.TRUCK: 1})    # 4 slots in total
print(lot.park(Car("MH01")))                          # takes slot 1
print(lot.park(Car("MH02")))                          # takes slot 2
print(lot.park(Car("MH03")))                          # no car slot left
print(lot.park(Bike("MH09")))                         # takes the bike slot 3
print(lot.park(Car("MH01")))                          # duplicate plate
print(lot.leave("MH01", 3))                           # 3 hours at 20 per hour
print(lot.park(Car("MH03")))                          # slot 1 is free again
print(lot.leave("XX99", 1))                           # unknown vehicle
print(lot.free_slots())                               # free slots per category
```

**Output:**
```text
MH01 parked at slot 1
MH02 parked at slot 2
No free slot for MH03
MH09 parked at slot 3
MH01 is already parked
MH01 left slot 1, fee = 60
MH03 parked at slot 1
XX99 not found
{'CAR': 0, 'BIKE': 0, 'TRUCK': 1}
```

**How to extend in an interview:** add multiple floors (a `Floor` class holding slots), entry/exit **tickets** with timestamps, different pricing **strategies**, and a nearest-slot allocation rule. Each is a new class or a small change, which shows the design is open to extension.

---

### Problem 15: Tic-Tac-Toe (Hard)

**Question:** Design an `n x n` Tic-Tac-Toe game with alternating players `X` and `O`. Reject invalid moves, detect the winner in **O(1)** per move, and print the board.

**Idea for O(1) win check:** keep a running **count per row, per column, and for the two diagonals**. Player X adds `+1`, player O adds `-1`. A line is complete when its count reaches `n` or `-n`.

**Concepts:** custom exception, encapsulating game state, `__str__`, turn management.

```python
class InvalidMoveError(Exception):                    # our own error for illegal moves
    pass                                              # no extra code needed

class TicTacToe:                                      # the game
    def __init__(self, n=3):                          # constructor, standard board is 3 x 3
        self.n = n                                    # board size
        self.grid = [["." for _ in range(n)] for _ in range(n)]    # empty board (rows built separately)
        self.rows = [0] * n                           # running score of every row
        self.cols = [0] * n                           # running score of every column
        self.diag = 0                                 # running score of the main diagonal
        self.anti = 0                                 # running score of the anti-diagonal
        self.turn = "X"                               # X starts
        self.winner = None                            # nobody has won yet
        self.moves = 0                                # number of moves played

    def play(self, row, col):                         # current player marks a cell
        if self.winner or self.moves == self.n * self.n:          # game already finished
            raise InvalidMoveError("Game is over")    # no more moves allowed
        if not (0 <= row < self.n and 0 <= col < self.n):         # coordinates outside the board
            raise InvalidMoveError("Outside the board")           # reject
        if self.grid[row][col] != ".":                # the cell is not empty
            raise InvalidMoveError("Cell already taken")          # reject
        self.grid[row][col] = self.turn               # place the mark
        self.moves += 1                               # count the move
        add = 1 if self.turn == "X" else -1           # X adds 1, O subtracts 1
        self.rows[row] += add                         # update the row score
        self.cols[col] += add                         # update the column score
        if row == col:                                # the cell lies on the main diagonal
            self.diag += add                          # update its score
        if row + col == self.n - 1:                   # the cell lies on the anti-diagonal
            self.anti += add                          # update its score
        scores = (self.rows[row], self.cols[col], self.diag, self.anti)    # the four lines through this cell
        if any(abs(s) == self.n for s in scores):     # one of them is completely filled by one player
            self.winner = self.turn                   # the current player wins
        else:                                         # no winner yet
            self.turn = "O" if self.turn == "X" else "X"     # switch to the other player
        return self.winner                            # None until someone wins

    def __str__(self):                                # text version of the board
        return "\n".join(" ".join(row) for row in self.grid)     # one line per row

g = TicTacToe()                                       # start a new game
for r, c in [(0, 0), (0, 1), (1, 1), (0, 2), (2, 2)]:           # five moves: X, O, X, O, X
    print(g.play(r, c))                               # prints None until someone wins
print(g)                                              # show the board
print("Winner:", g.winner)                            # X completed the main diagonal
try:                                                  # moves after the end are rejected
    g.play(2, 0)                                      # game is over
except InvalidMoveError as e:                         # catch our error
    print("Error:", e)                                # show the message

g2 = TicTacToe()                                      # a fresh game for the other errors
g2.play(0, 0)                                         # X plays the corner
try:                                                  # O tries the same cell
    g2.play(0, 0)                                     # taken
except InvalidMoveError as e:                         # catch the error
    print("Error:", e)                                # show the message
try:                                                  # O tries an impossible cell
    g2.play(5, 5)                                     # outside the board
except InvalidMoveError as e:                         # catch the error
    print("Error:", e)                                # show the message
print(g2.turn)                                        # failed moves do not change whose turn it is
```

**Output:**
```text
None
None
None
None
X
X O O
. X .
. . X
Winner: X
Error: Game is over
Error: Cell already taken
Error: Outside the board
O
```

**Complexity:** each move is **O(1)** time (instead of scanning the board in O(n)). Space is O(n squared) for the grid, and O(n) for the counters.

**Extensions to discuss:** an AI player class (strategy pattern), undo/redo (store moves in a stack), a `Player` class with names and scores, and a `Board` class separated from the `Game` class (Single Responsibility).

---

## 14. Cheat sheet and extra practice

### 14.1 Syntax cheat sheet

```text
class Child(Parent):                 inheritance
    cls_attr = 0                     class attribute (shared)
    def __init__(self, x):           constructor
        super().__init__()           call the parent constructor
        self.x = x                   instance attribute
        self._p = 1                  protected by convention
        self.__q = 2                 private (name mangled)
    @property                        getter
    def val(self): ...
    @val.setter                      setter
    def val(self, v): ...
    @classmethod                     gets cls
    def make(cls, ...): ...
    @staticmethod                    gets nothing
    def helper(...): ...
    def __str__(self): ...           print(obj)
    def __repr__(self): ...          debugging text
    def __eq__(self, o): ...         ==
    def __lt__(self, o): ...         <
    def __len__(self): ...           len(obj)
    def __iter__(self): ...          for x in obj
    def __getitem__(self, i): ...    obj[i]
    def __call__(self, ...): ...     obj(...)
```

### 14.2 What to say about the 4 pillars (interview script)

- **Encapsulation:** "I keep data private (`__balance`) and expose it through methods or properties that validate input."
- **Abstraction:** "I define what a class must do using an abstract base class, and hide how each subclass does it."
- **Inheritance:** "A child reuses and extends a parent. I use `super()` and prefer composition when the relationship is has-a."
- **Polymorphism:** "The same method call behaves differently depending on the object, through overriding and duck typing."

### 14.3 Extra practice problems

| Level | Problem | Idea |
|---|---|---|
| Easy | `Circle` with radius property and area | property validation |
| Easy | `Counter` that counts objects created | class attribute |
| Easy | `Temperature` converter with `from_fahrenheit` | classmethod |
| Medium | `Matrix` class with `+`, `*` and `__str__` | operator overloading |
| Medium | `Stack` and `Queue` with a common abstract base | abstraction |
| Medium | `Polynomial` class with `__add__` and `evaluate(x)` | operator overloading |
| Medium | `ShoppingCart` with discount strategies | strategy pattern |
| Medium | `Vector` with `__eq__`, `__hash__`, `__abs__` | hashable objects |
| Hard | Vending machine with states (idle, has money, dispensing) | state pattern |
| Hard | ATM with cards, PIN, accounts | multiple cooperating classes |
| Hard | Elevator controller | state + scheduling |
| Hard | Library with fines, due dates and reservations | extend Problem 10 |
| Hard | Chess pieces (abstract `Piece`, moves per subclass) | polymorphism |
| Hard | Text editor with undo/redo | command pattern with two stacks |

### 14.4 Readiness check

You are ready for OOP questions in the coding round if you can, without help:

- Write a class with `__init__`, methods, `__str__`, and explain what `self` is.
- Explain the 4 pillars with one small example each.
- Write inheritance with `super()` and override a method.
- Create an abstract base class and explain why it cannot be instantiated.
- Explain `@classmethod` vs `@staticmethod` vs instance methods.
- Overload `+`, `==`, `<` and `len()` for your own class.
- Explain composition vs inheritance and when you would choose each.
- Design a small system (library, parking lot, tic-tac-toe) by identifying classes, their attributes, and their relationships.

### 14.5 How to approach a "design a system" question (LLD)

1. **Clarify requirements:** what must it do, what are the limits?
2. **Find the nouns:** they become classes (`Book`, `Member`, `Library`).
3. **Find the verbs:** they become methods (`checkout`, `return_book`).
4. **Decide relationships:** is-a (inheritance) or has-a (composition)?
5. **Keep each class focused:** one responsibility each.
6. **Handle errors:** raise clear exceptions or return clear messages.
7. **Explain extension points:** how a new requirement can be added without breaking old code.

---

**Next topic suggestion:** Graphs (BFS, DFS, shortest paths), or Sorting and Binary Search. Tell me which one you want next.
