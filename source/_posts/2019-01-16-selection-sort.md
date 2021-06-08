---
title: 选择排序
date: 2019-01-16
categories: Algorithm
tags: [algorithm,c,sort]
---

选择排序，字面意思就是依次选出最大/小值、次最大/小值、次次最大/小值、次次次最大/小值......  

<!-- more -->

直接贴代码：  
```c
#include<stdio.h>
void selection_sort(int * a,int length)
{
    int i = 0;
    int j = 0;
    int min = 0;
    int temp = 0;
    for(i=0;i<length;i++)
    {
	min = i;
	for(j=i+1;j<length;j++)
	{
	    if(a[j]<a[min])
	    {
		min = j;
	    }
	}
	if(min!=i)
	{
	    temp = a[i];
	    a[i] = a[min];
	    a[min] = temp;
	}
    }
}
void main(void)
{
    int i = 0;
    int a[] = {4,5,2,6,1,3,9,7,8};
    int length = 0;
    length = sizeof(a)/4;
    selection_sort(a,length);
    for(i=0;i<length;i++)
    {
	printf("a[%d] = %d\n",i,a[i]);
    }
}
```

优化思路是可以在一个循环里面不仅仅是选取最大/小值，而是同时把最大值和最小值取出来。优化代码如下：  
```c
#include<stdio.h>

void selection_sort(int * a, int length)
{
    int i = 0;
    int min = 0;
    int max = 0;
    int left = 0;
    int right = 0;
    int temp = 0;
    for(left=0,right=length-1;left<right;left++,right--)
    {
	min = left;
	max = right;
	for(i=left;i<=right;i++)
	{
	    if(a[min]>a[i])
		min = i;
	    if(a[max]<a[i])
		max = i;
	}
	if(min!=left)
	{
	    temp = a[left];
	    a[left] = a[min];
	    a[min] = temp;
	    if(max==left)  //why
		max = min; //why
	}
	if(max!=right)
	{
	    temp = a[right];
	    a[right] = a[max];
	    a[max] = temp;
	}
    }
}

void main(void)
{
    int i = 0;
    int a[] = {4,5,2,6,1,3,9,7,8};
    int length = 0;
    length = sizeof(a)/4;
    selection_sort(a,length);
    for(i=0;i<length;i++)
    {
	printf("a[%d] = %d\n",i,a[i]);
    }
}
```

一开始想不通交换min值的时候为什么要重新获取max值，画个图就一目了然了。 
> if(max==left)  
>   max = min;  

![selectionSort](https://i.loli.net/2019/01/17/5c3fe03bb93a5.jpg)
因为i是由left坐标遍历到right坐标，所以最大值有可能落在left的坐标上，当最大值落在left坐标时，交换最小值的时候， a[left]与a[min]交换了，原本最大值是a[left]在left位置，交换最小值后转移到min位置，即此刻最大值的坐标落在min的位置，而不是原来left的位置。
