# PicoCTF19 So Meta

## Challenge

Find the flag in this [picture](https://2019shell1.picoctf.com/static/051b66965016a24bb62e5af66b28cbc0/pico_img.png). You can also find the file in /problems/so-meta_2_da856426d694a4f0637bf1b169d8524e.

![pico_img.png](assets/pico_img.png "pico_img.png")

## Hints

What does meta mean in the context of files?

Ever hear of metadata?

## Solution

Open up the file in a hex editor...

```bash
$ strings pico_img.png | grep -a pico
picoCTF{s0_m3ta_3d6ced35}
```

## Flag

`picoCTF{s0_m3ta_3d6ced35}`