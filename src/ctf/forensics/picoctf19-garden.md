# PicoCTF19 Garden

## Challenge

This [garden](https://2019shell1.picoctf.com/static/3e1f573c9b18e87b6764ec3a36868b5a/garden.jpg) contains more than it seems. You can also find the file in /problems/glory-of-the-garden_0_25ece79ae00914856938a4b19d0e31af on the shell server.

![garden.jpg](assets/garden.jpg "Garden.jpg")

## Hints

What is a hex editor?

## Solution

Open up a hex editor and search for pico...

```bash
$ strings garden.jpg | grep -a pico
Here is a flag "picoCTF{more_than_m33ts_the_3y30cAf8c6B}"
```

Also works

## Flag

`picoCTF{more_than_m33ts_the_3y3f089EdF0}`