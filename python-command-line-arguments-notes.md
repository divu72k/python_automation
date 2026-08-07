# Command Line Arguments in Python — Notes

Command line arguments are values passed to a Python script at runtime (from terminal/CMD) that let you change program behavior without editing code.

```
python add.py 10 20
```
- `python` → runs the interpreter
- `add.py` → script name
- `10 20` → arguments passed to the program

Python offers **3 ways** to handle them: `sys`, `getopt`, and `argparse`.

---

## 1. `sys.argv` (simplest method)

- `sys.argv` — list containing all command-line arguments
- `sys.argv[0]` — always the script name
- `sys.argv[1:]` — the actual arguments passed by the user

**Example:**
```python
import sys

print("Total arguments:", len(sys.argv))
print("Script name:", sys.argv[0])
print("Arguments:", sys.argv[1:])

total = 0
for arg in sys.argv[1:]:
    total += int(arg)
print("Sum =", total)
```
Run: `python add.py 5 10 15`
Output:
```
Total arguments: 4
Script name: add.py
Arguments: ['5', '10', '15']
Sum = 30
```
👉 Good for quick scripts, but no built-in flag/option handling.

---

## 2. `getopt` module

Used for Unix-style flags — short (`-h`) and long (`--help`) options.

```python
import getopt, sys

args = sys.argv[1:]
options = "hmo:"                             # -o expects a value (the colon)
long_options = ["Help", "My_file", "Output="]

try:
    arguments, values = getopt.getopt(args, options, long_options)
    for currentArg, currentVal in arguments:
        if currentArg in ("-h", "--Help"):
            print("Showing Help")
        elif currentArg in ("-m", "--My_file"):
            print("File name:", sys.argv[0])
        elif currentArg in ("-o", "--Output"):
            print("Output mode:", currentVal)
except getopt.error as err:
    print(str(err))
```
Run: `python script.py -o result.txt` → Output: `Output mode: result.txt`

Key points:
- `options = "hmo:"` → defines short flags (`:` after `o` means it takes a value)
- `long_options` → long flag equivalents
- `getopt.getopt()` → parses args into (option, value) pairs
- `try/except` → catches invalid options

---

## 3. `argparse` module (most powerful & recommended)

Features:
- Auto-generates help messages
- Supports default values, type checking, optional/required args
- Built-in `-h`/`--help` flag

**Basic parser:**
```python
import argparse
parser = argparse.ArgumentParser()
parser.parse_args()
```
`python script.py -h` → shows default usage/help text.

**With a description:**
```python
parser = argparse.ArgumentParser(description="Demo of argparse")
parser.parse_args()
```
The description text shows up in the `-h` output.

**Adding an optional argument:**
```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("-o", "--Output", help="Show output message")
args = parser.parse_args()

if args.Output:
    print("Output:", args.Output)
```
Run: `python script.py -o Hello` → Output: `Output: Hello`

- `add_argument()` defines short (`-o`) and long (`--Output`) forms
- `help=` text appears in usage message
- `args.Output` accesses the passed value

---

## Quick Comparison

| Module | Best for | Complexity |
|---|---|---|
| `sys.argv` | Simple positional args | Low |
| `getopt` | Unix-style flags/options | Medium |
| `argparse` | Full-featured CLI tools with help, defaults, type checks | Low effort, high power |

---
*Source: [GeeksforGeeks - Command Line Arguments in Python](https://www.geeksforgeeks.org/python/command-line-arguments-in-python/)*
