# PicoCTF19 The Numbers

## Challenge

The [numbers](https://2019shell1.picoctf.com/static/880cb722e666005093d6c2c5751ec042/the_numbers.png)... what do they mean?

![the_numbers.png](assets/the_numbers.png "the_numbers.png")

## Hint

The flag is in the format PICOCTF{}

## Solution

This looks like the Letter Number Cipher (known as A1Z26):

Use [this](https://www.dcode.fr/letter-number-cipher) to decode the message.

```
16 9 3 15 3 20 6 { 20 8 5 14 21 13 2 5 18 19 13 1 19 15 14 }
```

### Flag
`PICOCTF{THENUMBERSMASON}`