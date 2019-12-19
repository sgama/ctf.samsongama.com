# PicoCTF19 Shark Wire

## Challenge

We found this packet [capture](https://2019shell1.picoctf.com/static/ae9ca8cff43ed638ed5d137f9ece7455/capture.pcap). Recover the flag. You can also find the file in /problems/shark-on-wire-1_0_13d709ec13952807e477ba1b5404e620.

## Hints

Try using a tool like Wireshark.

What are streams?

## Solution

`Analyze > Follow UDP Stream`

Amazingly, it was Stream #6


## Flag

`picoCTF{StaT31355_636f6e6e}`

## Helpful tools

[https://networksecuritytools.com/list-wireshark-display-filters/](https://networksecuritytools.com/list-wireshark-display-filters/)

## Other solution

```python
#!/usr/bin/env python
from scapy.all import *
"""
We found this packet capture. Recover the flag. 
You can also find the file in /problems/shark-on-wire-1_0_13d709ec13952807e477ba1b5404e620.
"""
a = rdpcap('capture.pcap')
flag = []
for i in a[UDP]:
    try:
        if i[IP].src == '10.0.0.2' and i[IP].dst == '10.0.0.12':
            flag.append((i[Raw].load).decode())
    except IndexError:
        continue
print("".join(flag))
```