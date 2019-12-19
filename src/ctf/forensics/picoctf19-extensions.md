# PicoCTF19 Extensions

## Challenge

This is a really weird text file [TXT](https://2019shell1.picoctf.com/static/45886ed4b6d5d1dc74c4944fcf4b4041/flag.txt)? Can you find the flag?


## Hints

How do operating systems know what kind of file it is? (It's not just the ending!

Make sure to submit the flag as picoCTF{XXXXX}

## Solution

```bash
$ file flag.txt
flag.txt: PNG image data, 1697 x 608, 8-bit/color RGB, non-interlaced
$ cp flag.txt flag_extensions.png
```

Open the file and see the flag.

## Flag

`picoCTF{now_you_know_about_extensions}`