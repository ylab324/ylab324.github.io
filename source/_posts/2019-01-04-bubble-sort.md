---
title: 冒泡排序
date: 2019-01-04
categories: Algorithm
tags: [c,java,algorithm,sort]
---

冒泡排序，思路是相邻两个的两两比较。  

<!-- more -->

### 代码  
C  
```c
#include <stdio.h>

void sort(int* a, int length)
{
    int i = 0;
    int j = 0;
    for(i = 0; i < length; i++)
    {
        for(j = length - 1; j > i; j--)
        {
            if(a[j] < a[j-1])
            {
               int temp = a[j];
               a[j] = a[j-1];
               a[j-1] = temp;
            }
        }
    }
}

int main(void)
{
    int a[] = {4,5,2,7,1,6,3};
    int length = sizeof(a)/4;
    int i = 0;
    sort(a,length);
    for(i = 0; i < length; i++)
    {
        printf("a[%d] = %d\n",i,a[i]);
    }
}
```

java
```java
public class BubbleSort {
  
	public void sort(int[] a, int length) {
		for(int i = 0; i < length; i++) {
			for(int j = length -1; j > i; j--) {
				if(a[j] < a[j-1]) {
					int temp = a[j];
					a[j] = a[j-1];
					a[j-1] = temp;
				}
			}
		}
	}

	public static void main(String[] args) {
		int a[] = {4,5,2,7,1,6,3};
		BubbleSort bs = new BubbleSort();
		bs.sort(a,a.length);
		for(int i:a) {
			System.out.println(i);
		}
	}
}
```

### 优化
添加打印信息，修改代码如下：  
```c
#include <stdio.h>

void sort(int* a, int length)
{
    int i = 0;
    int j = 0;
	int s = 0;
    for(i = 0; i < length; i++)
    {
        for(j = length - 1; j > i; j--)
        {
            if(a[j] < a[j-1])
            {
               int temp = a[j];
               a[j] = a[j-1];
               a[j-1] = temp;
            }
        }
		for(s = 0; s < length; s++)
		{
			printf("%d ",a[s]);
		}
		printf("\n");
    }
}

int main(void)
{
    int a[] = {4,5,2,7,1,6,3};
    int length = sizeof(a)/4;
    int i = 0;
    sort(a,length);
    for(i = 0; i < length; i++)
    {
        printf("a[%d] = %d\n",i,a[i]);
    }
}
```
编译运行，得到有如下打印信息：  
> 1 4 5 2 7 3 6  
> 1 2 4 5 3 7 6  
> 1 2 3 4 5 6 7  
> 1 2 3 4 5 6 7  
> 1 2 3 4 5 6 7  
> 1 2 3 4 5 6 7  
> 1 2 3 4 5 6 7  

发现从第3行开始便是多余的动作，如何优化呢？当再没有两两交换发生时，退出便是，这里优化的外层的循环。修改代码如下：  
```c
#include <stdio.h>

void sort(int* a, int length)
{
	int i = 0;
	int j = 0;
	int s = 0;
	int flag = 0;
	for(i = 0; i < length; i++)
	{
		flag = 0; //优化
		for(j = length - 1; j > i; j--)
		{
			if(a[j] < a[j-1])
			{
				int temp = a[j];
				a[j] = a[j-1];
				a[j-1] = temp;
				flag = 1; //优化
			}
		}
		if(flag == 0) //优化
			return; //优化
		for(s = 0; s < length; s++)
		{
			printf("%d ",a[s]);
		}
		printf("\n");
	}
}

int main(void)
{
	int a[] = {4,5,2,7,1,6,3};
	int length = sizeof(a)/4;
	int i = 0;
	sort(a,length);
	for(i = 0; i < length; i++)
	{
		printf("a[%d] = %d\n",i,a[i]);
	}
}
```

是否有机会优化内层循环？对于一组数据，可能有一部分数据是有序的，此时记录最后一次发生交换的位置，下次比较就从开始比较到刚才记录的位置，可减少多余的比较，因为此位置后面的数据是有序的，后面就不需要比较这些内容。
修改代码如下：
```c
#include <stdio.h>

void sort(int* a, int length)
{
	int i = 0;
	int j = 0;
	int k = 0;
	int s = 0;
	int pos = 0;
	for(i = 0; i < length; i++)
	{
		for(j = length - 1; j > k; j--)
		{
			if(a[j] < a[j-1])
			{
				int temp = a[j];
				a[j] = a[j-1];
				a[j-1] = temp;
				pos = j;
			}
		}
		k = pos;
	}
}

int main(void)
{
	int a[] = {4,5,2,7,1,6,3};
	int length = sizeof(a)/4;
	int i = 0;
	sort(a,length);
	for(i = 0; i < length; i++)
	{
		printf("a[%d] = %d\n",i,a[i]);
	}
}
```

完整c代码：  
```c
#include <stdio.h>

void sort(int* a, int length)
{
	int i = 0;
	int j = 0;
	int k = 0;
	int flag = 0;
	int pos = 0;
	for(i = 0; i < length; i++)
	{
		flag = 0;
		for(j = length - 1; j > k; j--)
		{
			if(a[j] < a[j-1])
			{
				int temp = a[j];
				a[j] = a[j-1];
				a[j-1] = temp;
				flag = 1;
				pos = j;
			}
		}
		if(flag == 0)
			return;
		k = pos;
	}
}

int main(void)
{
	int a[] = {4,5,2,7,1,6,3};
	int length = sizeof(a)/4;
	int i = 0;
	sort(a,length);
	for(i = 0; i < length; i++)
	{
		printf("a[%d] = %d\n",i,a[i]);
	}
}
```

### 参考链接
[【排序】：冒泡排序以及三种优化](https://blog.csdn.net/hansionz/article/details/80822494)
