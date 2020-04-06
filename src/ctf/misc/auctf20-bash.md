# AUCTF20 Bash

This write-up contains 5 challenges that builds on top of each other.

## Bash 1

### Problem

SSH into the server

`ssh challenges.auctf.com -p 30040 -l level1`

password: `aubie`

### Solution

```bash
$ ssh challenges.auctf.com -p 30040 -l level1
level1@challenges.auctf.com's password:
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-91-generic x86_64)
< REDACTED >
$ ls -al
total 24
dr-xr-xr-x 1 root   root   4096 Apr  5 03:19 .
drwxr-xr-x 1 root   root   4096 Apr  4 22:16 ..
-rw-r--r-- 1 level1 level1  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 level1 level1 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 level1 level1  807 Apr  4  2018 .profile
-rw-rw-r-- 1 root   root     24 Apr  1 21:25 README
$ cat README
auctf{W3lcoM3_2_da_C7F}
```

### Flag

`auctf{W3lcoM3_2_da_C7F}`

## Bash 2

### Problem

SSH into the server

`ssh challenges.auctf.com -p 30040 -l level2`

password is the flag of the previous Bash challenge

### Solution

```bash
$ ls -al
total 28
dr-xr-xr-x 1 root   root   4096 Apr  5 03:19 .
drwxr-xr-x 1 root   root   4096 Apr  4 22:16 ..
-rw-r--r-- 1 level2 level2  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 level2 level2 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 level2 level2  807 Apr  4  2018 .profile
-r--r----- 1 level3 level3   22 Apr  1 21:25 flag.txt
-r-xr-x--- 1 level3 level2  110 Apr  1 21:25 random_dirs.sh
$ cat flag.txt
cat: flag.txt: Permission denied
$ cat random_dirs.sh
#!/bin/bash

x=$RANDOM

base64 flag.txt > /tmp/$x
function finish {
        rm  /tmp/$x
}
trap finish EXIT

sleep 15
```

The `flag` is owned by user `level3` and is in group `level2`, which is the group of my user. The flag is only readable by user `level3`.

The bash script under the correct user will be able to read the flag and place it into a worldwide readable file in `/tmp`.

Let's take a snapshot of the `/tmp` directory:

```bash
$ ls /tmp/
111  12  12183  3865  alf.sh  flag.txt  hello  hi  passcodes.sh  prova.sh
```

Let's run the script and throw it into the background:

```bash
$ sudo -u level3 ./random_dirs.sh
^Z[2] + Stopped                    sudo -u level3 ./random_dirs.sh
```

Let's view `/tmp` to see if any files were added:

```bash
$ ls /tmp/
111  12  12183  3865  8037  alf.sh  flag.txt  hello  hi  passcodes.sh  prova.sh
$ cat /tmp/8037
YXVjdGZ7ZzB0dEBfbXV2X2Zhczd9Cg==
```

It's a base64 string:

```bash
$ cat /tmp/8037 | base64 -d
auctf{g0tt@_muv_fas7}
```

### Flag

`auctf{g0tt@_muv_fas7}`

## Bash 3

### Problem

SSH into the server

`ssh challenges.auctf.com -p 30040 -l level3`

password is the flag to the previous Bash challenge

### Solution

```bash
$ ls -al
total 28
dr-xr-xr-x 1 root   root   4096 Apr  5 03:19 .
drwxr-xr-x 1 root   root   4096 Apr  4 22:16 ..
-rw-r--r-- 1 level3 level3  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 level3 level3 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 level3 level3  807 Apr  4  2018 .profile
-r--r----- 1 level4 level4   30 Apr  1 21:25 flag.txt
-r-xr-x--- 1 level4 level3  179 Apr  1 21:25 passcodes.sh
$ cat passcodes.sh
#!/bin/bash

x=$RANDOM
echo "Input the random number."
read input

if [[ "$input" -eq "$x" ]]
then
        echo "AWESOME sauce"
        cat flag.txt
else
        echo "$input"
        echo "$x try again"
fi
```

Similar file structure to last time. The script tries to ask you to guess a random number.

Bruteforce:

```bash
$ bash -c 'for i in {0..30000}; do echo i | sudo -u level4 ./passcodes.sh; done | grep -e "AWESOME" -e "auctf"'
# 5 minute later after brute forcing with fingers crossed there is an overlap
auctf{wut_r_d33z_RaNdom_numz}
```

### Flag

`auctf{wut_r_d33z_RaNdom_numz}`

## Bash 4

### Problem

SSH into the server

`ssh challenges.auctf.com -p 30040 -l level4`

### Solution

```bash
$ ls -al
total 28
dr-xr-xr-x 1 root   root   4096 Apr  5 03:19 .
drwxr-xr-x 1 root   root   4096 Apr  4 22:16 ..
-rw-r--r-- 1 level4 level4  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 level4 level4 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 level4 level4  807 Apr  4  2018 .profile
-r--r----- 1 level5 level5   25 Apr  1 21:25 flag.txt
-r-xr-x--- 1 level5 level4  209 Apr  1 21:25 print_file.sh
$ cat print_file.sh
#!/bin/bash

if [ ! -z "$@" ]
then
        cat $@ # 2>/dev/null
        # if [ ! $? -eq 0 ]
        # then
        #       echo "Printing error. Check file permissions"
        # fi
else
        echo "Please enter a file."
        echo "./print_file FILENAME"
fi
$ sudo -u level5 ./print_file.sh flag.txt
auctf{FunKy_P3rm1ssi0nZ}
```

Nothing new here.

### Flag

`auctf{FunKy_P3rm1ssi0nZ}`

## Bash 5

### Problem

`ssh challenges.auctf.com -p 30040 -l level5`

password is the previous Bash challenge flag

### Solution

```bash
$ ls -al
total 28
dr-xr-xr-x 1 root   root   4096 Apr  5 03:19 .
drwxr-xr-x 1 root   root   4096 Apr  4 22:16 ..
-rw-r--r-- 1 level5 level5  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 level5 level5 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 level5 level5  807 Apr  4  2018 .profile
-r--r----- 1 root   root     23 Apr  1 21:25 flag.txt
-r-xr-x--- 1 root   level5  137 Apr  1 21:25 portforce.sh
$ cat portforce.sh
#!/bin/bash

x=$(shuf -i 1024-65500 -n 1)
echo "Guess the listening port"
input=$(nc -lp $x)
echo "That was easy right? :)"
cat flag.txt
```

It seems like the script opens `netcat` listener and waits for it to close before printing the flag. Let's verify:

```bash
$ sudo -u root ./portforce.sh
Guess the listening port
```

It hangs there. We need to determine the port it's listening on.

The command `ps -ef` will show all running commands:

```bash
$ echo $$
13413
$ ps -ef | grep -e $$
level5   13413 13289  0 22:48 pts/23   00:00:00 -sh
level5   27145 13413  0 22:51 pts/23   00:00:00 ps -ef
level5   27146 13413  0 22:51 pts/23   00:00:00 grep -e 13413
```

Great, so let's open a second window and run the listener, then run the same command above again but filter for nc instead of the UID.

```bash
$ ps -ef | grep -e "nc"
level5    8866 19459  0 22:46 pts/3    00:00:00 nc localhost 23862
level5   12703 12692  0 22:51 pts/5    00:00:00 nc -lp 3830
level5   17378 17373  0 22:51 pts/19   00:00:00 nc -lp 54316
level5   19271 13413  0 22:52 pts/23   00:00:00 grep -e nc
root     30322 30307  0 22:52 pts/22   00:00:00 nc -lp 13177
level5   32391 32386  0 22:49 pts/25   00:00:00 nc -lp 64438
$ nc localhost 13177
^C
```

It's port `13177` this time. Let's go back to the netcat listener window. Looks like it exited:

```bash
$ sudo -u root ./portforce.sh
Guess the listening port
That was easy right? :)
auctf{n3tc@_purt_$can}
```

### Flag

`auctf{n3tc@_purt_$can}`