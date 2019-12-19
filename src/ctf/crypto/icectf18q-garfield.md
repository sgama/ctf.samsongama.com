# IceCTF18Q Garfield

![Garfeld.png](assets/garfeld.png "Garfeld.png")

```
IjgJUO{P_LOUV_AIRUS_GYQUTOLTD_SKRFB_TWNKCFT}
07271978
```
are found in the picture

Note:

`IjgJUO{P_LOUV_AIRUS_GYQUTOLTD_SKRFB_TWNKCFT}` -> seems to be the flag
`I + 0 = I`, `j - 7 = c`, `g - 2 = e` .... seee a pattern?

`Garfield` is spelt as `Garfeld`

```python
#!/usr/bin/env python
numbers = '07271978'
flag =[]
with open("message.txt") as handle:
    message = handle.read()
    counter = 0
    for c in message:
        c = c.upper()

        if (c in uppercase):
            index = uppercase.index(c)
            offset = int(numbers[counter % len(numbers)])
            new_char = uppercase[index - offset]
            flag.append(new_c)
            counter += 1
        else:
            flag.append(character)
print ''.join(flag)
```

ICECTF{I_DONT_THINK_GRONSFELD_LIKES_MONDAYS}