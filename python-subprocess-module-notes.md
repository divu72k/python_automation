# `subprocess` — Subprocess Management (Python) — Notes

The `subprocess` module lets you **spawn new processes**, connect to their input/output/error pipes, and get their return codes. It's meant to replace older modules/functions like `os.system` and the `os.spawn*` family.

- **Availability:** not Android, not iOS, not WASI (not supported on mobile/WebAssembly)
- Source: `Lib/subprocess.py`
- Related: PEP 324 (the proposal for this module)

---

## 1. `subprocess.run()` — the recommended high-level API

Use `run()` for almost all use cases. For advanced needs, use `Popen` directly.

```python
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None,
                capture_output=False, shell=False, cwd=None, timeout=None,
                check=False, encoding=None, errors=None, text=None,
                env=None, universal_newlines=None, **other_popen_kwargs)
```

Returns a **`CompletedProcess`** instance once the command finishes.

### Key parameters
| Parameter | Purpose |
|---|---|
| `capture_output=True` | Captures both stdout & stderr (internally sets both to `PIPE`). Can't be combined with manually setting `stdout`/`stderr`. |
| `timeout` | Seconds to wait; kills process and raises `TimeoutExpired` if exceeded. |
| `input` | Data sent to the subprocess's stdin (bytes, or str if `text`/`encoding` set). Implicitly sets `stdin=PIPE`. |
| `check=True` | Raises `CalledProcessError` if exit code is non-zero. |
| `encoding` / `errors` / `text=True` | Opens stdin/stdout/stderr in **text mode** instead of default binary mode. `text` is the modern alias for `universal_newlines`. |
| `env` | Dict of environment variables to use *instead of* inheriting the parent's environment. |

### Examples
```python
subprocess.run(["ls", "-l"])                     # doesn't capture output

subprocess.run("exit 1", shell=True, check=True)  # raises CalledProcessError

subprocess.run(["ls", "-l", "/dev/null"], capture_output=True)
# CompletedProcess(args=[...], returncode=0, stdout=b'...', stderr=b'')
```

> ⚠️ Changed in 3.12: On Windows, `shell=True` now searches `%COMSPEC%` / `System32\cmd.exe` instead of cwd + `%PATH%` (prevents a malicious local `cmd.exe` from being picked up).

---

## 2. `CompletedProcess` object (return value of `run()`)

| Attribute | Meaning |
|---|---|
| `args` | The command that was run |
| `returncode` | Exit status. `0` = success. Negative `-N` = killed by signal `N` (POSIX only) |
| `stdout` | Captured stdout (bytes or str), `None` if not captured |
| `stderr` | Captured stderr (bytes or str), `None` if not captured. If `stderr=STDOUT` was used, this is `None` and output is merged into `stdout` |
| `check_returncode()` | Raises `CalledProcessError` if `returncode != 0` |

---

## 3. Special values & exceptions

**Special stream values:**
- `subprocess.PIPE` — open a new pipe to the stream
- `subprocess.DEVNULL` — redirect to `os.devnull`
- `subprocess.STDOUT` — merge stderr into stdout

**Exceptions (all inherit from `SubprocessError`):**
- `TimeoutExpired` — raised when a timeout expires. Has `.cmd`, `.timeout`, `.output`/`.stdout`, `.stderr`
- `CalledProcessError` — raised (with `check=True`, or by `check_call`/`check_output`) on non-zero exit. Has `.returncode`, `.cmd`, `.output`/`.stdout`, `.stderr`
- Plain `OSError` — raised when, e.g., trying to execute a non-existent file (most common exception)
- `ValueError` — raised for invalid `Popen` arguments

---

## 4. Frequently used `Popen`/`run` arguments (deep dive)

- **`args`**: string or sequence of program args. **Prefer a sequence** — it handles escaping/quoting automatically (e.g., spaces in filenames). A single string is only OK if `shell=True`, or it just names the program with no arguments.
- **`stdin`/`stdout`/`stderr`**: accept `None` (no redirection), `PIPE`, `DEVNULL`, a file descriptor, or a file object. `stderr` can also be `STDOUT`.
- **`shell=True`**: runs the command through the system shell (`/bin/sh` on POSIX, `cmd.exe`/`COMSPEC` on Windows). Gives access to shell features (pipes, wildcards, env-var expansion, `~` expansion) — but read Security Considerations first (shell injection risk).
- Use `shlex.split()` to correctly tokenize a shell command string into a list for `args`.

---

## 5. The `Popen` class (low-level, full control)

```python
subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None,
                  stderr=None, preexec_fn=None, close_fds=True, shell=False,
                  cwd=None, env=None, universal_newlines=None, startupinfo=None,
                  creationflags=0, restore_signals=True, start_new_session=False,
                  pass_fds=(), *, group=None, extra_groups=None, user=None,
                  umask=-1, encoding=None, errors=None, text=None,
                  pipesize=-1, process_group=None)
```

Notable parameters:
- **`bufsize`**: `0`=unbuffered, `1`=line-buffered (text mode only), positive=buffer size, negative (default)=system default.
- **`executable`**: rarely needed — overrides the actual program run while `args` still supplies the displayed command name.
- **`close_fds`**: if `True` (default), closes all FDs except 0/1/2 in the child.
- **`cwd`**: working directory for the child process.
- **`start_new_session`** / **`process_group`**: POSIX-only session/group control (preferred over `preexec_fn` + `os.setsid()`/`os.setpgid()`).
- **`group` / `extra_groups` / `user` / `umask`**: POSIX-only privilege/identity controls for the child (added in 3.9).
- **`pipesize`**: change pipe buffer size (Linux only, added 3.10).

⚠️ **`preexec_fn` is NOT thread-safe** — can deadlock. Not supported in subinterpreters (3.8+, raises `RuntimeError`).

**Context manager support:**
```python
with Popen(["ifconfig"], stdout=PIPE) as proc:
    log.write(proc.stdout.read())
```
On exit: closes standard FDs and waits for the process.

**Auditing event:** `subprocess.Popen` is raised with `executable`, `args`, `cwd`, `env`.

---

## 6. Security Considerations

- Without `shell=True`, subprocess does **not** invoke a system shell — shell metacharacters are passed safely as literal arguments.
- With `shell=True`, **you** are responsible for quoting/escaping to avoid **shell injection**. Use `shlex.quote()` on POSIX-like platforms.
- On Windows, `.bat`/`.cmd` files may be run through a shell by the OS regardless of your settings — be cautious launching batch files with untrusted arguments.

---

## 7. Popen instance methods & attributes

| Method | Description |
|---|---|
| `poll()` | Checks if child has terminated; sets/returns `returncode` (or `None` if still running) |
| `wait(timeout=None)` | Blocks until child terminates; raises `TimeoutExpired` if it doesn't finish in time. ⚠️ Can deadlock with `PIPE` if child fills the OS pipe buffer — use `communicate()` instead |
| `communicate(input=None, timeout=None)` | Sends `input` to stdin, reads stdout/stderr until EOF, waits for exit. Returns `(stdout_data, stderr_data)`. Safe against the deadlock issue above. Buffers all data in memory — avoid for very large output |
| `send_signal(signal)` | Sends a signal to the child |
| `terminate()` | Sends `SIGTERM` (POSIX) or calls `TerminateProcess()` (Windows) |
| `kill()` | Sends `SIGKILL` (POSIX); alias for `terminate()` on Windows |

**Handling a timeout properly:**
```python
proc = subprocess.Popen(...)
try:
    outs, errs = proc.communicate(timeout=15)
except TimeoutExpired:
    proc.kill()
    outs, errs = proc.communicate()
```

**Attributes:** `args`, `stdin`, `stdout`, `stderr` (stream objects if `PIPE` was used, else `None`), `pid`, `returncode` (`None` until process ends; negative = killed by signal `N`).

---

## 8. Older high-level API (pre-3.5, still supported)

| Function | Behavior |
|---|---|
| `call(args, ...)` | Runs command, waits, returns `returncode`. Equivalent to `run(...).returncode` |
| `check_call(args, ...)` | Like `call()`, but raises `CalledProcessError` on non-zero exit. Equivalent to `run(..., check=True)` |
| `check_output(args, ...)` | Runs command, returns captured output; raises `CalledProcessError` on non-zero exit. Equivalent to `run(..., check=True, stdout=PIPE).stdout` |

⚠️ Don't use `stdout=PIPE`/`stderr=PIPE` with `call()`/`check_call()` — the child can block if it fills the OS pipe buffer since nothing reads from it.

`check_output` example capturing stderr too:
```python
subprocess.check_output("ls non_existent_file; exit 0", stderr=subprocess.STDOUT, shell=True)
```

---

## 9. Replacing older functions with `subprocess`

| Old style | New style |
|---|---|
| `` output=$(mycmd myarg) `` | `output = check_output(["mycmd", "myarg"])` |
| `` output=$(dmesg \| grep hda) `` | Build two `Popen`s and pipe manually (see below), or `check_output("dmesg | grep hda", shell=True)` for trusted input |
| `os.system(cmd)` | `call(cmd, shell=True)` |
| `os.spawnlp(os.P_NOWAIT, ...)` | `Popen([...]).pid` |
| `os.spawnlp(os.P_WAIT, ...)` | `call([...])` |
| `os.popen(cmd, 'w')` | `Popen(cmd, stdin=PIPE)` + `.stdin.close()` + check `.wait()` |

**Manual shell pipeline replacement:**
```python
p1 = Popen(["dmesg"], stdout=PIPE)
p2 = Popen(["grep", "hda"], stdin=p1.stdout, stdout=PIPE)
p1.stdout.close()  # let p1 receive SIGPIPE if p2 exits first
output = p2.communicate()[0]
```

---

## 10. Legacy shell invocation functions

These **implicitly invoke the shell** — none of the standard security/exception guarantees apply.

- `getstatusoutput(cmd)` → returns `(exitcode, output)`, trailing newline stripped
- `getoutput(cmd)` → like above but ignores exit code, returns just the output string

```python
subprocess.getstatusoutput('ls /bin/ls')       # (0, '/bin/ls')
subprocess.getoutput('ls /bin/ls')              # '/bin/ls'
```
Available on Unix and Windows.

---

## 11. Notes & gotchas

**Timeout behavior:**
- Process *creation* itself can't be interrupted on many platforms — a timeout exception isn't guaranteed until after process creation completes.
- Very small timeout values may trigger near-instant `TimeoutExpired` because creation/scheduling inherently takes time.

**Windows argument-to-string conversion** (when passing a sequence as `args`):
1. Arguments are space/tab delimited.
2. Double-quoted strings count as one argument, spaces and all.
3. `\"` = literal double-quote.
4. Backslashes are literal unless right before a `"`.
5. Backslash-pairs before a `"` collapse to literal backslashes; an odd number escapes the quote.
→ See `shlex` module for parsing/escaping help.

**Disabling `posix_spawn()`:**
- On Linux, subprocess prefers `vfork()`-based creation for performance.
- Can be disabled via the private, undocumented knob: `subprocess._USE_POSIX_SPAWN = False`

---

## Quick Reference Cheat Sheet

| Task | Recommended call |
|---|---|
| Run and wait, don't care about output | `subprocess.run(["cmd", "arg"])` |
| Run and capture stdout+stderr | `subprocess.run([...], capture_output=True, text=True)` |
| Run and raise on failure | `subprocess.run([...], check=True)` |
| Run and get just stdout as text | `subprocess.check_output([...], text=True)` |
| Send input to a process | `subprocess.run([...], input="data", text=True)` |
| Fine-grained control (streaming, signals) | `subprocess.Popen(...)` + `.communicate()` |

---
*Source: [Python Docs — subprocess: Subprocess management](https://docs.python.org/3/library/subprocess.html)*
