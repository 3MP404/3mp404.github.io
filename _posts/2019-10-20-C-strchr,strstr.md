---
layout: post
title: "C/C++ strchr,strstr"
categories: [C/C++함수]
tags: [C/C++]
redirect_from:
 - /2019/10/20/
---

strchr 함수다.

```c
#include<stdio.h>
#include<string.h>
int main()
{
	char a[]="helloCworld";
	char *p=strchr(a,'o');
	while(p!=NULL)
	{
		printf("%s\n",p);
		p=strchr(p+1,'o');
	}
}
```
![strchr 결과](https://user-images.githubusercontent.com/51374792/67154384-8237f680-f336-11e9-9a22-9d5e154bac28.png)

strchr은 문자열 내에서 특정 문자를 찾아준다.

찾은 문자의 주소를 반환하므로 그 뒤의 문장이 출력되게 된다.

만약 문자를 찾지 못하면 NULL을 반환하게 된다.

strstr 함수다.

```c
#include<stdio.h>
#include<string.h>
int main()
{
	char a[]="helloCworld";
	char *p=strstr(a,"oCw");
	printf("%s",p);
}
```
![strstr 결과](https://user-images.githubusercontent.com/51374792/67161101-33b84580-f392-11e9-9c0a-54462465ba2a.png)

strchr의 문자열 버전이라고 봐도 무방하다.
