# Python RegEx — Notes

A **RegEx (Regular Expression)** is a sequence of characters that forms a search pattern, used to check if a string contains a specified pattern.

## The `re` Module

Python has a built-in module called `re` for working with regular expressions.

```python
import re
```

**Basic example** — search a string to see if it starts with "The" and ends with "Spain":
```python
import re

txt = "The rain in Spain"
x = re.search("^The.*Spain$", txt)
```

---

## RegEx Functions

| Function | Description |
|---|---|
| `findall` | Returns a list containing all matches |
| `search` | Returns a **Match object** if there is a match anywhere in the string |
| `split` | Returns a list where the string has been split at each match |
| `sub` | Replaces one or many matches with a string |

---

## Metacharacters

Characters with a special meaning:

| Character | Description | Example |
|---|---|---|
| `[]` | A set of characters | `"[a-m]"` |
| `\` | Signals a special sequence (can also escape special characters) | `"\d"` |
| `.` | Any character (except newline) | `"he..o"` |
| `^` | Starts with | `"^hello"` |
| `$` | Ends with | `"planet$"` |
| `*` | Zero or more occurrences | `"he.*o"` |
| `+` | One or more occurrences | `"he.+o"` |
| `?` | Zero or one occurrences | `"he.?o"` |
| `{}` | Exactly the specified number of occurrences | `"he.{2}o"` |
| `\|` | Either or | `"falls\|stays"` |
| `()` | Capture and group | — |

---

## Flags

Add flags to a pattern to change matching behavior:

| Flag | Shorthand | Description |
|---|---|---|
| `re.ASCII` | `re.A` | Returns only ASCII matches |
| `re.DEBUG` | — | Returns debug information |
| `re.DOTALL` | `re.S` | Makes `.` match all characters, including newline |
| `re.IGNORECASE` | `re.I` | Case-insensitive matching |
| `re.MULTILINE` | `re.M` | Returns matches at the start/end of each line |
| `re.NOFLAG` | — | Specifies that no flag is set |
| `re.UNICODE` | `re.U` | Returns Unicode matches (default in Python 3) |
| `re.VERBOSE` | `re.X` | Allows whitespace/comments inside patterns for readability |

---

## Special Sequences

A `\` followed by a special character:

| Sequence | Description | Example |
|---|---|---|
| `\A` | Match if specified characters are at the beginning of the string | `"\AThe"` |
| `\b` | Match where specified characters are at the beginning or end of a word (use raw string `r"..."`) | `r"\bain"`, `r"ain\b"` |
| `\B` | Match where specified characters are present, but NOT at the beginning/end of a word | `r"\Bain"`, `r"ain\B"` |
| `\d` | Match where the string contains digits (0–9) | `"\d"` |
| `\D` | Match where the string does NOT contain digits | `"\D"` |
| `\s` | Match where the string contains a whitespace character | `"\s"` |
| `\S` | Match where the string does NOT contain a whitespace character | `"\S"` |
| `\w` | Match where the string contains word characters (a-z, A-Z, 0-9, `_`) | `"\w"` |
| `\W` | Match where the string does NOT contain word characters | `"\W"` |
| `\Z` | Match if specified characters are at the end of the string | `"Spain\Z"` |

---

## Sets

A set of characters inside `[]`:

| Set | Description |
|---|---|
| `[arn]` | Match if any of `a`, `r`, or `n` is present |
| `[a-n]` | Match any lowercase char alphabetically between `a` and `n` |
| `[^arn]` | Match any character EXCEPT `a`, `r`, and `n` |
| `[0123]` | Match if any of `0`, `1`, `2`, or `3` is present |
| `[0-9]` | Match any digit between `0` and `9` |
| `[0-5][0-9]` | Match any two-digit number from `00` to `59` |
| `[a-zA-Z]` | Match any char alphabetically between `a` and `z`, lower or upper case |
| `[+]` | Inside sets, `+ * . \| () $ {}` lose special meaning — `[+]` just matches a literal `+` |

---

## `findall()` — Get All Matches

Returns a list of all matches (in the order found), or an empty list if none found.

```python
import re

txt = "The rain in Spain"
x = re.findall("ai", txt)
print(x)   # ['ai', 'ai']
```

```python
x = re.findall("Portugal", txt)
print(x)   # []  (empty list — no match)
```

---

## `search()` — Get First Match

Returns a **Match object** for the first occurrence, or `None` if no match.

```python
import re

txt = "The rain in Spain"
x = re.search("\s", txt)
print("The first white-space character is located in position:", x.start())
```

```python
x = re.search("Portugal", txt)
print(x)   # None
```

---

## `split()` — Split at Matches

Returns a list where the string has been split at each match.

```python
import re

txt = "The rain in Spain"
x = re.split("\s", txt)
print(x)   # ['The', 'rain', 'in', 'Spain']
```

Control the number of splits with `maxsplit`:
```python
x = re.split("\s", txt, 1)
print(x)   # ['The', 'rain in Spain']  (split only at first match)
```

---

## `sub()` — Replace Matches

Replaces matches with a string of your choice.

```python
import re

txt = "The rain in Spain"
x = re.sub("\s", "9", txt)
print(x)   # The9rain9in9Spain
```

Control the number of replacements with `count`:
```python
x = re.sub("\s", "9", txt, 2)
print(x)   # The9rain9in Spain  (only first 2 replaced)
```

---

## Match Object

An object containing information about the search and its result. If there's no match, `None` is returned instead.

```python
import re

txt = "The rain in Spain"
x = re.search("ai", txt)
print(x)   # prints a Match object
```

**Useful Match object members:**
| Member | Description |
|---|---|
| `.span()` | Tuple of (start, end) positions of the match |
| `.string` | The original string passed into the function |
| `.group()` | The part of the string where the match occurred |

**Examples** (pattern finds a word starting with uppercase "S"):
```python
import re

txt = "The rain in Spain"
x = re.search(r"\bS\w+", txt)

print(x.span())    # (12, 18)
print(x.string)     # The rain in Spain
print(x.group())    # Spain
```

---

## Quick Reference Cheat Sheet

| Goal | Code |
|---|---|
| Check if pattern exists | `re.search(pattern, txt)` |
| Get all matches as a list | `re.findall(pattern, txt)` |
| Split string on pattern | `re.split(pattern, txt)` |
| Replace matches | `re.sub(pattern, replacement, txt)` |
| Case-insensitive match | `re.search(pattern, txt, re.IGNORECASE)` |
| Get match position | `match.span()` |
| Get matched text | `match.group()` |

---
*Source: [W3Schools — Python RegEx](https://www.w3schools.com/python/python_regex.asp)*
