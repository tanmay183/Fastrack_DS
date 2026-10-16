# Topic 2: Python Basics for DSA

**Data Types, Statements, Loops and Functions, with code, output and explanation**

> **Goal of this note:** before you start Math and the real DSA topics, your Python must be so smooth that syntax never slows you down. In a coding round, every minute you spend remembering how `split()` or `enumerate()` works is a minute lost from thinking about the actual problem.

How to use this note: type every example yourself (do not just read), predict the output **before** looking at it, and then compare.

---

## Table of Contents

1. [Python basics: variables, types, print](#1-python-basics-variables-types-print)
2. [Numbers: int, float, bool](#2-numbers-int-float-bool)
3. [Strings](#3-strings)
4. [Lists](#4-lists)
5. [Tuples](#5-tuples)
6. [Sets](#6-sets)
7. [Dictionaries (and Counter, defaultdict, deque)](#7-dictionaries-and-friends)
8. [None, truthiness, mutable vs immutable](#8-none-truthiness-mutable-vs-immutable)
9. [Operators and statements](#9-operators-and-statements)
10. [Conditional statements: if / elif / else](#10-conditional-statements)
11. [Loops](#11-loops)
12. [Functions](#12-functions)
13. [Input and output in coding rounds](#13-input-and-output-in-coding-rounds)
14. [Mini programs (put it all together)](#14-mini-programs-put-it-all-together)
15. [Common errors and what they mean](#15-common-errors-and-what-they-mean)
16. [Practice exercises](#16-practice-exercises)
17. [Quick revision sheet](#17-quick-revision-sheet)

---

## 1. Python basics: variables, types, print

### 1.1 Variables

A **variable** is a name that points to a value. You do not declare a type. Python figures it out.

```python
x = 10
y = 3.5
name = "Asha"
is_ready = True

print(x, type(x))
print(y, type(y))
print(name, type(name))
print(is_ready, type(is_ready))
```

**Output:**
```text
10 <class 'int'>
3.5 <class 'float'>
Asha <class 'str'>
True <class 'bool'>
```

**Explanation:** `type()` tells you the type of a value. Python has **dynamic typing**, so the same name can point to different types at different times:

```python
a = 5
print(type(a))
a = "hello"
print(type(a))
```

**Output:**
```text
<class 'int'>
<class 'str'>
```

### 1.2 Multiple assignment and swapping

```python
a, b, c = 1, 2, 3
a, b = b, a            # swap without a temp variable
print(a, b, c)

x = y = 0              # both names point to 0
print(x, y)
```

**Output:**
```text
2 1 3
0 0
```

**Explanation:** `a, b = b, a` is the Pythonic swap. You will use it in sorting, two pointers and array reversal.

### 1.3 Naming rules

- Use letters, digits and `_`. A name cannot start with a digit.
- Names are case sensitive (`count` and `Count` are different).
- Do not use keywords (`for`, `if`, `class`...) or shadow built-ins (`list`, `max`, `sum`, `str`, `dict`). Writing `list = [1, 2]` breaks `list()` later in your code.
- Style: `snake_case` for variables and functions, `UPPER_CASE` for constants.

### 1.4 Comments and indentation

Python uses **indentation** (4 spaces) instead of `{ }` to define blocks. Wrong indentation gives `IndentationError`.

```python
# This is a comment
x = 7
if x > 5:
    print("big")        # inside the if block (4 spaces)
    print("still inside")
print("outside")        # not inside the if block
```

**Output:**
```text
big
still inside
outside
```

### 1.5 print()

```python
print("a", "b", "c", sep="-")     # separator between values
print("Hello", end=" ")           # end replaces the newline
print("World")
print(1, 2, 3)
```

**Output:**
```text
a-b-c
Hello World
1 2 3
```

---

## 2. Numbers: int, float, bool

### 2.1 Arithmetic operators

```python
a, b = 17, 5
print("a + b =", a + b)
print("a - b =", a - b)
print("a * b =", a * b)
print("a / b =", a / b)     # true division, always float
print("a // b =", a // b)   # floor division
print("a % b =", a % b)     # remainder (modulo)
print("a ** b =", a ** b)   # power
```

**Output:**
```text
a + b = 22
a - b = 12
a * b = 85
a / b = 3.4
a // b = 3
a % b = 2
a ** b = 1419857
```

**Explanation:**

| Operator | Meaning | Where you will use it |
|---|---|---|
| `/` | normal division, always gives a float | averages |
| `//` | floor division (rounds **down**) | mid = (lo + hi) // 2, digits |
| `%` | remainder | even/odd, last digit, circular arrays |
| `**` | power | squares, 2**k |

### 2.2 Floor division with negatives (a classic trap)

```python
print(-7 // 2)
print(-7 % 2)
print(int(-7 / 2))
print(divmod(17, 5))
```

**Output:**
```text
-4
1
-3
(3, 2)
```

**Explanation:** `//` rounds toward **negative infinity**, so `-7 // 2` is `-4`, not `-3`. `int(-7 / 2)` truncates toward zero and gives `-3`. In Python, `a % b` always has the same sign as `b`, so `-7 % 2` is `1`. `divmod(a, b)` gives `(a // b, a % b)` in one call.

### 2.3 Big integers and float precision

```python
print(2 ** 100)                         # Python ints never overflow
print(0.1 + 0.2)
print(0.1 + 0.2 == 0.3)
print(abs((0.1 + 0.2) - 0.3) < 1e-9)    # safe way to compare floats
print(10 / 2)
print(type(10 / 2))
print(10 // 2)
```

**Output:**
```text
1267650600228229401496703205376
0.30000000000000004
False
True
5.0
<class 'float'>
5
```

**Explanation:**

- Python integers have **no overflow**. This is a big advantage over C++/Java in problems with huge numbers.
- Floats are stored in binary, so some decimals are slightly off. **Never compare floats with `==`**. Compare the difference with a tiny number.
- Prefer integer arithmetic (`//`, `%`) whenever possible.

### 2.4 Infinity (very useful in DSA)

```python
INF = float("inf")
print(INF > 10 ** 18)
print(min(INF, 5))
print(-INF < -10 ** 18)
```

**Output:**
```text
True
5
True
```

**Explanation:** Use `float("inf")` and `float("-inf")` as starting values for "minimum so far" and "maximum so far".

### 2.5 Type conversion and rounding

```python
print(int("42") + 1)
print(int(3.9))                  # cuts the decimal part
print(float("2.5") * 2)
print(str(100) + "!")
print(round(2.5), round(3.5), round(2.567, 2))
print(bool(0), bool(""), bool([]), bool(5))
print(bin(10), int("1010", 2))
```

**Output:**
```text
43
3
5.0
100!
2 4 2.57
False False False True
0b1010 10
```

**Explanation:**

- `int(3.9)` is `3` (truncation, not rounding).
- `round(2.5)` is `2` and `round(3.5)` is `4` because Python rounds exact halves to the **nearest even** number ("banker's rounding"). Do not use `round` when you need "round half up".
- `bin(10)` gives the binary string `'0b1010'`. `int("1010", 2)` converts a binary string back.

### 2.6 bool is a number too

```python
print(True + True)
print(sum([True, False, True, True]))
nums = [3, 8, 1, 6]
print(sum(x > 2 for x in nums))     # counts how many are greater than 2
```

**Output:**
```text
2
3
3
```

**Explanation:** `True` is `1` and `False` is `0`. Summing booleans is a neat way to count things that satisfy a condition.

---

## 3. Strings

A string is a sequence of characters. Strings are **immutable** (you cannot change a character in place).

### 3.1 Indexing and slicing

```python
s = "Python"
print(s[0], s[-1])      # first and last
print(s[1:4])           # index 1 up to (not including) 4
print(s[:2])
print(s[2:])
print(s[::-1])          # reverse
print(s[::2])           # every second character
print(len(s))
```

**Output:**
```text
P n
yth
Py
thon
nohtyP
Pto
6
```

**Explanation:**

```
 P   y   t   h   o   n
 0   1   2   3   4   5     (positive index)
-6  -5  -4  -3  -2  -1     (negative index)
```

Slice syntax is `s[start:stop:step]`. `start` is included, `stop` is **excluded**. A negative step walks backwards, so `[::-1]` reverses. Slicing never raises `IndexError` even when the range is too large, but a single bad index `s[10]` does.

### 3.2 Strings are immutable

```python
s = "hello"
try:
    s[0] = "H"
except TypeError as e:
    print("Error:", e)

s = "H" + s[1:]         # build a NEW string
print(s)
```

**Output:**
```text
Error: 'str' object does not support item assignment
Hello
```

**Explanation:** To "change" a string, create a new one. For many changes, convert to a list, edit the list, and use `"".join(...)`.

### 3.3 Useful string methods

```python
word = "Hello, World"
print(word.upper())
print(word.lower())
print(word.replace("World", "Python"))
print(word.split(", "))
print(word.find("World"))       # index of first match, -1 if absent
print(word.find("xyz"))
print(word.count("l"))
print(word.startswith("Hello"), word.endswith("d"))
print("-".join(["a", "b", "c"]))
print("123".isdigit(), "abc".isalpha(), "ab1".isalnum())
print("  padded  ".strip() + "|")
print("a b  c".split())          # split on any whitespace
```

**Output:**
```text
HELLO, WORLD
hello, world
Hello, Python
['Hello', 'World']
7
-1
3
True True
a-b-c
True True True
padded|
['a', 'b', 'c']
```

**Explanation:** Methods like `upper()` return a **new** string. They never change the original. `split()` with no argument splits on any run of spaces, which is the standard way to read numbers from input.

### 3.4 f-strings (formatted output)

```python
name = "Asha"
score = 92.456
print(f"{name} scored {score:.2f}")
print(f"{5:03d}")
print(f"{1234567:,}")
print(f"{name!r}")
```

**Output:**
```text
Asha scored 92.46
005
1,234,567
'Asha'
```

### 3.5 Characters and ASCII: `ord` and `chr`

```python
print(ord("a"), ord("A"))
print(chr(98))
print(ord("d") - ord("a"))      # position of 'd' in the alphabet (0-based)
print(chr(ord("a") + 2))
```

**Output:**
```text
97 65
b
3
c
```

**Explanation:** `ord` converts a character to its number, `chr` does the reverse. This is the base of Caesar cipher, anagram counting arrays (`count[ord(ch) - ord("a")]`), and many string problems.

### 3.6 Looping over strings: count vowels, check palindrome

```python
text = "education"
vowels = "aeiou"
count = 0
for ch in text:
    if ch in vowels:
        count += 1
print("Vowels:", count)

word = "Madam"
w = word.lower()
print(w == w[::-1])
```

**Output:**
```text
Vowels: 5
True
```

**Time and space:** Counting vowels is O(n) time, O(1) space. The palindrome check builds a reversed copy, so it is O(n) time and O(n) space.

---

## 4. Lists

A list is an **ordered, changeable (mutable)** collection. It is the most used structure in coding rounds (it works as an array, stack and more).

### 4.1 Create, index, slice, update

```python
nums = [10, 20, 30, 40, 50]
print(nums[0], nums[-1])
print(nums[1:4])
print(nums[::-1])
print(len(nums))

nums[2] = 99
print(nums)
```

**Output:**
```text
10 50
[20, 30, 40]
[50, 40, 30, 20, 10]
5
[10, 20, 99, 40, 50]
```

### 4.2 Adding and removing

```python
lst = [3, 1, 2]
lst.append(4)            # add at the end
print(lst)
lst.insert(0, 9)         # add at an index
print(lst)
lst.extend([7, 8])       # add many items
print(lst)
print(lst.pop())         # remove and return the last item
print(lst.pop(0))        # remove and return item at index 0
lst.remove(1)            # remove the first occurrence of VALUE 1
print(lst)
print(lst.index(4), lst.count(2))
```

**Output:**
```text
[3, 1, 2, 4]
[9, 3, 1, 2, 4]
[9, 3, 1, 2, 4, 7, 8]
8
9
[3, 2, 4, 7]
2 1
```

**Explanation (with cost):**

| Method | What it does | Time |
|---|---|---|
| `append(x)` | add at end | O(1) amortized |
| `pop()` | remove last | O(1) |
| `insert(i, x)` | add at index | O(n) |
| `pop(i)` / `pop(0)` | remove at index | O(n) |
| `remove(x)` | remove first value x | O(n) |
| `extend(other)` | add many | O(len(other)) |
| `index(x)`, `count(x)`, `x in lst` | search | O(n) |

### 4.3 Sorting

```python
a = [5, 2, 9, 1]
a.sort()                          # in place, returns None
print(a)
a.sort(reverse=True)
print(a)

b = sorted([5, 2, 9, 1])          # returns a NEW list, original unchanged
print(b)

words = ["banana", "kiwi", "apple"]
words.sort(key=len)               # sort by a custom key
print(words)
print(sorted("hello"))
```

**Output:**
```text
[1, 2, 5, 9]
[9, 5, 2, 1]
[1, 2, 5, 9]
['kiwi', 'apple', 'banana']
['e', 'h', 'l', 'l', 'o']
```

**Explanation:** `list.sort()` changes the list and returns `None` (a common bug is `a = a.sort()`, which makes `a` equal to `None`). `sorted(x)` works on any iterable and returns a new list. Both are O(n log n) and **stable** (equal items keep their original order).

### 4.4 Copy vs alias (very important)

```python
a = [1, 2, 3]
b = a              # NOT a copy: both names point to the same list
b.append(4)
print(a)

c = a.copy()       # real (shallow) copy; a[:] and list(a) also work
c.append(5)
print(a)
print(c)
```

**Output:**
```text
[1, 2, 3, 4]
[1, 2, 3, 4]
[1, 2, 3, 4, 5]
```

### 4.5 The 2D list trap

```python
wrong = [[0] * 3] * 3            # 3 references to the SAME inner list
wrong[0][0] = 1
print(wrong)

right = [[0] * 3 for _ in range(3)]   # 3 separate inner lists
right[0][0] = 1
print(right)
```

**Output:**
```text
[[1, 0, 0], [1, 0, 0], [1, 0, 0]]
[[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

**Explanation:** Always build grids and DP tables with a list comprehension. This one mistake has ruined many DP solutions.

### 4.6 List comprehension

```python
squares = [x * x for x in range(1, 6)]
evens = [x for x in range(10) if x % 2 == 0]
pairs = [(i, j) for i in range(2) for j in range(2)]
print(squares)
print(evens)
print(pairs)
```

**Output:**
```text
[1, 4, 9, 16, 25]
[0, 2, 4, 6, 8]
[(0, 0), (0, 1), (1, 0), (1, 1)]
```

**Pattern:** `[expression for item in iterable if condition]`

### 4.7 Handy built-ins on lists

```python
nums = [4, 9, 2, 7]
print(len(nums), sum(nums), min(nums), max(nums))
print(9 in nums, 5 in nums)
first, *rest = nums
print(first, rest)
print(nums + [1, 1])
print([0] * 4)
```

**Output:**
```text
4 22 2 9
True False
4 [9, 2, 7]
[4, 9, 2, 7, 1, 1]
[0, 0, 0, 0]
```

---

## 5. Tuples

A tuple is an **ordered, unchangeable (immutable)** collection.

```python
t = (1, 2, 3)
print(t[0], t[-1], len(t))
try:
    t[0] = 5
except TypeError as e:
    print("Error:", e)

point = (3, 4)
x, y = point               # unpacking
print(x, y)

single = (5,)              # the comma makes it a tuple
print(type(single), type((5)))

def min_max(arr):
    return min(arr), max(arr)    # returns a tuple

lo, hi = min_max([4, 1, 9])
print(lo, hi)
```

**Output:**
```text
1 3 3
Error: 'tuple' object does not support item assignment
3 4
<class 'tuple'> <class 'int'>
1 9
```

**Why tuples matter in DSA:**

- Tuples are **hashable**, so they can be set elements and dictionary keys. Lists cannot.
- Great for coordinates: `visited.add((row, col))`.
- Functions that "return two values" really return a tuple.

```python
visited = set()
visited.add((0, 1))
print((0, 1) in visited, (1, 0) in visited)
```

**Output:**
```text
True False
```

---

## 6. Sets

A set is an **unordered collection of unique items** with very fast lookup (average O(1)).

```python
s = {1, 2, 3, 3, 2}
print(s)
s.add(4)
s.remove(1)
print(s)
print(3 in s)
s.discard(100)             # no error if missing (remove would raise KeyError)
print(len(s))
```

**Output:**
```text
{1, 2, 3}
{2, 3, 4}
True
3
```

### 6.1 Set operations

```python
a = {1, 2, 3, 4}
b = {3, 4, 5}
print(a | b)       # union
print(a & b)       # intersection
print(a - b)       # difference
print(a ^ b)       # in one but not both
print({1, 2} <= a) # subset
```

**Output:**
```text
{1, 2, 3, 4, 5}
{3, 4}
{1, 2}
{1, 2, 5}
True
```

### 6.2 Remove duplicates and detect them

```python
nums = [1, 2, 2, 3, 3, 3]
print(len(nums) != len(set(nums)))      # contains duplicate?
print(sorted(set(nums)))
print(type({}), type(set()))            # {} is a dict! use set() for empty set
```

**Output:**
```text
True
[1, 2, 3]
<class 'dict'> <class 'set'>
```

**Rules to remember:** sets have **no order** and **no indexing** (`s[0]` fails). Elements must be hashable (numbers, strings, tuples are fine; lists are not).

---

## 7. Dictionaries (and friends)

A dictionary stores **key: value** pairs. Lookup, insert and delete are average O(1). Since Python 3.7 it remembers insertion order.

### 7.1 Basics

```python
student = {"name": "Asha", "age": 21}
print(student["name"])
student["city"] = "Pune"                 # add
student["age"] = 22                      # update
print(student)
print(student.get("marks"))              # None, no error
print(student.get("marks", 0))           # default value
print("name" in student)                 # checks keys
del student["age"]
print(list(student.keys()))
print(list(student.values()))

for k, v in student.items():
    print(k, "->", v)
```

**Output:**
```text
Asha
{'name': 'Asha', 'age': 22, 'city': 'Pune'}
None
0
True
['name', 'city']
['Asha', 'Pune']
name -> Asha
city -> Pune
```

**Explanation:** `student["marks"]` raises `KeyError` if the key is missing. `student.get("marks")` is the safe version.

### 7.2 The most important pattern: counting frequency

```python
word = "banana"
freq = {}
for ch in word:
    freq[ch] = freq.get(ch, 0) + 1
print(freq)
```

**Output:**
```text
{'b': 1, 'a': 3, 'n': 2}
```

**Dry run:** `b` is not present so get returns 0, giving 1. Then `a` becomes 1, `n` becomes 1, `a` becomes 2, `n` becomes 2, `a` becomes 3.

### 7.3 `Counter` (the shortcut)

```python
from collections import Counter

print(Counter("banana"))
c = Counter([1, 1, 2, 3, 3, 3])
print(c.most_common(1))
print(c[3], c[99])                 # missing key gives 0, not an error
print(Counter("listen") == Counter("silent"))    # anagram check
```

**Output:**
```text
Counter({'a': 3, 'n': 2, 'b': 1})
[(3, 3)]
3 0
True
```

### 7.4 `defaultdict` (no more "if key not in dict")

```python
from collections import defaultdict

groups = defaultdict(list)
for w in ["apple", "avocado", "banana", "blueberry", "cherry"]:
    groups[w[0]].append(w)
print(dict(groups))

count = defaultdict(int)
for x in [1, 2, 1, 1]:
    count[x] += 1
print(dict(count))
```

**Output:**
```text
{'a': ['apple', 'avocado'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}
{1: 3, 2: 1}
```

**Explanation:** `defaultdict(list)` automatically creates an empty list for a missing key. This is how adjacency lists for graphs are built.

### 7.5 `deque` (fast queue)

```python
from collections import deque

dq = deque([1, 2, 3])
dq.append(4)
dq.appendleft(0)
print(dq)
print(dq.popleft())
print(dq.pop())
print(list(dq))
```

**Output:**
```text
deque([0, 1, 2, 3, 4])
0
4
[1, 2, 3]
```

**Explanation:** `popleft()` is O(1), while `list.pop(0)` is O(n). Always use `deque` for queues and BFS.

### 7.6 Which structure to choose?

| Need | Use |
|---|---|
| ordered items, change them | `list` |
| fixed group, dictionary key or set item | `tuple` |
| uniqueness, fast "is it present?" | `set` |
| key to value mapping, counting | `dict` / `Counter` |
| queue (both ends) | `deque` |

---

## 8. None, truthiness, mutable vs immutable

### 8.1 `None`

`None` means "no value" (like `null`). A function with no `return` gives `None`. Check it with `is None`.

### 8.2 Truthiness

In `if` and `while`, these are treated as **False**: `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`, `None`, `False`. Everything else is **True**.

```python
for v in [0, "", [], None, "hi", [0]]:
    print(repr(v), bool(v))

arr = []
if not arr:
    print("empty list")
```

**Output:**
```text
0 False
'' False
[] False
None False
'hi' True
[0] True
empty list
```

**Explanation:** `if not arr:` is the Pythonic way to check for an empty list. Note `[0]` is True because the list is not empty.

### 8.3 Mutable vs immutable

| Immutable (cannot change in place) | Mutable (can change in place) |
|---|---|
| `int`, `float`, `bool`, `str`, `tuple`, `frozenset`, `None` | `list`, `dict`, `set`, `deque` |

Why it matters: immutables can be dictionary keys and set items. Mutables passed to a function can be **modified by the function** (see Section 12).

---

## 9. Operators and statements

### 9.1 What is a "statement"?

A **statement** is one instruction that Python executes. The kinds you need:

| Kind | Example |
|---|---|
| Expression statement | `print(x)`, `arr.append(5)` |
| Assignment | `x = 10`, `a, b = b, a`, `total += x` |
| Conditional | `if / elif / else` |
| Loop | `for`, `while` |
| Jump | `break`, `continue`, `return`, `pass` |
| Definition | `def f(): ...` |
| Import | `import math`, `from collections import deque` |
| Exception handling | `try / except / finally` |

### 9.2 Comparison and logical operators

```python
x = 7
print(x == 7, x != 7, x > 5, x <= 7)
print(1 < x < 10)                    # chained comparison
print(x > 5 and x < 10)
print(x < 5 or x == 7)
print(not x == 7)
```

**Output:**
```text
True False True True
True
True
True
False
```

**Short-circuiting:** in `A and B`, if `A` is False then `B` is never evaluated. In `A or B`, if `A` is True then `B` is never evaluated. This is why `if arr and arr[0] == 5:` is safe on an empty list.

```python
name = ""
print(name or "Guest")               # first truthy value
print(0 and 5, 3 and 5)
```

**Output:**
```text
Guest
0 5
```

### 9.3 `==` versus `is`, and membership

```python
a = [1, 2]
b = [1, 2]
print(a == b)         # same VALUE?
print(a is b)         # same OBJECT in memory?
x = None
print(x is None)
print(3 in [1, 2, 3], "z" not in "abc")
```

**Output:**
```text
True
False
True
True True
```

**Rule:** use `==` to compare values, and use `is` only for `None` (`if x is None`).

### 9.4 Assignment operators

```python
n = 10
n += 5
n -= 3
n *= 2
n //= 5
n %= 3
print(n)
n = 2
n **= 5
print(n)
```

**Output:**
```text
1
32
```

**Trace:** 10 → 15 → 12 → 24 → 4 (24 // 5) → 1 (4 % 3). Python has no `n++`. Write `n += 1`.

### 9.5 Bitwise operators (appear in many DSA problems)

```python
print(5 & 3)      # AND   101 & 011 = 001
print(5 | 3)      # OR    101 | 011 = 111
print(5 ^ 3)      # XOR   101 ^ 011 = 110
print(5 << 1)     # shift left = multiply by 2
print(5 >> 1)     # shift right = floor divide by 2
print(~5)         # NOT = -(n + 1)

n = 12
print("odd" if n & 1 else "even")        # last bit tells odd/even
print(7 ^ 7, 9 ^ 0)                      # x ^ x = 0, x ^ 0 = x
```

**Output:**
```text
1
7
6
10
2
-6
even
0 9
```

**Why XOR is special:** `x ^ x = 0` and `x ^ 0 = x`. So XOR-ing all numbers of an array cancels pairs and leaves the single unpaired one (classic "single number" problem).

### 9.6 Operator precedence (high to low)

```
**   >   unary - ~   >   *  /  //  %   >   +  -   >   << >>   >   &   >   ^   >   |
 >   comparisons (== < > in is)   >   not   >   and   >   or
```

When unsure, **use brackets**. Example: `2 + 3 * 4` is `14`, and `(2 + 3) * 4` is `20`.

### 9.7 `pass`, and basic error handling

```python
def todo():
    pass              # placeholder, does nothing

def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return "Cannot divide by zero"

print(safe_divide(10, 2))
print(safe_divide(5, 0))
```

**Output:**
```text
5.0
Cannot divide by zero
```

---

## 10. Conditional statements

### 10.1 if / elif / else

```python
marks = 78

if marks >= 90:
    grade = "A"
elif marks >= 75:
    grade = "B"
elif marks >= 50:
    grade = "C"
else:
    grade = "F"

print("Grade:", grade)
```

**Output:**
```text
Grade: B
```

**Explanation:** Conditions are checked **top to bottom**, and only the first true block runs. So `marks >= 75` does not need `marks < 90` because that case was already caught above.

### 10.2 One-line (ternary) if

```python
n = 7
label = "even" if n % 2 == 0 else "odd"
print(label)
print(max(3, 8), "big" if 8 > 5 else "small")
```

**Output:**
```text
odd
8 big
```

### 10.3 Nested conditions and combining checks

```python
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("Allowed")
    else:
        print("Need ID")
else:
    print("Too young")

# the same, cleaner
if age >= 18 and has_id:
    print("Allowed (clean version)")
```

**Output:**
```text
Allowed
Allowed (clean version)
```

---

## 11. Loops

### 11.1 `for` with `range`

`range(stop)`, `range(start, stop)`, `range(start, stop, step)`. The `stop` value is **never included**.

```python
for i in range(5):
    print(i, end=" ")
print()

for i in range(2, 7):
    print(i, end=" ")
print()

for i in range(10, 0, -3):
    print(i, end=" ")
print()
```

**Output:**
```text
0 1 2 3 4
2 3 4 5 6
10 7 4 1
```

### 11.2 Looping over sequences

```python
fruits = ["apple", "banana", "cherry"]

for f in fruits:                         # items directly
    print(f)

for i in range(len(fruits)):             # by index
    print(i, fruits[i])

for i, f in enumerate(fruits, start=1):  # index AND item (best way)
    print(i, f)
```

**Output:**
```text
apple
banana
cherry
0 apple
1 banana
2 cherry
1 apple
2 banana
3 cherry
```

### 11.3 `zip`, `reversed`, `sorted`, dict loops

```python
names = ["Asha", "Ravi", "Meena"]
marks = [88, 92, 79]

for n, m in zip(names, marks):           # walk two lists together
    print(n, m)

for x in reversed([1, 2, 3]):
    print(x, end=" ")
print()

for x in sorted([3, 1, 2], reverse=True):
    print(x, end=" ")
print()

d = {"a": 1, "b": 2}
for k, v in d.items():
    print(k, v)
```

**Output:**
```text
Asha 88
Ravi 92
Meena 79
3 2 1
3 2 1
a 1
b 2
```

### 11.4 `while` loop

Use `while` when you do not know in advance how many times to loop.

```python
n = 1234
total = 0
while n > 0:
    total += n % 10      # last digit
    n //= 10             # drop last digit
print(total)
```

**Output:**
```text
10
```

**Dry run:**

| n | n % 10 | total | n after `//= 10` |
|---|---|---|---|
| 1234 | 4 | 4 | 123 |
| 123 | 3 | 7 | 12 |
| 12 | 2 | 9 | 1 |
| 1 | 1 | 10 | 0 |

The loop stops when `n` becomes 0. **Always make sure something inside a `while` loop changes, otherwise it runs forever.**

### 11.5 `break`, `continue` and loop `else`

```python
for i in range(1, 8):
    if i == 3:
        continue         # skip the rest of this round
    if i == 6:
        break            # leave the loop completely
    print(i, end=" ")
print()
```

**Output:**
```text
1 2 4 5
```

**Loop `else`:** the `else` block runs only if the loop finished **without** `break`.

```python
for n in [29, 21]:
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            print(n, "is not prime")
            break
    else:
        print(n, "is prime")
```

**Output:**
```text
29 is prime
21 is not prime
```

### 11.6 Nested loops and patterns

```python
n = 4
for i in range(1, n + 1):
    print("*" * i)

print()

n = 3
for i in range(1, n + 1):
    print(" " * (n - i) + "*" * (2 * i - 1))

print()

for i in range(1, 4):
    for j in range(1, 4):
        print(i * j, end=" ")
    print()
```

**Output:**
```text
*
**
***
****

  *
 ***
*****

1 2 3
2 4 6
3 6 9
```

**Explanation:** The inner loop runs completely for every single step of the outer loop. Outer 3 times × inner 3 times = 9 prints. This is where O(n²) comes from.

### 11.7 Loop patterns you will use in almost every problem

```python
nums = [4, 9, 2, 7, 9]

# Pattern 1: accumulate (sum, count, max)
total = 0
count_even = 0
biggest = float("-inf")
for x in nums:
    total += x
    if x % 2 == 0:
        count_even += 1
    if x > biggest:
        biggest = x
print(total, count_even, biggest)

# Pattern 2: prefix sums
prefix = [0]
for x in nums:
    prefix.append(prefix[-1] + x)
print(prefix)

# Pattern 3: two pointers from both ends
arr = [1, 2, 3, 4, 5]
l, r = 0, len(arr) - 1
while l < r:
    print(arr[l], arr[r])
    l += 1
    r -= 1

# Pattern 4: sum with a generator expression
print(sum(x * x for x in range(4)))
```

**Output:**
```text
31 2 9
[0, 4, 13, 15, 22, 31]
1 5
2 4
14
```

### 11.8 Loop mistakes to avoid

**1. Changing a list while looping over it:**

```python
nums = [2, 4, 6, 8]
for x in nums:
    if x % 2 == 0:
        nums.remove(x)
print(nums)           # expected [], but...
```

**Output:**
```text
[4, 8]
```

Removing items shifts the remaining ones, so the loop skips elements. Build a new list instead:

```python
nums = [2, 4, 6, 8, 9]
nums = [x for x in nums if x % 2 != 0]
print(nums)
```

**Output:**
```text
[9]
```

**2. Off-by-one errors:** `range(n)` gives `0..n-1`. To include `n`, write `range(n + 1)` or `range(1, n + 1)`.

**3. Infinite `while` loops:** forgetting to update the loop variable.

---

## 12. Functions

A function is a reusable block of code. In coding rounds you will write helper functions, and every recursive solution is a function.

### 12.1 Defining and calling

```python
def add(a, b):
    return a + b

print(add(3, 4))
print(add("Hi ", "there"))

def greet(name):
    print("Hi", name)        # no return statement

result = greet("Asha")
print(result)
```

**Output:**
```text
7
Hi there
Hi Asha
None
```

**Explanation:** `return` sends a value back and ends the function immediately. A function without `return` gives back `None`. Python does not check types, so `add` works for numbers and strings.

### 12.2 Default and keyword arguments

```python
def power(base, exp=2):
    return base ** exp

print(power(5))              # exp uses default 2
print(power(2, 10))
print(power(exp=3, base=2))  # keyword arguments, any order
```

**Output:**
```text
25
1024
8
```

### 12.3 Returning multiple values

```python
def stats(arr):
    return min(arr), max(arr), sum(arr) / len(arr)

lo, hi, avg = stats([4, 8, 6])
print(lo, hi, avg)
```

**Output:**
```text
4 8 6.0
```

### 12.4 `*args` and `**kwargs`

```python
def total(*args):            # any number of positional arguments, as a tuple
    return sum(args)

def show(**kwargs):          # any number of keyword arguments, as a dict
    for k, v in kwargs.items():
        print(k, "=", v)

print(total(1, 2, 3, 4))
show(a=1, b=2)

nums = [5, 6, 7]
print(total(*nums))          # * unpacks a list into arguments
```

**Output:**
```text
10
a = 1
b = 2
18
```

### 12.5 Scope: local and global

```python
x = 10                       # global

def f():
    x = 99                   # a NEW local x, global is untouched
    return x

def g():
    global x                 # now we really change the global
    x += 1

print(f(), x)
g()
print(x)
```

**Output:**
```text
99 10
11
```

**Advice:** avoid `global`. Pass values in and return values out. For nested helper functions that update an outer variable, use `nonlocal`:

```python
def count_evens(arr):
    count = 0

    def check(v):
        nonlocal count
        if v % 2 == 0:
            count += 1

    for v in arr:
        check(v)
    return count

print(count_evens([1, 2, 4, 5, 6]))
```

**Output:**
```text
3
```

### 12.6 Mutable arguments can be changed by the function

```python
def add_item(lst):
    lst.append(99)           # changes the caller's list

def inc(x):
    x += 1                   # rebinds a local name only

a = [1]
add_item(a)
print(a)

n = 5
inc(n)
print(n)
```

**Output:**
```text
[1, 99]
5
```

**Explanation:** Python passes references. A list (mutable) can be changed through the reference. An int (immutable) cannot, so `x += 1` only creates a new local number. This matters in DFS/backtracking where you pass a list and modify it.

### 12.7 The mutable default argument trap

```python
def bad(item, bucket=[]):          # the SAME list is reused on every call
    bucket.append(item)
    return bucket

print(bad(1))
print(bad(2))

def good(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket

print(good(1))
print(good(2))
```

**Output:**
```text
[1]
[1, 2]
[1]
[2]
```

**Rule:** never use `[]`, `{}` or `set()` as a default value. Use `None` and create it inside.

### 12.8 Recursion: a function that calls itself

Every recursive function needs (1) a **base case** that stops, and (2) a **smaller problem** in each call.

```python
def factorial(n):
    if n <= 1:                   # base case
        return 1
    return n * factorial(n - 1)  # smaller problem

print(factorial(5))
```

**Output:**
```text
120
```

**How it unfolds:**

```
factorial(5)
= 5 * factorial(4)
= 5 * (4 * factorial(3))
= 5 * (4 * (3 * factorial(2)))
= 5 * (4 * (3 * (2 * factorial(1))))
= 5 * (4 * (3 * (2 * 1)))      <- base case reached, now results come back
= 120
```

Time O(n), space O(n) for the call stack. Python's default recursion limit is about 1000, so very deep recursion raises `RecursionError`.

```python
def sum_digits(n):
    if n < 10:
        return n
    return n % 10 + sum_digits(n // 10)

print(sum_digits(1234))
```

**Output:**
```text
10
```

### 12.9 Lambda (small anonymous functions) and sorting keys

```python
square = lambda x: x * x
print(square(6))

students = [("Ravi", 85), ("Asha", 92), ("Meena", 85)]
students.sort(key=lambda s: (-s[1], s[0]))     # marks high to low, then name A to Z
print(students)

pairs = [(1, "b"), (2, "a")]
print(sorted(pairs, key=lambda p: p[1]))
```

**Output:**
```text
36
[('Asha', 92), ('Meena', 85), ('Ravi', 85)]
[(2, 'a'), (1, 'b')]
```

**Explanation:** `key=` tells `sort` what to compare. Returning a **tuple** sorts by several fields; a minus sign flips a number's order.

### 12.10 Built-in helpers worth knowing

```python
nums = [1, 2, 3, 4, 5]
print(list(map(lambda x: x * 2, nums)))
print(list(filter(lambda x: x % 2 == 0, nums)))
print(any(x > 4 for x in nums), all(x > 0 for x in nums))
print(max(["kiwi", "banana", "fig"], key=len))
print(list(zip([1, 2, 3], "abc")))
print(sum(nums), min(nums), max(nums), len(nums))
```

**Output:**
```text
[2, 4, 6, 8, 10]
[2, 4]
True True
banana
[(1, 'a'), (2, 'b'), (3, 'c')]
15 1 5 5
```

### 12.11 Docstrings and type hints (good habit)

```python
def area(width: int, height: int) -> int:
    """Return the area of a rectangle."""
    return width * height

print(area(3, 4))
```

**Output:**
```text
12
```

Type hints do not enforce anything. They only help you and your reader.

---

## 13. Input and output in coding rounds

On platforms such as HackerRank, HackerEarth, CodeChef and company assessment portals, you often read from **standard input** and print to **standard output**.

### 13.1 Reading input

```python
# Read one integer
n = int(input())

# Read a line of space-separated integers into a list
arr = list(map(int, input().split()))

# Read two integers on one line
a, b = map(int, input().split())

# Read a string
s = input().strip()
```

**Sample input:**
```text
5
10 20 30 40 50
```

With the first two reading lines above, `n` is `5` and `arr` is `[10, 20, 30, 40, 50]`.

**Explanation:** `input()` always returns a **string**. `split()` breaks it into pieces, `map(int, ...)` converts each piece, and `list(...)` collects the results.

### 13.2 Multiple test cases

```python
t = int(input())
for _ in range(t):
    n = int(input())
    arr = list(map(int, input().split()))
    print(sum(arr))
```

### 13.3 Fast input for large data

When input has 10⁵ or more numbers, `input()` can be slow. Use:

```python
import sys
input = sys.stdin.readline

n = int(input())
arr = list(map(int, input().split()))
```

(`readline` keeps the trailing newline, but `int()` and `split()` ignore it. Use `.strip()` for strings.)

### 13.4 Printing results

```python
arr = [1, 2, 3]
print(*arr)                          # unpack: space separated
print(" ".join(map(str, arr)))       # same result
print(*arr, sep=",")
print("Sum:", sum(arr))
print(f"Average: {sum(arr) / len(arr):.2f}")
```

**Output:**
```text
1 2 3
1 2 3
1,2,3
Sum: 6
Average: 2.00
```

**Tips:**

- Print exactly what the problem asks. Extra text like "Answer is" can fail the test.
- Many platforms (LeetCode style) give you a function to complete (`def twoSum(self, nums, target)`) and do the input/output for you. Read the template carefully.

---

## 14. Mini programs (put it all together)

Each program below uses several concepts from this note.

### 14.1 FizzBuzz

```python
for i in range(1, 16):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

**Output:**
```text
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
```

**Key idea:** check the most specific condition (`% 15`) first.

### 14.2 Second largest element (single pass)

```python
def second_largest(arr):
    first = second = float("-inf")
    for x in arr:
        if x > first:
            second = first
            first = x
        elif first > x > second:
            second = x
    return second

print(second_largest([10, 5, 20, 8, 20]))
```

**Output:**
```text
10
```

**Time O(n), Space O(1).** The condition `first > x > second` ignores duplicates of the maximum.

### 14.3 First non-repeating character

```python
def first_unique(s):
    freq = {}
    for ch in s:
        freq[ch] = freq.get(ch, 0) + 1
    for ch in s:
        if freq[ch] == 1:
            return ch
    return None

print(first_unique("swiss"))
print(first_unique("aabb"))
```

**Output:**
```text
w
None
```

**Time O(n), Space O(k)** where k is the number of distinct characters.

### 14.4 Remove duplicates but keep the order

```python
def dedupe(arr):
    seen = set()
    result = []
    for x in arr:
        if x not in seen:
            seen.add(x)
            result.append(x)
    return result

print(dedupe([3, 1, 3, 2, 1, 4]))
```

**Output:**
```text
[3, 1, 2, 4]
```

### 14.5 Prime numbers up to 20

```python
def is_prime(n):
    if n < 2:
        return False
    i = 2
    while i * i <= n:
        if n % i == 0:
            return False
        i += 1
    return True

print([n for n in range(1, 21) if is_prime(n)])
```

**Output:**
```text
[2, 3, 5, 7, 11, 13, 17, 19]
```

**Time O(√n) per number.** We only test divisors up to √n because divisors come in pairs.

### 14.6 Reverse a number

```python
def reverse_number(n):
    rev = 0
    while n > 0:
        rev = rev * 10 + n % 10
        n //= 10
    return rev

print(reverse_number(1234))
print(reverse_number(121) == 121)      # palindrome number?
```

**Output:**
```text
4321
True
```

### 14.7 Word frequency

```python
from collections import Counter

sentence = "the cat and the hat and the bat"
freq = Counter(sentence.split())
print(freq.most_common(2))
```

**Output:**
```text
[('the', 3), ('and', 2)]
```

---

## 15. Common errors and what they mean

```python
tests = [
    lambda: [1, 2, 3][5],
    lambda: {"a": 1}["b"],
    lambda: int("abc"),
    lambda: 10 / 0,
    lambda: undefined_name,
    lambda: "a" + 1,
]
for t in tests:
    try:
        t()
    except Exception as e:
        print(type(e).__name__, "-", e)
```

**Output:**
```text
IndexError - list index out of range
KeyError - 'b'
ValueError - invalid literal for int() with base 10: 'abc'
ZeroDivisionError - division by zero
NameError - name 'undefined_name' is not defined
TypeError - can only concatenate str (not "int") to str
```

| Error | Usual cause | Fix |
|---|---|---|
| `IndexError` | index outside `0..len-1` | check bounds, `range(len(a))` |
| `KeyError` | missing dict key | `d.get(key)` or `if key in d` |
| `ValueError` | wrong value for a conversion | clean the input first |
| `TypeError` | mixing types, calling something wrong | convert with `int()`, `str()` |
| `NameError` | variable not defined or misspelled | check spelling and scope |
| `ZeroDivisionError` | divisor is 0 | check before dividing |
| `IndentationError` | inconsistent spaces | use 4 spaces everywhere |
| `RecursionError` | no base case or too deep | fix the base case |

---

## 16. Practice exercises

Try these without looking anything up. Solve each in under 10 minutes.

**Numbers and strings**
1. Read an integer and print whether it is even or odd.
2. Print the sum of digits of a number (use a `while` loop).
3. Count the vowels and consonants in a string.
4. Check whether two strings are anagrams (two ways: `sorted` and `Counter`).
5. Reverse the words of a sentence: `"I love Python"` becomes `"Python love I"`.

**Lists, sets, dicts**
6. Find the largest and smallest element without using `max` and `min`.
7. Remove duplicates from a list, keeping the order.
8. Print the frequency of every element in a list.
9. Find the elements common to two lists (using sets).
10. Group words by their length using `defaultdict(list)`.

**Loops and functions**
11. Print all primes up to N.
12. Print the first N Fibonacci numbers (iteratively).
13. Write a function that returns `(count_of_even, count_of_odd)` for a list.
14. Write a recursive function for `power(a, b)`.
15. Sort a list of `(name, marks)` by marks descending and then by name ascending.

---

## 17. Quick revision sheet

### Types at a glance

| Type | Example | Ordered | Mutable | Duplicates | Lookup |
|---|---|---|---|---|---|
| `int`/`float` | `5`, `2.5` | | no | | |
| `str` | `"abc"` | yes | no | yes | O(n) |
| `list` | `[1, 2]` | yes | yes | yes | O(n) |
| `tuple` | `(1, 2)` | yes | no | yes | O(n) |
| `set` | `{1, 2}` | no | yes | no | O(1) |
| `dict` | `{"a": 1}` | insertion | yes | keys unique | O(1) |

### Must-remember lines

```python
a, b = b, a                              # swap
for i, x in enumerate(arr): ...          # index and value
for x, y in zip(a, b): ...               # two lists together
arr[::-1]                                # reverse
sorted(arr, key=lambda x: (-x[1], x[0])) # multi-key sort
freq[x] = freq.get(x, 0) + 1             # count
[x for x in arr if x > 0]                # comprehension
grid = [[0] * m for _ in range(n)]       # 2D list (the right way)
float("inf"), float("-inf")              # initial min/max
" ".join(map(str, arr))                  # print list
n = int(input()); arr = list(map(int, input().split()))
```

### Top 8 mistakes

1. `[[0] * m] * n` for a 2D grid (all rows are the same list).
2. `a = a.sort()` (it gives `None`).
3. Using `list.pop(0)` as a queue (use `deque`).
4. Modifying a list while looping over it.
5. Mutable default arguments (`def f(x, lst=[])`).
6. Comparing floats with `==`.
7. Forgetting that `range(n)` stops at `n - 1`.
8. Naming a variable `list`, `max`, `sum` or `str`.

### Readiness check before the Math section

You are ready if you can, without help:
- Read `n` and an array from input and print results in the required format.
- Count frequencies using a dictionary and using `Counter`.
- Write nested loops, `while` loops, and use `break`/`continue`.
- Write functions with default arguments, multiple returns, and a recursive base case.
- Explain the time cost of `x in list` vs `x in set` (from Topic 1).

---

**Next topic:** Math for DSA (even/odd, divisibility, GCD/LCM, primes, modular arithmetic, and number tricks that show up again and again in coding rounds).
