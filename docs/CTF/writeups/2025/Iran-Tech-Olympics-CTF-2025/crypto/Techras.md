---
showFullContent: false
readingTime: true
---

## Twin Oracle

|Category|Difficulty|Score|Solves|First 🩸|
|:-|:-|:-|:-|:-|
|Crypto|Baby 👶|29|301|G3|

## Code / Description

```
I'm having trouble determining when to discard a number versus when to reuse it in Techras. Any advice would be helpful!
```

```py
#!/usr/bin/env python3

from Crypto.Util.number import *
from string import *
from flag import flag

def pad(flag):
	r = len(flag) % 8
	if r != 0:
		flag = flag[:-1] + (8 - r) * printable[:63][getRandomRange(0, 62)].encode() + flag[-1:]
	return flag

def genkey(nbit):
	p, q = [getPrime(nbit) for _ in ':)']
	n = p * q
	return n, (p, q)

def encrypt(msg, pubkey):
	msg = pad(msg)
	e = getPrime(32)
	m = bytes_to_long(msg)
	c = pow(m, e, pubkey)
	return str(c) + str(e)

nbit = 1024
pubkey, _ = genkey(nbit)

print(f'n = {pubkey}')
for _ in range(110):
	print(f'c = {encrypt(flag, pubkey)}')
```


## Overview



## Challenge Analysis









## Solution




## Final Code




## Flag

```
ASIS{d0nT___rEuS3___peXp!}
```

## Authors

> [Kourosh Rajabzadeh](https://github.com/KooroshRZ)