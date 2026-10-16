# 🔤 Strings in DSA (Python) — Theory, Patterns, Algorithms & 15 Problems

> Covers: how strings work in Python, every important method, string patterns (two pointers, sliding window, hashing, stack, DP, expand-around-center, trie), pattern-matching algorithms (Naive, KMP, Rabin–Karp, Z), and 15 standard problems from easy to hard with commented code and verified output.

---

## 📑 Table of Contents
1. [What is a String?](#1-what-is-a-string)
2. [Python String Basics & Immutability](#2-python-string-basics--immutability)
3. [Important String Methods](#3-important-string-methods)
4. [Complexity Table & Python Gotchas](#4-complexity-table--python-gotchas)
5. [Character Tricks (ASCII, frequency arrays)](#5-character-tricks)
6. [String Patterns (Techniques)](#6-string-patterns-techniques)
7. [Pattern Matching Algorithms](#7-pattern-matching-algorithms)
8. [15 Standard Problems](#8-15-standard-problems-easy--hard)
9. [Pattern Cheat Sheet](#9-pattern-cheat-sheet)
10. [Mistakes, Tips & Practice List](#10-mistakes-tips--practice-list)

---

## 1. What is a String?

A **string** is a sequence of characters. In memory it is like an array of characters: `"HELLO"` → `H E L L O` at indexes `0..4`.

Most string problems are **array problems on characters**, so every array idea (two pointers, sliding window, prefix, hashing, sorting) applies. The extra string-specific topics are:
- immutability and building strings efficiently,
- character arithmetic (`ord`, `chr`),
- substrings / subsequences / palindromes / anagrams,
- pattern matching (KMP, Rabin–Karp),
- prefix structures (Trie).

**Key vocabulary**
| Term | Meaning | Example (`s = "abcde"`) |
|---|---|---|
| Substring | contiguous part | `"bcd"` |
| Subsequence | order kept, may skip characters | `"ace"` |
| Prefix / Suffix | starts at 0 / ends at the end | `"abc"` / `"cde"` |
| Palindrome | reads same forwards and backwards | `"madam"` |
| Anagram | same characters, different order | `"listen"` ↔ `"silent"` |

> Count: a string of length `n` has **n(n+1)/2 substrings** and **2ⁿ subsequences**.

---

## 2. Python String Basics & Immutability

**Strings are immutable in Python**: you cannot change a character in place. Every "modification" creates a **new string** (O(n)).

```python
s = "hello world"
print(s[0], s[-1])                 # first and last character
print(s[0:5])                      # slice [start:stop) -> 'hello'
print(s[::-1])                     # reversed copy using step -1
print(len(s))                      # length
try:
    s[0] = 'H'                     # trying to modify a character
except TypeError as e:
    print("Error:", e)             # strings do not support item assignment
s2 = 'H' + s[1:]                   # correct way: build a NEW string
print(s2)
chars = list(s)                    # convert to a list to modify characters
chars[0] = 'J'                     # lists ARE mutable
print(''.join(chars))              # join the list back into a string
print("a" < "b", "apple" < "apricot", "Z" < "a")  # comparison is lexicographic (ASCII order)
```
**Output**
```
h d
hello
dlrow olleh
11
Error: 'str' object does not support item assignment
Hello world
Jello world
True True True
```

---

## 3. Important String Methods

```python
s = "  Hello, World!  "
print(s.strip())                          # remove leading/trailing whitespace
print(s.strip().lower())                  # lowercase
print(s.strip().upper())                  # uppercase
t = "a,b,,c"
print(t.split(","))                       # split by separator -> list
print("  many   spaces here ".split())    # split() with no argument splits on any whitespace and drops empties
print("-".join(["2026", "11", "12"]))     # join list of strings with a separator
print("banana".replace("a", "o"))         # replace all occurrences
print("banana".find("na"))                # index of first match, -1 if missing
print("banana".find("x"))                 # not found -> -1 (index() would raise ValueError)
print("banana".count("a"))                # non-overlapping occurrences
print("banana".startswith("ban"), "banana".endswith("na"))
print("abc123".isalpha(), "123".isdigit(), "abc123".isalnum())   # character-class checks
print("a-b-c".partition("-"))             # (before, separator, after) around first separator
print("42".zfill(5))                      # pad with zeros on the left
print(f"{3.14159:.2f} | {'x':>4} | {7:03d}")   # f-string formatting: 2 decimals, right-align, zero-pad
print("abc"*3)                            # repetition
print("b" in "abc", "ac" in "abc")        # 'in' checks a SUBSTRING (contiguous)
print(sorted("hello"))                    # sorted() on a string -> list of characters
print("".join(sorted("hello")))           # sorted string
print(list(reversed("abc")))              # reversed() returns an iterator
```
**Output**
```
Hello, World!
hello, world!
HELLO, WORLD!
['a', 'b', '', 'c']
['many', 'spaces', 'here']
2026-11-12
bonono
2
-1
3
True True
False True True
('a', '-', 'b-c')
00042
3.14 |    x | 007
abcabcabc
True False
['e', 'h', 'l', 'l', 'o']
ehllo
['c', 'b', 'a']
```

**Most used in interviews:** `len, split, join, strip, lower, isalnum, isdigit, find, count, startswith, ord, chr, sorted, Counter`.

---

## 4. Complexity Table & Python Gotchas

| Operation | Time |
|---|---|
| `s[i]` | O(1) |
| `len(s)` | O(1) |
| Slice `s[a:b]` | O(b − a) (creates a copy) |
| `s + t` | O(len(s) + len(t)) |
| `s == t` | O(min length) |
| `x in s` (substring search) | O(n·m) worst case |
| `s.find`, `s.count`, `s.replace` | O(n·m) worst case |
| `"".join(list)` | O(total length) |
| `sorted(s)` | O(n log n) |

**Gotchas**
1. **`s += ch` inside a loop** can become O(n²) because each concatenation copies the string. Collect pieces in a **list** and `''.join()` at the end.
2. **Slicing inside loops** (`s[i:]`) copies each time → hidden O(n²).
3. `str.split()` and `str.split(" ")` are different (the second keeps empty strings).
4. Comparing characters: `'a' < 'b'` works, but compare **case-insensitively** with `.lower()` first when needed.
5. In C++/Java strings have different mutability rules; in Python convert to `list` when you need to swap characters in place.

---

## 5. Character Tricks

```python
print(ord('a'), ord('A'), ord('0'))       # character -> ASCII code: 97 65 48
print(chr(98), chr(65))                   # ASCII code -> character
print(ord('d') - ord('a'))                # position of a letter in the alphabet (0-based) = 3
print(chr(ord('a') + 25))                 # 25th letter after 'a' = 'z'
count = [0] * 26                          # frequency array for lowercase letters (faster than dict)
for ch in "banana":
    count[ord(ch) - ord('a')] += 1        # map 'a'->0, 'b'->1, ... 'z'->25
print(count[:3], count[13])               # a, b, c counts and 'n' (index 13)
from collections import Counter
print(Counter("banana"))                  # Counter works for any characters
print("5".isdigit(), int("5") + 1)        # char digit to int
print(ord('7') - ord('0'))                # digit char -> integer without int()
```
**Output**
```
97 65 48
b A
3
z
[3, 1, 0] 2
Counter({'a': 3, 'n': 2, 'b': 1})
True 6
7
```

**Memory trick:** `ord(c) - ord('a')` gives a 0..25 index. For all ASCII use `[0] * 128` or `[0] * 256`.

---

## 6. String Patterns (Techniques)

### Pattern 1 — Two Pointers
Use `left` and `right` at both ends moving inward (palindromes, reverse) or two pointers moving in the same direction (compare two strings, remove characters).
**Problems:** 1 (Valid Palindrome).

### Pattern 2 — Frequency Counting / Hashing
Count characters with an array of 26 / `Counter` / dict. Two strings are anagrams iff their counts match. Group by a canonical **key** (sorted string or count tuple).
**Problems:** 2, 3, 6.

### Pattern 3 — Sliding Window
Maintain a window `[left, right]` over the string with a frequency map; expand `right`; shrink `left` while the window is invalid (or while it is still valid for "shortest" problems).
**Template**
```py
left = 0
for right, ch in enumerate(s):
    add ch to window
    while window is invalid:
        remove s[left]; left += 1
    update answer with (right - left + 1)
```
**Problems:** 8, 11, 15.

### Pattern 4 — Expand Around Center
For palindromic substrings: every palindrome has a center (a character for odd length, a gap between two characters for even length). Expand outward while characters match. O(n²), simple, no extra memory.
**Problems:** 9. (Manacher's algorithm does it in O(n).)

### Pattern 5 — Stack
Nested/matching structures: parentheses, decode string, remove adjacent duplicates, simplify path.
**Problems:** 5, 10.

### Pattern 6 — Dynamic Programming on Strings
`dp[i][j]` = answer for prefix `s[:i]` and `t[:j]`. Used for LCS, edit distance, palindromic subsequence, regex/wildcard matching, word break.
**Problems:** 12, 13.

### Pattern 7 — Pattern Matching (KMP / Rabin–Karp / Z)
Find where a pattern occurs in a text in O(n + m) instead of O(n·m). See Section 7.
**Problems:** 7.

### Pattern 8 — Trie (Prefix Tree)
A tree where each edge is a character; shared prefixes share a path. Gives O(L) insert/search/prefix-search, and powers autocomplete, word search, longest-common-prefix.
**Problems:** 14.

### Pattern 9 — Sorting / Canonical Form
Sort characters (or the strings) so equal-looking items become identical (anagram groups, largest number).

### Pattern recognition table
| Clue | Technique |
|---|---|
| "palindrome" | two pointers / expand around center / DP |
| "anagram", "permutation of", "same characters" | frequency count |
| "longest substring with …" / "shortest window …" | sliding window |
| "valid parentheses", "nested", "k[encoded]" | stack |
| "subsequence", "edit", "transform one string to another" | DP |
| "find pattern in text" | KMP / Rabin–Karp / built-in `find` |
| "prefix", "dictionary of words", "autocomplete" | Trie |
| "group strings by …" | hash map with canonical key |

---

## 7. Pattern Matching Algorithms

**Problem:** find all positions where `pat` (length m) occurs in `text` (length n).

### 7.1 Naive Search — O(n·m)
Slide the pattern over every position and compare. Simple, fine for small inputs.

```python
def naive_search(text, pat):
    n, m = len(text), len(pat)
    res = []                                   # starting indexes of matches
    for i in range(n - m + 1):                 # every possible alignment
        if text[i:i + m] == pat:               # compare the window with the pattern
            res.append(i)
    return res

print(naive_search("abababa", "aba"))          # overlapping matches are found too
print(naive_search("hello", "xyz"))
```
**Output**
```
[0, 2, 4]
[]
```

### 7.2 KMP (Knuth–Morris–Pratt) — O(n + m)
**Idea:** When a mismatch occurs, do not restart from scratch. Use the **LPS array** ("longest proper prefix which is also a suffix") of the pattern to know how far we can safely fall back.

- `lps[i]` = length of the longest proper prefix of `pat[:i+1]` that is also a suffix of it.
- Example `pat = "ABABCABAB"` → `lps = [0,0,1,2,0,1,2,3,4]`.

**Search logic:** compare `text[i]` and `pat[j]`. Match → advance both. Mismatch with `j > 0` → `j = lps[j-1]` (text pointer `i` never moves back). Full match → record `i - j`, then `j = lps[j-1]`.

```python
def build_lps(pat):
    lps = [0] * len(pat)                       # lps[i] for every prefix of the pattern
    length = 0                                 # length of the current matching prefix-suffix
    i = 1                                      # lps[0] is always 0
    while i < len(pat):
        if pat[i] == pat[length]:              # extends the current prefix-suffix
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:                       # mismatch: fall back to a shorter prefix-suffix
            length = lps[length - 1]           # (do NOT advance i)
        else:                                  # no prefix-suffix possible here
            lps[i] = 0
            i += 1
    return lps

def kmp_search(text, pat):
    if not pat:
        return []
    lps = build_lps(pat)                       # preprocess pattern in O(m)
    res = []
    i = j = 0                                  # i walks the text, j walks the pattern
    while i < len(text):
        if text[i] == pat[j]:                  # characters match -> advance both
            i += 1
            j += 1
            if j == len(pat):                  # whole pattern matched
                res.append(i - j)              # record start index
                j = lps[j - 1]                 # continue looking for overlapping matches
        elif j > 0:                            # mismatch after some matches
            j = lps[j - 1]                     # reuse already matched prefix, don't move i back
        else:                                  # mismatch at pattern start
            i += 1                             # just move forward in the text
    return res

print(build_lps("ABABCABAB"))
print(kmp_search("ABABDABACDABABCABAB", "ABABCABAB"))
print(kmp_search("aaaaa", "aa"))               # overlapping matches: 0,1,2,3
```
**Output**
```
[0, 0, 1, 2, 0, 1, 2, 3, 4]
[10]
[0, 1, 2, 3]
```

### 7.3 Rabin–Karp (Rolling Hash) — O(n + m) average
**Idea:** Convert each window into a **hash number**; slide the window by removing the leading character and adding the new one in O(1). Only if hashes match, verify by direct comparison (to avoid false positives).

`hash(window) = Σ ord(c_k) · base^(m-1-k) mod p`

```python
def rabin_karp(text, pat, base=256, mod=10**9 + 7):
    n, m = len(text), len(pat)
    if m > n:
        return []
    h = pow(base, m - 1, mod)                  # weight of the leading character in a window
    ph = th = 0                                # pattern hash, current text window hash
    for i in range(m):                         # hash of the pattern and of the first window
        ph = (ph * base + ord(pat[i])) % mod
        th = (th * base + ord(text[i])) % mod
    res = []
    for i in range(n - m + 1):
        if ph == th and text[i:i + m] == pat:  # equal hashes -> verify to rule out collisions
            res.append(i)
        if i < n - m:                          # roll the window one step to the right
            th = ((th - ord(text[i]) * h) * base + ord(text[i + m])) % mod
    return res

print(rabin_karp("GEEKS FOR GEEKS", "GEEK"))
```
**Output**
```
[0, 10]
```
**Use:** multiple-pattern search, plagiarism detection, finding duplicate substrings (rolling hash + binary search).

### 7.4 Z-Algorithm — O(n + m)
`z[i]` = length of the longest substring starting at `i` that matches a prefix of `s`. To search, build `pat + "#" + text` and look for `z[i] == len(pat)`.

```python
def z_function(s):
    n = len(s)
    z = [0] * n
    l = r = 0                                  # [l, r) = rightmost segment matching a prefix
    for i in range(1, n):
        if i < r:                              # inside a known matching segment: reuse info
            z[i] = min(r - i, z[i - l])
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:   # extend the match character by character
            z[i] += 1
        if i + z[i] > r:                       # update the rightmost segment
            l, r = i, i + z[i]
    return z

print(z_function("aabxaab"))
pat, text = "aab", "xaabxaab"
z = z_function(pat + "#" + text)               # '#' must not appear in pat/text
print([i - len(pat) - 1 for i in range(len(z)) if z[i] == len(pat)])  # start indexes in text
```
**Output**
```
[0, 1, 0, 0, 3, 1, 0]
[1, 5]
```

### 7.5 Other algorithms (know by name)
| Algorithm | Purpose |
|---|---|
| **Manacher** | longest palindromic substring in O(n) |
| **Suffix Array / Suffix Tree** | many substring queries, longest repeated substring |
| **Aho–Corasick** | search many patterns at once |
| **Boyer–Moore** | fast practical search (skips using bad-character rule) |

---

## 8. 15 Standard Problems (Easy → Hard)

---

### 🟢 Problem 1 — Valid Palindrome (Easy) · *Two pointers*
**Idea:** Ignore non-alphanumeric characters and case. Move `l` forward and `r` backward past junk, compare lowercase characters.

```python
def is_palindrome(s):
    l, r = 0, len(s) - 1                       # pointers at both ends
    while l < r:
        while l < r and not s[l].isalnum():    # skip punctuation/spaces on the left
            l += 1
        while l < r and not s[r].isalnum():    # skip punctuation/spaces on the right
            r -= 1
        if s[l].lower() != s[r].lower():       # compare ignoring case
            return False
        l += 1                                 # move inward
        r -= 1
    return True

print(is_palindrome("A man, a plan, a canal: Panama"))
print(is_palindrome("race a car"))
print(is_palindrome(" "))                      # empty after cleaning -> palindrome
```
**Output**
```
True
False
True
```
**Complexity:** O(n) time, O(1) space.

---

### 🟢 Problem 2 — Valid Anagram (Easy) · *Frequency array*
**Idea:** Two strings are anagrams iff every letter count matches. Increment for `s`, decrement for `t`; all zeros at the end means equal.

```python
def is_anagram(s, t):
    if len(s) != len(t):                       # different lengths can never be anagrams
        return False
    count = [0] * 26                           # letter balance: +1 for s, -1 for t
    for a, b in zip(s, t):
        count[ord(a) - ord('a')] += 1          # letter from s
        count[ord(b) - ord('a')] -= 1          # letter from t
    return all(c == 0 for c in count)          # every letter balanced

print(is_anagram("anagram", "nagaram"))
print(is_anagram("rat", "car"))
```
**Output**
```
True
False
```
**Complexity:** O(n) time, O(1) space (26 slots). For Unicode use `Counter`.

---

### 🟢 Problem 3 — First Unique Character in a String (Easy) · *Counting*
**Idea:** First pass: count frequencies. Second pass: return the first index whose count is 1.

```python
from collections import Counter

def first_uniq_char(s):
    freq = Counter(s)                          # pass 1: count every character
    for i, ch in enumerate(s):                 # pass 2: scan in original order
        if freq[ch] == 1:                      # first character that appears exactly once
            return i
    return -1                                  # no unique character

print(first_uniq_char("leetcode"))
print(first_uniq_char("loveleetcode"))
print(first_uniq_char("aabb"))
```
**Output**
```
0
2
-1
```
**Complexity:** O(n) time, O(1) space for lowercase letters.

---

### 🟢 Problem 4 — Longest Common Prefix (Easy) · *Vertical scan*
**Idea:** Compare column by column. Stop at the first index where any string differs or ends.

```python
def longest_common_prefix(strs):
    if not strs:
        return ""
    first = strs[0]                            # use the first word as the reference
    for i in range(len(first)):                # column index
        for s in strs[1:]:
            if i == len(s) or s[i] != first[i]:  # other word is too short or character differs
                return first[:i]               # prefix ends right before this column
    return first                               # the entire first word is a common prefix

print(longest_common_prefix(["flower", "flow", "flight"]))
print(longest_common_prefix(["dog", "racecar", "car"]))
print(longest_common_prefix(["abc"]))
```
**Output**
```
fl

abc
```
**Complexity:** O(S) where S = total number of characters. O(1) extra space.

---

### 🟡 Problem 5 — Valid Parentheses (Medium) · *Stack*
**Idea:** Push opening brackets. On a closing bracket, the top of the stack must be its matching opener. At the end the stack must be empty.

```python
def is_valid(s):
    pairs = {')': '(', ']': '[', '}': '{'}     # closing -> matching opening
    stack = []
    for ch in s:
        if ch in pairs:                        # closing bracket
            if not stack or stack[-1] != pairs[ch]:  # nothing to match or wrong type
                return False
            stack.pop()                        # matched, remove the opener
        else:
            stack.append(ch)                   # opening bracket: wait for its partner
    return not stack                           # leftover openers -> invalid

print(is_valid("()[]{}"))
print(is_valid("(]"))
print(is_valid("([{}])"))
print(is_valid("(("))
```
**Output**
```
True
False
True
False
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 6 — Group Anagrams (Medium) · *Hash map with canonical key*
**Idea:** All anagrams share the same letter-count signature. Use the **count tuple** as the dictionary key (O(n·k) instead of sorting each word O(n·k log k)).

```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)                 # signature -> list of words
    for word in strs:
        count = [0] * 26                       # letter counts of this word
        for ch in word:
            count[ord(ch) - ord('a')] += 1
        groups[tuple(count)].append(word)      # lists aren't hashable, tuples are
    return list(groups.values())

print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
print(group_anagrams([""]))
```
**Output**
```
[['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
[['']]
```
**Complexity:** O(N·K) time (N words of length K), O(N·K) space.

---

### 🟡 Problem 7 — Find the Index of the First Occurrence (strStr) (Medium) · *KMP*
**Idea:** Return the first index of `needle` in `haystack` using the KMP search from Section 7.2, stopping at the first match.

```python
def build_lps(pat):
    lps, length, i = [0] * len(pat), 0, 1      # lps array, current border length, index
    while i < len(pat):
        if pat[i] == pat[length]:              # border grows
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:                       # fall back to a shorter border
            length = lps[length - 1]
        else:
            i += 1                             # lps[i] stays 0
    return lps

def str_str(haystack, needle):
    if not needle:                             # empty needle matches at 0
        return 0
    lps = build_lps(needle)
    j = 0                                      # matched length in needle
    for i, ch in enumerate(haystack):          # i never moves backward
        while j > 0 and ch != needle[j]:       # mismatch: shrink the matched prefix
            j = lps[j - 1]
        if ch == needle[j]:                    # extend the match
            j += 1
        if j == len(needle):                   # full match found
            return i - j + 1                   # start index
    return -1

print(str_str("sadbutsad", "sad"))
print(str_str("leetcode", "leeto"))
print(str_str("hello", "ll"))
```
**Output**
```
0
-1
2
```
**Complexity:** O(n + m) time, O(m) space. (Python's `haystack.find(needle)` is the practical choice, KMP is what interviewers want to see.)

---

### 🟡 Problem 8 — Longest Substring Without Repeating Characters (Medium) · *Sliding window*
**Idea:** Keep the last seen index of every character. When `s[right]` was seen inside the current window, jump `left` just past that previous index.

```python
def length_of_longest_substring(s):
    last = {}                                  # character -> last index where it appeared
    left = 0                                   # window start
    best = 0
    for right, ch in enumerate(s):
        if ch in last and last[ch] >= left:    # repeated character inside the current window
            left = last[ch] + 1                # shrink window to just after the old occurrence
        last[ch] = right                       # record newest position
        best = max(best, right - left + 1)     # window [left, right] is valid here
    return best

print(length_of_longest_substring("abcabcbb"))
print(length_of_longest_substring("bbbbb"))
print(length_of_longest_substring("pwwkew"))
print(length_of_longest_substring(""))
```
**Output**
```
3
1
3
0
```
**Complexity:** O(n) time, O(min(n, alphabet)) space.

---

### 🟡 Problem 9 — Longest Palindromic Substring (Medium) · *Expand around center*
**Idea:** Try each index as the center of an **odd** palindrome and each gap as the center of an **even** palindrome, expanding while characters match. Keep the longest.

```python
def longest_palindrome(s):
    start = end = 0                            # boundaries of the best palindrome found
    def expand(l, r):                          # expand outward while it stays a palindrome
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1
            r += 1
        return l + 1, r - 1                    # last valid boundaries
    for i in range(len(s)):
        for l, r in (expand(i, i), expand(i, i + 1)):   # odd center, even center
            if r - l > end - start:            # longer than the best so far
                start, end = l, r
    return s[start:end + 1]

print(longest_palindrome("babad"))             # "bab" ("aba" is also valid)
print(longest_palindrome("cbbd"))
print(longest_palindrome("a"))
```
**Output**
```
bab
bb
a
```
**Complexity:** O(n²) time, O(1) space. (Manacher = O(n).)

---

### 🟡 Problem 10 — Decode String (Medium) · *Stack*
**Idea:** `k[encoded]` → repeat `encoded` k times. Keep a stack of `(string built so far, repeat count)`. On `[` push and reset; on `]` pop and combine.

```python
def decode_string(s):
    stack = []                                 # saves (previous string, repeat count) at each '['
    cur = ''                                   # string being built at the current nesting level
    num = 0                                    # number being read (can have several digits)
    for ch in s:
        if ch.isdigit():
            num = num * 10 + int(ch)           # build multi-digit numbers like 12
        elif ch == '[':
            stack.append((cur, num))           # save the outer context
            cur, num = '', 0                   # start fresh inside the brackets
        elif ch == ']':
            prev, k = stack.pop()              # restore outer context
            cur = prev + cur * k               # repeat inner string k times and attach
        else:
            cur += ch                          # normal letter
    return cur

print(decode_string("3[a]2[bc]"))
print(decode_string("3[a2[c]]"))
print(decode_string("2[abc]3[cd]ef"))
```
**Output**
```
aaabcbc
accaccacc
abcabccdcdcdef
```
**Complexity:** O(output length) time and space.

---

### 🔴 Problem 11 — Longest Repeating Character Replacement (Medium-Hard) · *Sliding window with max frequency*
**Idea:** A window is valid if `window_size - (count of most frequent char) <= k` (we can replace the others). Never shrink more than one step; `max_freq` only needs to *increase* to improve the answer.

```python
def character_replacement(s, k):
    count = [0] * 26                           # letter counts inside the window
    left = 0
    max_freq = 0                               # highest frequency seen in any window so far
    best = 0
    for right, ch in enumerate(s):
        idx = ord(ch) - ord('A')
        count[idx] += 1                        # add right character
        max_freq = max(max_freq, count[idx])   # update most frequent letter count
        while (right - left + 1) - max_freq > k:   # more than k characters need replacing
            count[ord(s[left]) - ord('A')] -= 1    # drop left character
            left += 1
        best = max(best, right - left + 1)     # valid window length
    return best

print(character_replacement("ABAB", 2))
print(character_replacement("AABABBA", 1))
```
**Output**
```
4
4
```
**Complexity:** O(n) time, O(1) space.

---

### 🔴 Problem 12 — Longest Common Subsequence (Medium-Hard) · *2D DP*
**Idea:** `dp[i][j]` = LCS length of `a[:i]` and `b[:j]`.
- If `a[i-1] == b[j-1]` → `dp[i][j] = dp[i-1][j-1] + 1`
- else → `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]      # extra row/column for empty prefixes
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:                # last characters match -> extend diagonal
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:                                   # skip a character from a or from b
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]

print(lcs("abcde", "ace"))
print(lcs("abc", "abc"))
print(lcs("abc", "def"))
```
**Output**
```
3
3
0
```
**Complexity:** O(m·n) time and space (space can be reduced to O(min(m, n)) with two rows). Related: Longest Palindromic Subsequence = LCS(s, reverse(s)).

---

### 🔴 Problem 13 — Edit Distance (Hard) · *2D DP*
**Idea:** Minimum insert / delete / replace operations to convert `word1` → `word2`.
`dp[i][j]` = cost for `word1[:i]` → `word2[:j]`.
- Equal last chars → `dp[i-1][j-1]`.
- Else `1 + min(dp[i-1][j]` (delete), `dp[i][j-1]` (insert), `dp[i-1][j-1]` (replace)`)`.

```python
def edit_distance(w1, w2):
    m, n = len(w1), len(w2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i                                # w1[:i] -> "" needs i deletions
    for j in range(n + 1):
        dp[0][j] = j                                # "" -> w2[:j] needs j insertions
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if w1[i - 1] == w2[j - 1]:              # characters equal -> no operation needed
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j],    # delete from w1
                                   dp[i][j - 1],    # insert into w1
                                   dp[i - 1][j - 1])  # replace a character
    return dp[m][n]

print(edit_distance("horse", "ros"))
print(edit_distance("intention", "execution"))
print(edit_distance("", "abc"))
```
**Output**
```
3
5
3
```
**Complexity:** O(m·n) time, O(m·n) space (two rows possible).

---

### 🔴 Problem 14 — Implement Trie (Prefix Tree) (Medium-Hard) · *Trie*
**Idea:** Each node has children keyed by character and a flag `is_end` marking a complete word. Insert/search/prefix all walk down one character at a time → O(L).

```python
class TrieNode:
    def __init__(self):
        self.children = {}                     # character -> TrieNode
        self.is_end = False                    # True if a word ends at this node

class Trie:
    def __init__(self):
        self.root = TrieNode()                 # empty root node

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:        # create the path if it doesn't exist
                node.children[ch] = TrieNode()
            node = node.children[ch]           # go one level down
        node.is_end = True                     # mark the end of a complete word

    def _walk(self, s):                        # follow s from the root; None if the path breaks
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node

    def search(self, word):                    # exact word exists?
        node = self._walk(word)
        return node is not None and node.is_end

    def starts_with(self, prefix):             # any word starts with prefix?
        return self._walk(prefix) is not None

t = Trie()
t.insert("apple")
print(t.search("apple"))                       # inserted word
print(t.search("app"))                         # only a prefix so far
print(t.starts_with("app"))
t.insert("app")
print(t.search("app"))                         # now it is a word
```
**Output**
```
True
False
True
True
```
**Complexity:** O(L) per operation, O(total characters) space.

---

### 🔴 Problem 15 — Minimum Window Substring (Hard) · *Sliding window + counts*
**Idea:** Find the shortest substring of `s` containing all characters of `t` (with multiplicity). Expand `right` to make the window valid; then shrink `left` as far as it stays valid, record, and repeat.

- `need[c]` = how many more of `c` we still need (can go negative = surplus).
- `missing` = how many required characters are still missing in total.

```python
from collections import Counter

def min_window(s, t):
    if not t or len(t) > len(s):
        return ""
    need = Counter(t)                          # required count for each character
    missing = len(t)                           # total required characters still missing
    left = 0
    best = (float('inf'), 0, 0)                # (length, start, end)
    for right, ch in enumerate(s):
        if need[ch] > 0:                       # this character was still required
            missing -= 1
        need[ch] -= 1                          # take it (negative = surplus)
        if missing == 0:                       # window contains everything from t
            while need[s[left]] < 0:           # drop surplus characters from the left
                need[s[left]] += 1
                left += 1
            if right - left + 1 < best[0]:     # record a better window
                best = (right - left + 1, left, right)
            need[s[left]] += 1                 # remove s[left] to break validity and look for the next window
            missing += 1
            left += 1
    return "" if best[0] == float('inf') else s[best[1]:best[2] + 1]

print(min_window("ADOBECODEBANC", "ABC"))
print(min_window("a", "a"))
print(min_window("a", "aa"))
```
**Output**
```
BANC
a
```
**Complexity:** O(n + m) time, O(alphabet) space.

---

## 9. Pattern Cheat Sheet

| # | Problem | Pattern | Complexity |
|---|---|---|---|
| 1 | Valid Palindrome | two pointers | O(n) |
| 2 | Valid Anagram | frequency array | O(n) |
| 3 | First Unique Character | counting | O(n) |
| 4 | Longest Common Prefix | vertical scan | O(S) |
| 5 | Valid Parentheses | stack | O(n) |
| 6 | Group Anagrams | hash map + canonical key | O(N·K) |
| 7 | strStr | KMP | O(n+m) |
| 8 | Longest Substring Without Repeating | sliding window + last index | O(n) |
| 9 | Longest Palindromic Substring | expand around center | O(n²) |
| 10 | Decode String | stack | O(output) |
| 11 | Longest Repeating Char Replacement | sliding window + max freq | O(n) |
| 12 | Longest Common Subsequence | 2D DP | O(mn) |
| 13 | Edit Distance | 2D DP | O(mn) |
| 14 | Implement Trie | trie | O(L) per op |
| 15 | Minimum Window Substring | sliding window + counts | O(n+m) |

**Which technique when? (quick decision)**
```
Palindrome?            → two pointers / expand center
Anagram / counts?      → frequency array / Counter
Substring condition?   → sliding window
Brackets / nesting?    → stack
Two strings compared?  → DP table
Many words + prefix?   → Trie
Find pattern in text?  → KMP / .find()
```

---

## 10. Mistakes, Tips & Practice List

### Common mistakes
1. **Modifying a string in place** (`s[i] = 'x'`) → convert to `list`, edit, then `''.join`.
2. **Building strings with `+=` in a loop** → use a list and `join`.
3. **Off-by-one in slices**: `s[a:b]` excludes `b`.
4. Forgetting **case and non-alphanumeric characters** in palindrome problems.
5. **Empty string / single character** edge cases (`""`, `"a"`).
6. Using `str.index` when the substring may be missing (raises `ValueError`) → use `find`.
7. Sliding window: updating the answer **before** the window is valid.
8. In DP tables, forgetting the **extra row/column** for the empty prefix.
9. Assuming only lowercase letters. Check constraints before using `[0] * 26`.

### Interview talking points
- "Strings are immutable in Python, so repeated concatenation is O(n²); I'll collect pieces in a list and join."
- "For anagram checks I use a 26-length count array: O(n) time, O(1) space."
- "KMP avoids re-comparing characters using the LPS array → O(n + m)."
- "Sliding window works when expanding/shrinking changes validity monotonically."

### Practice list
| Level | Problems |
|---|---|
| Easy | Reverse String (344), Valid Palindrome II (680), Isomorphic Strings (205), Ransom Note (383), Roman to Integer (13), Add Binary (67), Implement strStr (28), Is Subsequence (392) |
| Medium | Longest Palindromic Substring (5), Palindromic Substrings (647), Group Anagrams (49), String to Integer (8), Permutation in String (567), Find All Anagrams (438), Minimum Remove to Make Valid Parentheses (1249), Generate Parentheses (22), Word Break (139), Longest Palindromic Subsequence (516), Reorganize String (767), Zigzag Conversion (6) |
| Hard | Minimum Window Substring (76), Edit Distance (72), Regular Expression Matching (10), Wildcard Matching (44), Longest Valid Parentheses (32), Palindrome Pairs (336), Shortest Palindrome (214), Distinct Subsequences (115) |

### ✅ Final advice for 12 November 2026
- Priority order for revision: **sliding window (8, 11, 15) → DP on strings (12, 13) → stack (5, 10) → KMP & Trie**.
- For each solved problem, say out loud: *brute force → what is repeated → which pattern removes it*.
- You have now covered: **Math → Arrays → Searching → Sorting → Strings**. Next recommended topics: **Hashing, Recursion & Backtracking, Linked List, Stack & Queue, Trees, Graphs, Dynamic Programming**. Tell me which one you want and I'll prepare it in the same format. 🚀
