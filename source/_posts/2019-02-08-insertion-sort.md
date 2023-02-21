---
title: 插入排序
date: 2019-02-08
categories: Algorithm
tags: [algorithm,c,sort]
---

如何去理解插入排序？我的方法是：已知选择排序的思路是选出特定元素去组成新的序列，对于插入排序而言，是要把元素插入到有序序列。  

<!-- more -->


代码如下：  
```c
#include <stdio.h>

void sort(int a[], int length)
{
        //直接插入排序，产生从小到大的序列
        int i = 0;
        int j = 0;
        int t = 0;
        for(i=1;i<length;i++)
        {   
                if(a[i]<a[i-1]) //这个条件是要判断当前元素是否要插入到有序序列
                {   
                        t = a[i]; //把要插入的值先保存出来
                        for(j=i-1;j>=0&&a[j]>t;j--)
                        {   
                                a[j+1] = a[j]; //把元素往后挪，直到找到要插入的位置
                        }   
                        a[j+1] = t; //终于找到要插入的位置了，赋值
                }   
        }   
}

void main()
{
        int a[] = {5,2,3,4,9,8,7,1,6};
        int i = 0;
        int length = sizeof(a)/4;
        sort(a,length);
        for(i=0;i<length;i++)
        {   
                printf("a[%d] = %d\n",i,a[i]);
        }   
}
```
以上为直接插入排序，如果数据量比较大，尝试用二分插入排序，可以减小比较次数。因为相对于直接插入排序，一边比较一边找插入位置而言，二分插入排序的做法是先通过比较找到要插入的位置，然后挪元素，这里二分法找位置会较快。  
二分插入排序，代码如下：  
```c
#include <stdio.h>
void sort(int a[], int length)
{   
        //二分插入排序                    
        int i = 0;
        int j = 0;
        int t = 0;
        for(i=1;i<length;i++)
        {
                if(a[i]<a[i-1])
                {
                        t = a[i];
                        int left = 0;
                        int right = i-1;
                        while(left<=right)
                        {
                                int mid = (left+right)/2;
                                if(t<a[mid])
                                {
                                        right = mid-1;
                                }
                                else
                                {
                                        left = mid+1;
                                }
                        }
                        for(j=i-1;j>=left;j--)
                        {
                                a[j+1] = a[j];
                        }
                        a[left] = t;
                }
        }
}   
void main()
{   
        int a[] = {5,2,3,4,9,8,7,1,6};
        int i = 0;
        int length = sizeof(a)/4;
        sort(a,length);
        for(i=0;i<length;i++)
        {
                printf("a[%d] = %d\n",i,a[i]);
        }
}
```
