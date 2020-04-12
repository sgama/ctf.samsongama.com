# PicoCTF19 leap-frog

## Challenge

Can you jump your way to win in the following [program](https://2019shell1.picoctf.com/static/a7394c555b8cd0a7eac288c45367c49d/rop) and get the flag? You can find the program in /problems/leap-frog_1_2944cde4843abb6dfd6afa31b00c703c on the shell server? [Source](https://2019shell1.picoctf.com/static/a7394c555b8cd0a7eac288c45367c49d/rop.c).

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

TODO