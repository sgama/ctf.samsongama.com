# PicoCTF19 OverFlow 2

## Challenge

Now try overwriting arguments. Can you get the flag from this [program](https://2019shell1.picoctf.com/static/2b7b3583b687134589781c4e22ef5760/vuln)? You can find it in /problems/overflow-2 on the shell server. [Source](https://2019shell1.picoctf.com/static/2b7b3583b687134589781c4e22ef5760/vuln.c).

## Hints

GDB can print the stack after you send arguments

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/overflow-2$ ls -al
total 92
drwxr-xr-x   2 root       root          4096 Sep 28 22:04 .
drwxr-x--x 684 root       root         69632 Oct 10 18:02 ..
-r--r-----   1 hacksports overflow-2_3    33 Sep 28 22:04 flag.txt
-rwxr-sr-x   1 hacksports overflow-2_3  7500 Sep 28 22:04 vuln
-rw-rw-r--   1 hacksports hacksports     794 Sep 28 22:04 vuln.c
```

Let's view `vuln.c`

```
samson@pico-2019-shell1:/problems/overflow-2$ cat vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 176
#define FLAGSIZE 64

void flag(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xDEADBEEF)
    return;
  if (arg2 != 0xC0DED00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```
