# PicoCTF19 Lies Within

## Challenge

Theres something in the [building](https://2019shell1.picoctf.com/static/aec3861fc4d5bce4d39dc0db196426de/buildings.png). Can you retrieve the flag?

![buildings.png](assets/buildings.png "buildings.png")

## Hints

There is data encoded somewhere, there might be an online decoder

## Solution

Opened up in hexedit, checked for `pico` -> no results

```bash
$ strings buildings.png | grep -a pico
```

Used this online tool: [https://stylesuxx.github.io/steganography/](https://stylesuxx.github.io/steganography/)

## Flag

`picoCTF{h1d1ng_1n_th3_b1t5}`