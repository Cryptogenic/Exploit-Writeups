## crackme01 (RE, 75pts)

#### Challenge Description

this is crackme01. crackme01 is a crackme. so crackme! Author: @X3eRo0

#### Overview

This challenge was pretty simple, it was just a bunch of `printf()`'s that would build the string, each one behind a check against a byte of the input.

```c
printf("en");
...
printf("cry");
if ( v7 != 65 )
  exit(0);
printf("ptC");
if ( v8 != 32 )
  exit(0);
printf("TF{");
if ( v9 != 33 )
  exit(0);
printf("gdb");
if ( v10 != 101 )
  exit(0);
printf("_or");
if ( v11 != 25 )
  exit(0);
printf("_r2?");
if ( v12 != 9 )
  exit(0);
puts("}");
```

Flag:

```
encryptCTF{gdb_or_r2?}
```