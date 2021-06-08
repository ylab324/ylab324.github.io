---
title: '快速排序'
date: 2020-09-14
categories: Algorithm
tags: [algorithm,c,sort]
---

快速排序，基本思想是：通过一趟排序将要排序的数据分割成独立两部分，其中一部分的所有数据都比另外一部分的所有数据小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序数列。  

<!-- more -->


:joy:  
一趟快速排序的算法是：  
1. 设置两个变量i、j，排序开始的时候：i=0，j=N-1；  
2. 以第一个数组元素作为关键数据，赋值给key，即key=A[0]；  
3. 从j开始向前搜索，即由后开始向前搜索(j--)，找到第一个小于key的A[j]，将A[j]的值赋给A[i]；  
4. 从i开始向后搜索，即由前开始向后搜索(i++)，找到第一个大于key的A[i]，将A[i]的值赋给A[j]；  
5. 重复第3、4步，直到i=j。  

```c
#include <stdio.h>

void swap(int src[], int left, int right)
{
	int tmp = src[left];
	src[left] = src[right];
	src[right] = tmp;
}

int partArray(int src[], int low, int high)
{
	int pivot = src[low];
	while(low < high)
	{
		while(low < high && src[high] >= pivot)
		{
			high--;
		}
		swap(src,low,high);

		while(low < high && src[low] <= pivot)
		{
			low++;
		}
		swap(src,low,high);
	}
	return low;
}

void do_quick_sort(int src[], int low, int high)
{
	int pivot = 0;
	if(low <= high)
	{
		pivot = partArray(src,low,high);
		do_quick_sort(src,low,pivot-1);
		do_quick_sort(src,pivot+1,high);
	}
}

void quick_sort(int src[], int length)
{
	do_quick_sort(src, 0, length-1);
}

void main()
{
        int a[] = {5,2,3,4,0,9,8,7,1,6};
        int length = sizeof(a)/4;
        quick_sort(a,length);
        for(int i=0;i<length;i++)
        {
        	printf("a[%d] = %d\n",i,a[i]);
        }
}
```
