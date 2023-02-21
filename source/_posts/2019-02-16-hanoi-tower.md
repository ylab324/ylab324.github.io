---
title: '汉诺塔'
date: 2019-02-16
categories: Algorithm
tags: [algorithm,c,hanoi]
---

递归实现汉诺塔问题。  

<!-- more -->


```c
#include <stdio.h>

void moveOne(int diskNum, char init, char dest)
{
	printf("move disk %d from %c to %c\n",diskNum,init,dest);
}

void move(int diskNum, char init, char temp, char dest)
{
	if(diskNum == 1)
	{
		moveOne(diskNum, init, dest);
	}
	else
	{
		move(diskNum-1, init, dest, temp);
		moveOne(diskNum, init, dest);
		move(diskNum-1, temp, init, dest);
	}
}

void main()
{
	move(3,'A','B','C');
}
```

输出如下：  
> move disk 1 from A to C  
> move disk 2 from A to B  
> move disk 1 from C to B  
> move disk 3 from A to C  
> move disk 1 from B to A  
> move disk 2 from B to C  
> move disk 1 from A to C  
