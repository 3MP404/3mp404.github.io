---
layout: post
title: "OOB(Out OF Bound)"
categories: [해킹기법]
tags: [pwnable]
redirect_from:
 - /2019/05/27
---
만약에 이런 C 프로그램이 있다고 치자.


```c
#include <stdio.h>
int main() 
{ 
int b[9]; 
int a=10; 
scanf("%d",&b[-1]); 
printf("%p\n%p\n%d\n",&a,&b[-1],a); 
}
```


참고로 필자가 소스를 작성한 환경은 칼리 리눅스이다.

위 프로그램을 작동시키면, 이런식으로 나올것이다.


```bash
15 
0x7fff0b1610fc 
0x7fff0b1610fc 
15 
root@kali:~#
```


소스를 하나하나 살펴보자.

b를 9칸짜리 배열로 선언하고, a를 10으로 선언한다.

그리고 b의 [-1]인덱스에 입력받는다.

그 뒤에 b[-1]의 주소값과 a의 주소값을 출력한 후, a의 값을 출력한다.

배열의 최대 크기가 넘어가는 인덱스의 값을 건드리면 그만큼 떨어진 주소값의 값이 바뀐다.


```bash
(gdb) set disassembly-flavor intel 
(gdb) disas main 
Dump of assembler code for function main: 
   0x0000000000001145 <+0>: push   rbp 
   0x0000000000001146 <+1>: mov    rbp,rsp 
   0x0000000000001149 <+4>: sub    rsp,0x40 
   0x000000000000114d <+8>: mov    DWORD PTR [rbp-0x34],0xa 
   0x0000000000001154 <+15>: lea    rax,[rbp-0x30] 
   0x0000000000001158 <+19>: sub    rax,0x4 
   0x000000000000115c <+23>: mov    rsi,rax 
   0x000000000000115f <+26>: lea    rdi,[rip+0xe9e]        # 0x2004 
   0x0000000000001166 <+33>: mov    eax,0x0 
   0x000000000000116b <+38>: call   0x1040 <__isoc99_scanf@plt> 
   0x0000000000001170 <+43>: mov    edx,DWORD PTR [rbp-0x34] 
   0x0000000000001173 <+46>: lea    rax,[rbp-0x30] 
   0x0000000000001177 <+50>: lea    rsi,[rax-0x4] 
   0x000000000000117b <+54>: lea    rax,[rbp-0x34] 
   0x000000000000117f <+58>: mov    ecx,edx 
   0x0000000000001181 <+60>: mov    rdx,rsi 
   0x0000000000001184 <+63>: mov    rsi,rax 
   0x0000000000001187 <+66>: lea    rdi,[rip+0xe79]        # 0x2007 
   0x000000000000118e <+73>: mov    eax,0x0 
   0x0000000000001193 <+78>: call   0x1030 <printf@plt> 
   0x0000000000001198 <+83>: mov    eax,0x0 
   0x000000000000119d <+88>: leave   
   0x000000000000119e <+89>: ret     
End of assembler dump. 
(gdb) q
```


이 프로그램을 스택으로 그려보자.



|         |
|:--------|
| b[9] |rbp-0x30
|a| rbp-0x34





총 차이가 4가 나는 것을 알 수 있다.

그러나 b[0]의 주소가 rbp-0x30이다.

여기서 4가 더 작아져야 할 필요가 있다.

여기에 OOB가 사용되는 것이다.

b[-1]과 같은 식으로 배열 인덱스에 음수를 넣어주면, 주소값이 계속하여 줄어들게 된다.

자료형이 int이므로 하나에 4차이가 난다.

그러므로 b[-1]의 주소는 rbp-0x30-4 = rbp-0x34

a의 주소값을 가지게 되는 것이다.

그러므로 b[-1]을 입력받으면 a의 값을 조작할 수 있게 된다.

그래서 10으로 선언됬던 a의 값이 b[-1]에 15를 입력하면 15가 되는 것이다.