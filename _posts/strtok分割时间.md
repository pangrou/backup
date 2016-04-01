---
title: strtok与sscanf
date: 2016-01-27 10:44:54
tags:
---

<!--more-->

两者都可以分解字符串，下面用两个函数分别实现把"08:50:11"解析成对应的十分秒。

### 1.strtok函数

#### 原型
```
char *strtok(char s[], const char *delim);
```
#### 功能
分解字符串为一组字符串。s为要分解的字符串，delim为分隔符字符串。
例如：strtok("abc,def,ghi",",")，最后可以分割成为abc def ghi.

### 代码部分
```cpp
#ifdef __APPLE__
#define error printf
#endif

#include<stdio.h>
#include<stdlib.h>
#include<string.h>

typedef struct DATE_TIME_S{
    unsigned char hour;
    unsigned char min;
    unsigned char sec;
}DATE_TIME_S;

void conv_time(char *time_str,DATE_TIME_S *dt);

void main(void)
{
	char *test = "08:50:11";
	DATE_TIME_S *dt;
	conv_time(test, dt);
	printf("dt->hour[%d]\n", dt->hour);
	printf("dt->min[%d]\n", dt->min);
	printf("dt->sec[%d]\n", dt->sec);
}

void conv_time(char *time_str,DATE_TIME_S *dt)
{
	if((time_str == NULL) || (dt == NULL)) {
		return;
	}
	int len = strlen(time_str);
	if((len > 8) || (len < 3)) {
		return;
	}
	int offset = 0;
	char temp[9];
	memset(temp, 0, sizeof(temp));
	strcpy(temp, time_str);

	char *p = NULL;
	char *delim = ":";
	p = strtok(temp, delim);
	if(p != NULL) {
		dt->hour = atoi(p);
	}
	offset += strlen(p) + 1;
	strcpy(temp, time_str + offset);
	p = strtok(temp, delim);
	if(p != NULL) {
		dt->min = atoi(p);
	}
	offset += strlen(p) + 1;
	if(len > offset) {
		strcpy(temp, time_str + offset);
		p = strtok(temp, delim);
		if(p) {
			dt->sec = atoi(p);
		}
	}else {
		dt->sec = 0;
	}
}

```
### 运行结果
```cpp
➜  Desktop git:(master) ✗ ./t
dt->hour[8]
dt->min[50]
dt->sec[11]
```

### 2.sscanf函数

#### 原型
```
int sscanf( const char *, const char *, ...);
```
#### 功能
成功则返回参数数目，失败则返回-1，错误原因存于errno中。
sscanf("1 2 3","%d %d %d",buf1, buf2, buf3); 
成功调用返回值为3，即buf1，buf2，buf3均成功转换。

### 代码部分
```cpp

void conv_time(char *time_str,DATE_TIME_S *dt)
{
	if((time_str == NULL) || (dt == NULL)) {
		return;
	}
	int num;
	num = sscanf(time_str,"%d:%d:%d",&dt->hour,&dt->min,&dt->sec);
	if(num < 2) {
		dt->hour = 0;
	}else if(num < 3){
		dt->sec = 0;
	}
}

```
