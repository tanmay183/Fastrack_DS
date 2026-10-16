# 📐 Math in DSA (Python) — Complete Guide for Coding Rounds

> Goal: Cover every math idea you need for DSA problems, with 15 solved examples (easy → hard), each with line-by-line comments and verified output.

---

## 📑 Table of Contents
1. [Why Math matters in DSA](#1-why-math-matters-in-dsa)
2. [All Math Concepts You Need](#2-all-math-concepts-you-need)
3. [Famous Math Algorithms (Must Know)](#3-famous-math-algorithms-must-know)
4. [15 Standard Problems (Easy → Hard)](#4-15-standard-problems-easy--hard)
5. [Cheat Sheet & Formulas](#5-cheat-sheet--formulas)
6. [Common Mistakes & Contest Tips](#6-common-mistakes--contest-tips)
7. [Practice List](#7-practice-list)

---

## 1. Why Math matters in DSA
Many "hard-looking" problems have a **short math trick** that turns an O(n²) or O(n!) idea into O(n), O(log n) or even O(1). Examples: sum formula instead of loop, sqrt limit for primes, fast power instead of repeated multiplication, modulo so numbers do not explode.

---

## 2. All Math Concepts You Need

### 2.1 Basic number properties
- **Even/Odd:** `n % 2 == 0` (or `n & 1`)
- **Divisibility:** `a % b == 0` means b divides a.
- **Digits:** last digit = `n % 10`, remove last digit = `n // 10`, number of digits = `len(str(n))`.
- **Floor / Ceil division:** `a // b` is floor. Ceil of a/b = `(a + b - 1) // b`.
- Python has **no integer overflow** (big integers are automatic). In C++/Java you must care, but the *interviewer's expected idea* (mod, long) is still the same.

### 2.2 Modular arithmetic
Used whenever the answer is "print answer modulo 10^9+7".
- `(a + b) % m = ((a % m) + (b % m)) % m`
- `(a * b) % m = ((a % m) * (b % m)) % m`
- `(a - b) % m` → in Python already non-negative. In C++/Java use `(a - b + m) % m`.
- **Division is NOT allowed directly.** Use **modular inverse**: `a / b ≡ a * b^(m-2) (mod m)` when `m` is prime (Fermat's Little Theorem).
- Common modulus: `MOD = 10**9 + 7` (prime).

### 2.3 GCD and LCM
- `gcd(a, b)` = largest number dividing both.
- `lcm(a, b) = a * b // gcd(a, b)`
- `gcd(a, b) = gcd(b, a % b)` (Euclid). Time: O(log(min(a,b))).
- Python: `math.gcd`, `math.lcm`.

### 2.4 Prime numbers
- Prime = only divisors 1 and itself. 1 is **not** prime. 2 is the only even prime.
- **Check up to √n only** (if n has a factor > √n, it also has one < √n).
- **Sieve of Eratosthenes** → all primes up to N in O(N log log N).
- **Prime factorization** by trial division O(√n), or with **Smallest Prime Factor (SPF)** sieve → O(log n) per query.

### 2.5 Divisors
- Number of divisors if `n = p1^a1 * p2^a2 ...` is `(a1+1)(a2+1)...`
- Sum of divisors = product of `(p^(a+1) - 1)/(p - 1)`.
- Enumerate divisors by looping `i` up to √n and taking both `i` and `n // i`.

### 2.6 Euler's Totient φ(n)
- Count of numbers in `1..n` coprime with n.
- `φ(n) = n * Π (1 - 1/p)` over distinct primes p of n.
- Euler's theorem: `a^φ(m) ≡ 1 (mod m)` if gcd(a, m) = 1.

### 2.7 Fast (Binary) Exponentiation
- Compute `a^b` in O(log b): square the base, halve the exponent.
- With modulus: `pow(a, b, m)` in Python (built-in, fast).

### 2.8 Combinatorics
- **Factorial:** n! ; **Permutation:** `nPr = n! / (n-r)!`
- **Combination:** `nCr = n! / (r! (n-r)!)`
- Properties: `nCr = nC(n-r)`, `nCr = (n-1)C(r-1) + (n-1)Cr` (Pascal's triangle).
- **Catalan numbers:** `C(n) = (2n)! / ((n+1)! n!)` → counts balanced brackets, unique BSTs, etc.
- **Stars and bars:** ways to put n identical items in k boxes = `C(n+k-1, k-1)`.
- **Inclusion–Exclusion:** `|A ∪ B| = |A| + |B| - |A ∩ B|`.
- **Pigeonhole:** n+1 items in n boxes → some box has 2.

### 2.9 Sums and Series
| Series | Formula |
|---|---|
| 1 + 2 + … + n | `n(n+1)/2` |
| 1² + 2² + … + n² | `n(n+1)(2n+1)/6` |
| 1³ + … + n³ | `(n(n+1)/2)²` |
| AP: a, a+d, … (n terms) | `n/2 * (2a + (n-1)d)` |
| GP: a, ar, ar², … (n terms) | `a(r^n - 1)/(r - 1)` |

### 2.10 Powers, Logs, Roots
- `log2(n)` = number of times you can halve n (explains O(log n)).
- Power of two check: `n > 0 and (n & (n-1)) == 0`.
- Integer square root: `math.isqrt(n)` (exact, avoids float errors).
- Number of digits of n: `floor(log10(n)) + 1`.

### 2.11 Bit-math tricks
- `x ^ x = 0`, `x ^ 0 = x` → find single/missing number with XOR.
- `n & (n-1)` removes lowest set bit; `n & -n` isolates lowest set bit.
- `1 << k` = 2^k; `n >> k` = n // 2^k.

### 2.12 Number systems / Base conversion
- Decimal → base b: repeatedly take `n % b`, then `n //= b`.
- base b → decimal: `result = result * b + digit`.
- Excel column titles are "bijective base-26".

### 2.13 Matrix math
- Matrix multiplication; **Matrix exponentiation** solves linear recurrences (Fibonacci) in O(log n).

### 2.14 Geometry basics
- Distance: `√((x2-x1)² + (y2-y1)²)` (compare squared distances to avoid floats).
- Area of triangle (coordinates): `|x1(y2-y3)+x2(y3-y1)+x3(y1-y2)| / 2`.
- Cross product sign tells left/right turn / collinearity.
- Slope as a fraction: reduce `(dy, dx)` by gcd → avoids float precision.

### 2.15 Probability & Expected Value (basics)
- `P(A) = favourable / total`.
- Expected value `E[X] = Σ x * P(x)`; linearity: `E[X+Y] = E[X] + E[Y]`.

### 2.16 Complexity math
- Harmonic series `n + n/2 + n/3 + … ≈ n log n` → why the sieve and "loop over multiples" are fast.
- Recurrence `T(n) = 2T(n/2) + n → O(n log n)`.
- `n ≤ 10^5` → O(n log n); `n ≤ 10^7` → O(n) ; `n ≤ 10^9` → O(√n) or O(log n); `n ≤ 20` → O(2^n).

### 2.17 Chinese Remainder Theorem (CRT) & Extended Euclid
- Extended Euclid: finds x, y with `a*x + b*y = gcd(a, b)`. Gives modular inverse for any coprime pair.
- CRT: combine congruences `x ≡ r1 (mod m1)`, `x ≡ r2 (mod m2)` into a single one (coprime moduli).

---

## 3. Famous Math Algorithms (Must Know)

| # | Algorithm | Used for | Time |
|---|---|---|---|
| 1 | Euclid's GCD | gcd, lcm, simplify fractions, slopes | O(log n) |
| 2 | Sieve of Eratosthenes | all primes ≤ N | O(N log log N) |
| 3 | SPF Sieve | fast factorization of many queries | O(N log log N) build, O(log n) query |
| 4 | Trial-division factorization | factor one number | O(√n) |
| 5 | Binary Exponentiation | a^b, a^b mod m | O(log b) |
| 6 | Modular Inverse (Fermat / Ext. Euclid) | division under mod, nCr mod p | O(log m) |
| 7 | Pascal's Triangle / nCr precompute | combinations | O(n²) or O(n) |
| 8 | Euler Totient | coprime counting, mod exponent | O(√n) |
| 9 | Matrix Exponentiation | Fibonacci / linear recurrences | O(k³ log n) |
| 10 | Catalan DP | BST count, brackets | O(n²) |
| 11 | Extended Euclid + CRT | congruence systems | O(log n) |
| 12 | Kadane / Prefix sums (math-flavoured) | range sums | O(n) |
| 13 | Gaussian-sum / XOR trick | missing/duplicate number | O(n) |

---

## 4. 15 Standard Problems (Easy → Hard)

---

### 🟢 Problem 1 — Digit Sum and Reverse of a Number (Easy)
**Idea:** `n % 10` gives the last digit, `n // 10` drops it. Build the reverse with `rev = rev*10 + digit`.

```python
def digit_sum_and_reverse(n):
    total = 0            # will store sum of all digits
    rev = 0              # will store the reversed number
    while n > 0:         # repeat until no digits are left
        digit = n % 10   # extract the last digit (12345 % 10 = 5)
        total += digit   # add the digit to the running sum
        rev = rev * 10 + digit  # shift rev left by one place, then append digit
        n //= 10         # remove the last digit (12345 // 10 = 1234)
    return total, rev    # return both results as a tuple

print(digit_sum_and_reverse(12345))  # test with 12345
```
**Output**
```
(15, 54321)
```
**Complexity:** O(log10 n) time, O(1) space.

---

### 🟢 Problem 2 — Palindrome Number (Easy)
**Idea:** Reverse the number mathematically (no string conversion) and compare with the original.

```python
def is_palindrome(x):
    if x < 0:            # negative numbers have '-' sign, so can never be palindromes
        return False
    original = x         # keep a copy because x will be destroyed in the loop
    rev = 0              # this will hold the reversed number
    while x > 0:         # process each digit
        rev = rev * 10 + x % 10  # append last digit of x to rev
        x //= 10         # remove last digit from x
    return rev == original  # palindrome if reverse equals the original

print(is_palindrome(121))   # reads same both ways
print(is_palindrome(123))   # does not
print(is_palindrome(-121))  # negative -> False
```
**Output**
```
True
False
False
```
**Complexity:** O(log10 n) time, O(1) space.

---

### 🟢 Problem 3 — Prime Check using √n (Easy)
**Idea:** If n has a divisor, one of the pair is ≤ √n. So check only up to √n.

```python
import math               # for math.isqrt (exact integer square root)

def is_prime(n):
    if n < 2:             # 0, 1 and negatives are not prime
        return False
    if n < 4:             # 2 and 3 are prime
        return True
    if n % 2 == 0:        # all even numbers > 2 are not prime
        return False
    for i in range(3, math.isqrt(n) + 1, 2):  # check odd divisors up to sqrt(n)
        if n % i == 0:    # found a divisor -> not prime
            return False
    return True           # no divisor found -> prime

print(is_prime(29))                                  # prime
print(is_prime(1))                                   # not prime
print(is_prime(100))                                 # not prime
print([x for x in range(1, 31) if is_prime(x)])      # all primes up to 30
```
**Output**
```
True
False
False
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```
**Complexity:** O(√n) time, O(1) space.

---

### 🟢 Problem 4 — GCD and LCM (Euclid's Algorithm) (Easy)
**Idea:** `gcd(a, b) = gcd(b, a % b)` until b becomes 0. Then `lcm = a*b // gcd`.

```python
def gcd(a, b):
    while b != 0:        # keep going until the remainder is zero
        a, b = b, a % b  # replace (a, b) by (b, a mod b)
    return a             # when b is 0, a holds the GCD

def lcm(a, b):
    return a // gcd(a, b) * b  # divide first, then multiply (keeps numbers smaller)

print(gcd(48, 18))   # common divisors: 1,2,3,6 -> 6
print(lcm(48, 18))   # smallest common multiple
```
**Output**
```
6
144
```
**Trace:** (48,18) → (18,12) → (12,6) → (6,0) → answer 6.
**Complexity:** O(log(min(a,b))).

---

### 🟢 Problem 5 — Missing Number using Sum Formula (Easy)
**Idea:** Array has n distinct numbers from `0..n` with one missing. Expected sum = `n(n+1)/2`. Missing = expected − actual.

```python
def missing_number(nums):
    n = len(nums)                  # numbers should be 0..n, one is missing
    expected = n * (n + 1) // 2    # Gaussian sum of 0..n
    actual = sum(nums)             # sum of what we actually have
    return expected - actual       # the difference is the missing value

print(missing_number([3, 0, 1]))                 # 0..3, missing 2
print(missing_number([9, 6, 4, 2, 3, 5, 7, 0, 1]))  # 0..9, missing 8
```
**Output**
```
2
8
```
**Complexity:** O(n) time, O(1) space. (XOR trick also works and never overflows in C++/Java.)

---

### 🟡 Problem 6 — Sieve of Eratosthenes (Medium)
**Idea:** Assume all numbers are prime. For each prime p, cross out its multiples starting from p².

```python
def sieve(n):
    is_p = [True] * (n + 1)          # is_p[i] tells whether i is still considered prime
    is_p[0] = False                  # 0 is not prime
    if n >= 1:
        is_p[1] = False              # 1 is not prime
    i = 2                            # start from the first prime
    while i * i <= n:                # only need primes up to sqrt(n)
        if is_p[i]:                  # if i is prime ...
            for j in range(i * i, n + 1, i):  # ... cross out multiples starting at i*i
                is_p[j] = False      # j is a multiple of i -> composite
        i += 1                       # move to next number
    return [k for k in range(n + 1) if is_p[k]]  # collect indices still marked True

print(sieve(30))        # primes up to 30
print(len(sieve(100)))  # how many primes up to 100
```
**Output**
```
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
25
```
**Complexity:** O(N log log N) time, O(N) space.

---

### 🟡 Problem 7 — Prime Factorization (Trial Division + SPF) (Medium)
**Idea A:** divide by every d while `d*d <= n`. **Idea B:** precompute the smallest prime factor for every number, then factor any number in O(log n).

```python
def prime_factors(n):
    factors = {}                       # prime -> exponent
    d = 2                              # smallest candidate divisor
    while d * d <= n:                  # no need to go beyond sqrt(remaining n)
        while n % d == 0:              # while d divides n completely ...
            factors[d] = factors.get(d, 0) + 1  # ... count one more power of d
            n //= d                    # ... and divide it out
        d += 1                         # try next divisor
    if n > 1:                          # whatever remains > 1 is itself a prime
        factors[n] = factors.get(n, 0) + 1
    return factors

def build_spf(n):
    spf = list(range(n + 1))           # spf[i] = i initially (i is its own smallest prime factor)
    i = 2
    while i * i <= n:                  # loop through potential primes
        if spf[i] == i:                # i is prime (nobody smaller changed it)
            for j in range(i * i, n + 1, i):  # visit multiples of i
                if spf[j] == j:        # only set if not already set by a smaller prime
                    spf[j] = i         # i is the smallest prime factor of j
        i += 1
    return spf

def factorize(x, spf):
    res = []                           # list of prime factors (with repetition)
    while x > 1:                       # until x becomes 1
        res.append(spf[x])             # smallest prime factor of x
        x //= spf[x]                   # divide it out
    return res

print(prime_factors(360))              # 360 = 2^3 * 3^2 * 5
spf = build_spf(100)                   # build table up to 100 once
print(factorize(84, spf))              # factorize quickly using table
```
**Output**
```
{2: 3, 3: 2, 5: 1}
[2, 2, 3, 7]
```
**Complexity:** trial division O(√n); SPF query O(log n).

---

### 🟡 Problem 8 — Fast Exponentiation (Pow(x, n) and a^b mod m) (Medium)
**Idea:** `a^b = (a²)^(b/2)` if b is even, else `a * a^(b-1)`. Only O(log b) multiplications.

```python
MOD = 10**9 + 7                      # the famous prime modulus

def mod_pow(base, exp, mod):
    result = 1                       # identity for multiplication
    base %= mod                      # reduce base first
    while exp > 0:                   # process every bit of exp
        if exp & 1:                  # if the current lowest bit is 1
            result = result * base % mod  # multiply result by current base
        base = base * base % mod     # square the base (a, a^2, a^4, a^8 ...)
        exp >>= 1                    # shift exponent right (divide by 2)
    return result

def my_pow(x, n):                    # LeetCode 50 style (float, negative exponent)
    if n < 0:                        # x^-n = (1/x)^n
        x = 1 / x                    # invert base
        n = -n                       # make exponent positive
    result = 1.0                     # running answer
    while n:                         # loop over bits of n
        if n & 1:                    # lowest bit set?
            result *= x              # include current x in answer
        x *= x                       # square x
        n >>= 1                      # next bit
    return result

print(mod_pow(2, 10, 1000))          # 1024 % 1000
print(mod_pow(2, 100, MOD))          # huge power, reduced mod 1e9+7
print(my_pow(2.0, -2))               # 1 / (2^2)
print(my_pow(2.0, 10))               # 2^10
```
**Output**
```
24
976371285
0.25
1024.0
```
**Complexity:** O(log exp) time, O(1) space.

---

### 🟡 Problem 9 — Trailing Zeroes in n! (Medium)
**Idea:** A trailing zero comes from a factor 10 = 2×5. There are always more 2s than 5s, so count the 5s: `n/5 + n/25 + n/125 + …`

```python
def trailing_zeroes(n):
    count = 0              # total number of factor 5s found
    while n > 0:           # keep dividing by 5
        n //= 5            # numbers divisible by 5, then 25, then 125 ...
        count += n         # each level adds that many extra factors of 5
    return count

print(trailing_zeroes(5))     # 5! = 120 -> 1 zero
print(trailing_zeroes(25))    # 25/5 + 25/25 = 5 + 1
print(trailing_zeroes(100))   # 20 + 4
```
**Output**
```
1
6
24
```
**Complexity:** O(log5 n) time, O(1) space.

---

### 🟡 Problem 10 — Excel Sheet Column Number/Title (Base Conversion) (Medium)
**Idea:** Letters A–Z act like digits 1–26 (base 26 with no zero). Convert both ways.

```python
def title_to_number(s):
    result = 0                                   # numeric value being built
    for ch in s:                                 # go left to right
        result = result * 26 + (ord(ch) - ord('A') + 1)  # shift by 26, add letter value (A=1..Z=26)
    return result

def number_to_title(n):
    res = []                                     # collect letters (in reverse order)
    while n > 0:
        n -= 1                                   # shift to 0-based so A=0 ... Z=25
        res.append(chr(n % 26 + ord('A')))       # last letter = remainder converted to char
        n //= 26                                 # move to the next higher position
    return ''.join(reversed(res))                # reverse because we built from the end

print(title_to_number("A"))     # first column
print(title_to_number("AB"))    # 1*26 + 2
print(title_to_number("ZY"))    # 26*26 + 25
print(number_to_title(701))     # reverse conversion
print(number_to_title(28))      # reverse conversion
```
**Output**
```
1
28
701
ZY
AB
```
**Complexity:** O(length of string) / O(log26 n).

---

### 🔴 Problem 11 — nCr mod p with Modular Inverse (Medium-Hard)
**Idea:** `nCr = n! / (r!(n-r)!)`. Division under mod is replaced by multiplying with the **modular inverse** `x^(p-2) mod p` (Fermat). Precompute factorials and inverse factorials once, then every query is O(1).

```python
MOD = 10**9 + 7                                  # prime modulus
N = 1000                                         # maximum n we will ever query

fact = [1] * (N + 1)                             # fact[i] = i! mod MOD
for i in range(1, N + 1):
    fact[i] = fact[i - 1] * i % MOD              # i! = (i-1)! * i

inv_fact = [1] * (N + 1)                         # inv_fact[i] = (i!)^(-1) mod MOD
inv_fact[N] = pow(fact[N], MOD - 2, MOD)         # Fermat: inverse of N! is (N!)^(MOD-2)
for i in range(N, 0, -1):
    inv_fact[i - 1] = inv_fact[i] * i % MOD      # (i-1)!^-1 = (i!)^-1 * i

def nCr(n, r):
    if r < 0 or r > n:                           # invalid choice -> 0 ways
        return 0
    return fact[n] * inv_fact[r] % MOD * inv_fact[n - r] % MOD  # n! * (r!)^-1 * ((n-r)!)^-1

print(nCr(10, 3))                                # 10*9*8/6
print(nCr(5, 2))
print(nCr(5, 7))                                 # r > n
print(nCr(1000, 500))                            # huge value, reduced mod 1e9+7
```
**Output**
```
120
10
0
159835829
```
**Complexity:** O(N) precompute, O(1) per query.

---

### 🔴 Problem 12 — Euler's Totient Function φ(n) (Hard)
**Idea:** For every distinct prime p dividing n, multiply result by `(1 − 1/p)`. Do it in integers: `result -= result // p`.

```python
def phi(n):
    result = n                      # start with n, we will remove non-coprime numbers
    p = 2                           # first candidate prime
    while p * p <= n:               # only check up to sqrt(n)
        if n % p == 0:              # p is a prime factor of n
            while n % p == 0:       # strip every power of p from n
                n //= p
            result -= result // p   # multiply result by (1 - 1/p) using integers
        p += 1                      # next candidate
    if n > 1:                       # leftover n is a prime > sqrt(original n)
        result -= result // n       # apply the same factor for it
    return result

print(phi(10))   # numbers 1,3,7,9 are coprime with 10
print(phi(36))   # 36 * (1-1/2) * (1-1/3)
print(phi(13))   # prime p -> p-1
```
**Output**
```
4
12
12
```
**Complexity:** O(√n) time, O(1) space.

---

### 🔴 Problem 13 — Fibonacci in O(log n) using Matrix Exponentiation (Hard)
**Idea:** `[[1,1],[1,0]]^n = [[F(n+1), F(n)], [F(n), F(n-1)]]`. Use binary exponentiation on the 2×2 matrix.

```python
def mat_mul(A, B):
    return [                                              # standard 2x2 matrix multiplication
        [A[0][0]*B[0][0] + A[0][1]*B[1][0], A[0][0]*B[0][1] + A[0][1]*B[1][1]],
        [A[1][0]*B[0][0] + A[1][1]*B[1][0], A[1][0]*B[0][1] + A[1][1]*B[1][1]],
    ]

def mat_pow(M, n):
    result = [[1, 0], [0, 1]]        # identity matrix (acts like the number 1)
    while n > 0:                     # binary exponentiation on matrices
        if n & 1:                    # if current bit is 1
            result = mat_mul(result, M)  # multiply into the answer
        M = mat_mul(M, M)            # square the matrix
        n >>= 1                      # next bit
    return result

def fib(n):
    return mat_pow([[1, 1], [1, 0]], n)[0][1]  # element [0][1] of the matrix power is F(n)

print(fib(0))    # base case
print(fib(10))   # 0 1 1 2 3 5 8 13 21 34 55
print(fib(50))   # too slow with naive recursion, instant here
print(fib(90))   # big number, Python handles it with no overflow
```
**Output**
```
0
55
12586269025
2880067194370816120
```
**Complexity:** O(log n) time (8 multiplications per step), O(1) space.

---

### 🔴 Problem 14 — Unique Binary Search Trees (Catalan Numbers) (Hard)
**Idea:** Pick root `i`; left subtree uses `i-1` nodes, right uses `n-i`. `dp[n] = Σ dp[i-1] * dp[n-i]`. This is the Catalan recurrence; closed form `C(2n, n)/(n+1)`.

```python
from math import comb                          # comb(n, r) = nCr

def num_trees(n):
    dp = [0] * (n + 1)                         # dp[k] = number of BSTs with k nodes
    dp[0] = 1                                  # empty tree: exactly 1 way
    for nodes in range(1, n + 1):              # compute for every size
        for root in range(1, nodes + 1):       # choose each value as the root
            left = root - 1                    # nodes that go to the left subtree
            right = nodes - root               # nodes that go to the right subtree
            dp[nodes] += dp[left] * dp[right]  # independent choices multiply
    return dp[n]

def catalan(n):
    return comb(2 * n, n) // (n + 1)           # closed-form Catalan number

print(num_trees(3))      # LeetCode 96 example
print(num_trees(5))
print(catalan(5))        # must match num_trees(5)
print(num_trees(19))     # largest value that fits in 32-bit int
```
**Output**
```
5
42
42
1767263190
```
**Complexity:** DP O(n²) time, O(n) space; formula O(n).

---

### 🔴 Problem 15 — Extended Euclid, Modular Inverse and Chinese Remainder Theorem (Hardest)
**Idea:** Extended Euclid finds `x, y` with `a*x + b*y = gcd`. If gcd = 1 then `x` is the inverse of `a` mod `b`. CRT merges congruences one by one using these inverses.

```python
def ext_gcd(a, b):
    if b == 0:                              # base case: gcd(a, 0) = a  with  a*1 + 0*0 = a
        return a, 1, 0
    g, x1, y1 = ext_gcd(b, a % b)           # solve smaller problem: b*x1 + (a%b)*y1 = g
    x = y1                                  # back-substitute to get coefficient of a
    y = x1 - (a // b) * y1                  # and coefficient of b
    return g, x, y                          # a*x + b*y = g

def mod_inverse(a, m):
    g, x, _ = ext_gcd(a, m)                 # a*x + m*y = g
    if g != 1:                              # inverse exists only if a and m are coprime
        return None
    return x % m                            # make it positive inside [0, m-1]

def crt(rems, mods):                        # moduli must be pairwise coprime
    x, M = rems[0], mods[0]                 # current solution: x (mod M)
    for r, m in zip(rems[1:], mods[1:]):    # merge the remaining congruences one by one
        g, p, _ = ext_gcd(M, m)             # M*p + m*q = 1  -> p is inverse of M (mod m)
        x = (x + (r - x) * p * M) % (M * m) # shift x by multiples of M so that x = r (mod m)
        M *= m                              # new combined modulus
    return x

print(ext_gcd(30, 18))                      # (gcd, x, y) with 30x + 18y = gcd
print(mod_inverse(3, 11))                   # 3 * 4 = 12 = 1 (mod 11)
print(mod_inverse(4, 8))                    # gcd(4,8)=4 -> no inverse
print(crt([2, 3, 2], [3, 5, 7]))            # x%3=2, x%5=3, x%7=2
```
**Output**
```
(6, -1, 2)
4
None
23
```
**Check:** `30*(-1) + 18*2 = 6` ✔, and `23 % 3 = 2`, `23 % 5 = 3`, `23 % 7 = 2` ✔.
**Complexity:** O(log m) per merge.

---

## 5. Cheat Sheet & Formulas

```python
import math
math.gcd(a, b)            # gcd
math.lcm(a, b)            # lcm (Python 3.9+)
math.isqrt(n)             # exact integer sqrt
math.comb(n, r)           # nCr (exact big integer)
math.perm(n, r)           # nPr
math.factorial(n)         # n!
pow(a, b, m)              # fast a^b mod m
pow(a, -1, m)             # modular inverse (Python 3.8+)
divmod(a, b)              # (a // b, a % b) in one call
bin(n).count('1')         # number of set bits
n.bit_length()            # bits needed for n
```

| Situation | Go-to trick |
|---|---|
| "Answer modulo 1e9+7" | do `% MOD` after every `+` and `*`; use inverse for division |
| Primality of many numbers | Sieve |
| Factorize many numbers | SPF sieve |
| a^b with huge b | binary exponentiation |
| Count of pairs with gcd = 1 | Euler totient / inclusion-exclusion |
| Large n (≤ 10^18) recurrence | matrix exponentiation |
| Count sequences / brackets / BSTs | Catalan |
| Find missing/duplicate | sum formula or XOR |
| Slope / ratio comparisons | reduce by gcd, never use float |

---

## 6. Common Mistakes & Contest Tips
1. **Use `//` not `/`** for integer division (`/` gives float and loses precision for big numbers).
2. **Never use `n ** 0.5`** to get exact sqrt of big n → use `math.isqrt(n)`.
3. **Sieve starts crossing at `i*i`**, not `2*i` (faster, same result).
4. **Forgetting 1 is not prime** and `0! = 1`.
5. **Mod in the middle of every multiplication**; in Python big ints still work but become slow.
6. **Modular inverse only works** if the modulus is prime (Fermat) or gcd = 1 (Extended Euclid).
7. Always check **constraints first**: they tell you the expected complexity.
8. Test edge cases: `n = 0, 1, 2`, negative numbers, very large numbers, `a = b`.

---

## 7. Practice List (after finishing the 15 above)
| Level | Problem |
|---|---|
| Easy | Count Primes (LC 204), Happy Number (LC 202), Power of Two/Three (LC 231/326), Plus One (LC 66) |
| Easy | Excel Sheet Column Number (LC 171), Add Digits (LC 258) |
| Medium | Pow(x, n) (LC 50), Sqrt(x) (LC 69), Factorial Trailing Zeroes (LC 172) |
| Medium | Unique Paths (LC 62, combinatorics), Max Points on a Line (LC 149, gcd slope) |
| Medium | Ugly Number II (LC 264), Count Primes in a Range (segmented sieve) |
| Hard | Super Pow (LC 372), Number of Ways to Reorder Array to Get Same BST (LC 1569) |
| Hard | Count Different Palindromic Subsequences, Fibonacci Nth with mod (matrix) |
| CodeChef / CF | Chef & Divisors, Codeforces 1000-1500 rated number theory problems |

---

