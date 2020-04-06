# AUCTF20 Extraordinary

## Problem

On their way back from the market, Alice and Bob noticed a little device on the ground. Next to it was a piece of paper with what looked like a bunch of scrambled numbers on it. It looked completely random. They took it to the lost and found, but on their way they played with it a little bit (don't tell anyone!). The device was never picked up, so we get to play with it a little bit, too. Can you figure out how the device works?

`b'6\x1d\x0cT*\x12\x18V\x05\x13c1R\x07u#\x021Jq\x05\x02n\x03t%1\\\x04@V7P\\\x17aN'`

`nc challenges.auctf.com 30030`

## Solution

We are given a byte array that appears to be the hex representation of a string. However there are some illegal characters.

Let's connect to the service and do some investigation:

```bash
$ nc challenges.auctf.com 30030
> a
b''
> aa
b'\x14'
> aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
b'\x14\x02\x15\x07\x1a\x0fR\x17R3>\x13R4\x12R>\x18Q\x143>Q5\x11>YVS\x17\x02YXVS\x1c\x14\x02\x15\x07\x1a\x0fR\x17R3>\x13R4\x12R>\x18Q\x143>Q5\x11>YVS\x17\x02YXVS\x1c\x14\x02\x15\x07\x1a\x0fR\x17R3>\x13R4\x12R>\x18Q\x143>Q'
> b
b'\x03'
> bb
b'\x03\x17'
> bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
b'\x03\x17\x01\x16\x04\x19\x0cQ\x14Q0=\x10Q7\x11Q=\x1bR\x170=R6\x12=ZUP\x14\x01Z[UP\x1f\x03\x17\x01\x16\x04\x19\x0cQ\x14Q0=\x10Q7\x11Q=\x1bR\x170=R6\x12=ZUP\x14\x01Z[UP\x1f\x03\x17\x01\x16\x04\x19\x0cQ\x14Q0=\x10Q7\x11Q=\x1bR\x170=R'
> c
b'\x02'
> cc
b'\x02\x16'
> d
b'\x05'
> ddd
b'\x05\x11\x07'
> dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd
b'\x05\x11\x07\x10\x02\x1f\nW\x12W6;\x16W1\x17W;\x1dT\x116;T0\x14;\\SV\x12\x07\\]SV\x19\x05\x11\x07\x10\x02\x1f\nW\x12W6;\x16W1\x17W;\x1dT\x116;T0\x14;\\SV\x12\x07\\]SV\x19\x05\x11\x07\x10\x02\x1f\nW\x12W6;\x16W1\x17W;\x1dT\x116;T'
```

It seems that it starts off as a vignere cipher, but at some points there are weird bit shifts. Additionally it seems to repeat.
Based on two similar inputs of the same size producing a pattern of identical size which repeats.

In the case of:
```bash
> aa
b'\x14'
> aaaa
b'\x14\x02\x15\x07
> b
b'\x03'
> bb
b'\x03\x17'
```

I assume the flag must be coming in the format `auctf{ ... }` so this is probably a [XOR Cipher](https://en.wikipedia.org/wiki/XOR_cipher).

Let's attempt to brute force the key, short example here:
```python
#!/usr/bin/env python

c = b'\x03\x17\x01\x16\x04\x19\x0cQ\x14Q0=\x10Q7\x11Q=\x1bR\x170=R6\x12=ZUP\x14\x01Z[UP\x1f\x03\x17\
x01\x16\x04\x19\x0cQ\x14Q0=\x10Q7\x11'
keys = ['a', 'b', 'c']

for j in keys:
    flag = ''
    for i in c:
        flag += chr(int(i) ^ ord(j))
    print(flag)
```

```bash
$ python test.py | grep -e "auctf"
auctf{n3v3R_r3Us3_y0uR_0Tp_872vc8972}auRStf{n3v3R_r3Us
```

## Flag

auctf{n3v3R_r3Us3_y0uR_0Tp_872vc8972}