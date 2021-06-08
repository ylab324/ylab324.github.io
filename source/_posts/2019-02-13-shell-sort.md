---
title: '希尔排序'
date: 2019-02-13
categories: Algorithm
tags: [algorithm,c,sort]
---

希尔排序，实质是分组插入排序。  

<!-- more -->


代码有点难懂，需要手动演算才更好理解。  
```c
#include <stdio.h>

void sort(int a[], int length)
{
        for (int gap = length / 2; gap > 0; gap /= 2)
                for (int i = gap; i < length; i++)
                        for (int j = i - gap; j >= 0 && a[j] > a[j + gap]; j -= gap)
                        {
                                int temp = a[j];
                                a[j] = a[j+gap];
                                a[j+gap] = temp;
                        }
}

void main()
{
        int a[] = {5,2,3,4,0,9,8,7,1,6};
        int length = sizeof(a)/4;
        sort(a,length);
        for(int i=0;i<length;i++)
        {
                printf("a[%d] = %d\n",i,a[i]);
        }
}
```

#### 参考资料  
1. [图解希尔排序](https://www.itcodemonkey.com/article/3286.html)  
2. [白话经典算法系列之三 希尔排序的实现](https://blog.csdn.net/MoreWindows/article/details/6668714)
