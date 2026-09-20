# 06 · Strings
> **Reach for it when:** the work is palindromes, parsing, or nested structure — small-alphabet array problems plus the discipline interviewers actually probe.

## 🧠 Mental model
String problems are array problems over a small alphabet, but the follow-ups probe parsing discipline. Palindrome checks are two pointers converging; finding the *best* palindrome is trying every center — there are 2n − 1 of them (each character and each gap) — and expanding while the ends match. Parsing tasks (atoi, calculators, run-length decoding) reward an explicit state machine or a stack of frozen contexts, because nesting is where mental recursion breaks. Build strings through lists/stacks and join at the end: treat `s += ch` inside a loop as O(n²) — harmless at n ≤ 10³, fatal at 10⁶. When the interviewer extends a string problem ("integers can be negative", "whitespace is allowed", "digits repeat"), the state machine absorbs the change in one branch instead of a rewrite.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "longest palindromic substring" | expand around 2n − 1 centers |
| "palindrome after deleting at most one char" | two pointers + spend-the-budget retry |
| "convert string to signed 32-bit int" | state machine: skip → sign → digits → clamp |
| "add two binary/decimal strings" | right-to-left carry loop |
| "nested brackets with repeat counts k[...]" | stack of (prefix, count) frozen at '[' |
| "evaluate + − * / expression" | term stack: * and / apply now, − pushes negated (sheet 08) |
| "minimum removals for valid parentheses" | index stack marks offenders, both directions |
| "next lexicographic permutation" | right-to-left scan for the first ascent |
| "restore IPs / split into valid parts" | backtracking over cut positions (sheet 09) |

## 🛠️ Templates

### T1 — Expand-around-center palindrome
Longest palindromic substring without O(n²) DP table.
```python
def longest_palindrome(s):
    def grow(l, r):
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1
            r += 1
        return l + 1, r - l - 1       # overshot by one; step back to the match

    best_start, best_len = 0, 0
    for i in range(len(s)):           # 2n-1 centers: chars and gaps
        for st, ln in (grow(i, i), grow(i, i + 1)):   # odd and even lengths
            if ln > best_len:
                best_start, best_len = st, ln
    return s[best_start:best_start + best_len]
```
O(n²) time, O(1) space (Manacher's O(n) exists — name-drop, don't code).

### T2 — Two-pointer check with one deletion budget
Valid palindrome after removing at most one character.
```python
def valid_palindrome(s):
    def ok(l, r):
        while l < r:
            if s[l] != s[r]:
                return False
            l += 1
            r -= 1
        return True

    l, r = 0, len(s) - 1
    while l < r:
        if s[l] != s[r]:
            return ok(l + 1, r) or ok(l, r - 1)   # spend the single deletion
        l += 1
        r -= 1
    return True
```
O(n) time, O(1) space.

### T3 — atoi-style parse state machine
String to clamped 32-bit integer, spec-exact.
```python
def my_atoi(s):
    INT_MIN, INT_MAX = -2**31, 2**31 - 1
    i, n = 0, len(s)
    while i < n and s[i] == ' ':                  # 1) skip leading spaces
        i += 1
    sign = 1
    if i < n and s[i] in '+-':                   # 2) one optional sign
        sign = -1 if s[i] == '-' else 1
        i += 1
    num = 0
    while i < n and s[i].isdigit():              # 3) digits, clamp as we go
        num = num * 10 + int(s[i])
        if sign == 1 and num > INT_MAX:
            return INT_MAX
        if sign == -1 and -num < INT_MIN:
            return INT_MIN
        i += 1
    return sign * num                            # 4) junk after: ignored
```
O(n) time, O(1) space.

### T4 — String-builder stack (decode-string)
Nested `k[encoded]` decoding via frozen contexts.
```python
def decode_string(s):
    stack = []                # (string_so_far, repeat_count) at each '['
    cur, k = "", 0
    for ch in s:
        if ch.isdigit():
            k = k * 10 + int(ch)          # multi-digit counts
        elif ch == '[':
            stack.append((cur, k))        # freeze outer context
            cur, k = "", 0
        elif ch == ']':
            prev, rep = stack.pop()
            cur = prev + cur * rep        # materialize one nesting level
        else:
            cur += ch
    return cur
```
O(length of decoded output) time, O(nesting depth) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| expand-around-center | O(n²) | O(1) |
| palindrome two-pointer check | O(n) | O(1) |
| one-deletion palindrome | O(n) | O(1) |
| atoi state machine | O(n) | O(1) |
| add-binary carry loop | O(max(n, m)) | O(1) extra |
| decode-string | O(output length) | O(depth) |
| next-permutation scan | O(n) | O(1) |
| restore-ip-addresses | O(1) (≤ 3³ cuts) | O(4) |

## ⚠️ Traps interviewers probe
- Even-length palindromes come from `(i, i + 1)` centers — forget the gap centers and "abba" disappears.
- valid-palindrome-ii: on mismatch, try skip-left OR skip-right — one branch fails inputs where only the other deletion works.
- atoi is a spec-reading test: leading spaces, ONE optional sign, stop at the first non-digit, clamp DURING accumulation ("+-12" → 0, "-91283472332" → −2³¹).
- add-binary: process the final carry after both strings end ("1" + "1" = "10"); loop while either index lives or carry is set.
- decode-string: repeat counts are multi-digit ("100[leetcode]"); accumulate k = k·10 + digit.
- next-permutation: find the rightmost i with nums[i] < nums[i+1]; swap with the rightmost strictly greater element in the suffix; then REVERSE the suffix (it's non-increasing) to get the smallest next arrangement; no ascent → full reverse.
- restore-ip: exactly four parts consuming the whole string; each part 1–3 chars; "0" is legal, "01" is not; values ≤ 255.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Longest Palindromic Substring | Medium | https://leetcode.com/problems/longest-palindromic-substring/ |
| Valid Palindrome | Easy | https://leetcode.com/problems/valid-palindrome/ |
| Valid Palindrome II | Medium | https://leetcode.com/problems/valid-palindrome-ii/ |
| String to Integer (atoi) | Medium | https://leetcode.com/problems/string-to-integer-atoi/ |
| Add Binary | Easy | https://leetcode.com/problems/add-binary/ |
| Decode String | Medium | https://leetcode.com/problems/decode-string/ |
| Basic Calculator II | Medium | https://leetcode.com/problems/basic-calculator-ii/ |
| Minimum Remove to Make Valid Parentheses | Medium | https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/ |
| Next Permutation | Medium | https://leetcode.com/problems/next-permutation/ |
| Restore IP Addresses | Medium | https://leetcode.com/problems/restore-ip-addresses/ |
