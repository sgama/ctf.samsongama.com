# PicoCTF19 like1000

## Challenge

This [.tar](https://2019shell1.picoctf.com/static/8694f84879d3b7c0dcf775930f4665fc/1000.tar) file got tarred alot. Also available at /problems/like1000_0_369bbdba2af17750ddf10cc415672f1c.

## Hints

Try and script this, it'll save you alot of time

## Solution

Assuming it was tar-ed 1000x. The file inside is 999.tar. Countdown

```bash
#!/bin/bash
for i in {1000..1}
do
   tar -xvf $i.tar
   rm $i.tar
done
```

or

```python
import tarfile

for i in range(1000,0,-1):
    tarfile.open(str(i) + '.tar').extractall()
```

## Flag

`picoCTF{StaT31355_636f6e6e}`
