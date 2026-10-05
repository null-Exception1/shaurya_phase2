# basic-mod2

now the message given is

`432 126 192 354 98 296 272 311 146 297 364 373 262 277 215 377 255 407 72 46 213 448 212`

```
Take each number mod 41 and find the modular inverse for the result. Then map to the following character set: 1-26 are the alphabet, 27-36 are the decimal digits, and 37 is an underscore.

Wrap your decrypted message in the academy flag format (i.e. academy{decrypted_message})

```

firstly you discover that your ioqm olympiad training from 10th didnt go to shit, as it turns out you know what a damn modular inverse is

so basically mod 41, then modular inverse, 1-26 are alphabets, 27-36 decimals, 37-underscore

so first lets do mod 41

```py

>>> enc ="432 126 192 354 98 296 272 311 146 297 364 373 262 277 215 377 255 407 72 46 213 448 212".split(" ")
>>> enc = [int(x) for x in enc]
>>> enc
[432, 126, 192, 354, 98, 296, 272, 311, 146, 297, 364, 373, 262, 277, 215, 377, 255, 407, 72, 46, 213, 448, 212]
>>> enc = [x%41 for x in enc]
>>> enc
[22, 3, 28, 26, 16, 9, 26, 24, 23, 10, 36, 4, 16, 31, 10, 8, 9, 38, 31, 5, 8, 38, 7]

```

alright buddy boy, lets modular inverse that shit straight up

```py

def mod_inverse(a, m):
    for x in range(1, m):
        if (a * x) % m == 1:
            return x
    return None

enc = [22, 3, 28, 26, 16, 9, 26, 24, 23, 10, 36, 4, 16, 31, 10, 8, 9, 38, 31, 5, 8, 38, 7]

s = "abcdefghijklmnopqrstuvwxyz0123456789_"
for i in enc:

    print(s[mod_inverse(i,41)-1],end="")

# 1nv3r53ly_h4rd_950d690f
```

wrap in academy{} format and then ez win



# something i did get stuck on

- i almost forgot i had to take mod inverse and thought instead that i had to mod 41, then mod 37 after that to make it compatible to the 37 letter set



