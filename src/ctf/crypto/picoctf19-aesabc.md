# PicoCTF19 AES-ABC

## Challenge

AES-ECB is bad, so I rolled my own cipher block chaining mechanism - Addition Block Chaining! You can find the source here: [aes-abc.py](https://2019shell1.picoctf.com/static/f8b82da049bf23cb598c4bdac73207e8/aes-abc.py). The AES-ABC flag is [body.enc.ppm](https://2019shell1.picoctf.com/static/f8b82da049bf23cb598c4bdac73207e8/body.enc.ppm)

aes-abc.py
```python
#!/usr/bin/env python

from Crypto.Cipher import AES
from key import KEY
import os
import math

BLOCK_SIZE = 16
UMAX = int(math.pow(256, BLOCK_SIZE))


def to_bytes(n):
    s = hex(n)
    s_n = s[2:]
    if 'L' in s_n:
        s_n = s_n.replace('L', '')
    if len(s_n) % 2 != 0:
        s_n = '0' + s_n
    decoded = s_n.decode('hex')

    pad = (len(decoded) % BLOCK_SIZE)
    if pad != 0: 
        decoded = "\0" * (BLOCK_SIZE - pad) + decoded
    return decoded


def remove_line(s):
    # returns the header line, and the rest of the file
    return s[:s.index('\n') + 1], s[s.index('\n')+1:]


def parse_header_ppm(f):
    data = f.read()

    header = ""

    for i in range(3):
        header_i, data = remove_line(data)
        header += header_i

    return header, data
        

def pad(pt):
    padding = BLOCK_SIZE - len(pt) % BLOCK_SIZE
    return pt + (chr(padding) * padding)


def aes_abc_encrypt(pt):
    cipher = AES.new(KEY, AES.MODE_ECB)
    ct = cipher.encrypt(pad(pt))

    blocks = [ct[i * BLOCK_SIZE:(i+1) * BLOCK_SIZE] for i in range(len(ct) / BLOCK_SIZE)]
    iv = os.urandom(16)
    blocks.insert(0, iv)
    
    for i in range(len(blocks) - 1):
        prev_blk = int(blocks[i].encode('hex'), 16)
        curr_blk = int(blocks[i+1].encode('hex'), 16)

        n_curr_blk = (prev_blk + curr_blk) % UMAX
        blocks[i+1] = to_bytes(n_curr_blk)

    ct_abc = "".join(blocks)
 
    return iv, ct_abc, ct


if __name__=="__main__":
    with open('flag.ppm', 'rb') as f:
        header, data = parse_header_ppm(f)
    
    iv, c_img, ct = aes_abc_encrypt(data)

    with open('body.enc.ppm', 'wb') as fw:
        fw.write(header)
        fw.write(c_img)
```

## Hints

You probably want to figure out what the flag looks like in ECB form...

## Solution

Let's take a look at how it was encrypted:
```python
def aes_abc_encrypt(pt):
    cipher = AES.new(KEY, AES.MODE_ECB)
    ct = cipher.encrypt(pad(pt))

    blocks = [ct[i * BLOCK_SIZE:(i+1) * BLOCK_SIZE] for i in range(len(ct) / BLOCK_SIZE)]
    iv = os.urandom(16)
    blocks.insert(0, iv)
    
    for i in range(len(blocks) - 1):
        prev_blk = int(blocks[i].encode('hex'), 16)
        curr_blk = int(blocks[i+1].encode('hex'), 16)

        n_curr_blk = (prev_blk + curr_blk) % UMAX
        blocks[i+1] = to_bytes(n_curr_blk)

    ct_abc = "".join(blocks)
 
    return iv, ct_abc, ct
```

Uhoh, it's in ECB (Electronic Codebook) mode which encrypts data 16 bits at a time.

But instead of XOR-ing the ECB blocks, it seems to be adding them. Weird, did they roll their own crypto algorithm?

Let's first convert all the ciphertext blocks back to ecb blocks.

Let's create an `aes_abc_decrypt(ct):
```python
def aes_abc_decrypt(c_img):
    blocks = [c_img[i * BLOCK_SIZE:(i+1) * BLOCK_SIZE] for i in range(len(c_img) / BLOCK_SIZE)]

    for i in range(len(blocks) - 2, -1, -1):
        n_curr_blk = from_bytes(blocks[i+1])
        n_prev_blk = from_bytes(blocks[i])

        curr_blk = (n_curr_blk - n_prev_blk) % UMAX

        blocks[i+1] = to_bytes(curr_blk)

    ct = ''.join(blocks[1:])

    return ct
```

Final script is in assets

```
$ file decrypt.ppm
decrypt.ppm: Netpbm image data, size = 1895 x 820, rawbits, pixmap
```

Use this [converter](https://cloudconvert.com/ppm-to-pdf)

Looks like garbage maybe it's not the correct size... perhaps brute force necessary.

## Flag

``