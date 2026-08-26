# Pwntools (Python)

CTF/exploit-dev toolkit — process interaction, networking, packing, and shellcode helpers in one library.

## Setup
- install: `pip install pwntools`
- import everything: `from pwn import *`
- set target architecture/os (affects packing, shellcraft, etc.): `context.arch = 'amd64'`, `context.os = 'linux'`
- logging verbosity: `context.log_level = 'debug'` (or `'info'`, `'warn'`)

## Connecting to a target
- local process: `p = process("./binary")`
- remote host: `p = remote("target.com", 1337)`
- attach gdb to a local process for debugging: `gdb.attach(p)`
- SSH into a box and run something there: `s = ssh(host="target", user="user", password="pw"); p = s.process("./binary")`

## Sending / receiving data
- send raw bytes: `p.send(b"data")`
- send bytes + newline: `p.sendline(b"data")`
- receive until a delimiter: `p.recvuntil(b"prompt: ")`
- receive a fixed number of bytes: `p.recv(1024)`
- receive one line: `p.recvline()`
- send then wait for a prompt in one call: `p.sendlineafter(b"prompt: ", b"payload")`
- drop into an interactive session (manual control in terminal): `p.interactive()`

## Packing / unpacking values
- pack an int to bytes (respects `context.arch`/endianness): `p32(value)` / `p64(value)`
- unpack bytes back to an int: `u32(data)` / `u64(data)`
- pad/align data: `flat(a, b, c)` — concatenates and auto-packs mixed values (ints, bytes, etc.)

## Payload / exploit-primitive helpers
- cyclic pattern for finding offsets (e.g. buffer overflow crash offset): `cyclic(200)`
- find the offset of a value within a cyclic pattern: `cyclic_find(0x61616161)`
- generate shellcode for the current arch: `shellcraft.sh()` (assembles to raw shellcode via `asm(shellcraft.sh())`)
- assemble/disassemble: `asm("nop; nop; ret")`, `disasm(bytes_data)`

## Binary/ELF introspection
- load a binary for inspection: `elf = ELF("./binary")`
- get a function/symbol address: `elf.symbols['main']`
- get the PLT/GOT entry for a function: `elf.plt['puts']`, `elf.got['puts']`
- check enabled protections: `elf.checksec()` (also available as CLI: `checksec ./binary`)

## Leaking / defeating ASLR
- once you leak a libc address, load a matching libc to compute offsets: `libc = ELF("./libc.so.6"); libc.address = leaked_base`
- compute an absolute address from a leaked base + known offset: `libc.symbols['system'] + libc.address`

## Misc
- one-shot gadget lookup / cross-referencing typically pairs with `one_gadget` (separate tool) or `ROPgadget`
- building ROP chains: `rop = ROP(elf); rop.call('system', [elf.got['puts']])` then `rop.chain()` for the raw bytes
- CLI companion for quick lookups without writing a script: `pwn checksec`, `pwn cyclic 100`, `pwn shellcraft`

**Import:** `from pwn import *`
