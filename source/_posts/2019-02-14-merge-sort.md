---
title: '归并排序'
date: 2019-02-14
categories: Algorithm
tags: [algorithm,c,sort]
---

归并排序，是指将两个有序序列合并成一个有序序列的操作。两个有序序列如何获得？通过从上到下递归分解序列，当元素为1时就是有序序列。接着从下到上合并序列，最后得到一个有序序列。  

<!-- more -->

  
```c
#include<stdio.h>
#include<string.h>

/**
 * 合并数组
 *
 * @param arrays
 * @param L      指向数组第一个元素
 * @param M      指向数组分隔的元素
 * @param R      指向数组最后的元素
 */
void merge(int arrays[], int L, int M, int R) {

	int lengthLeft = M - L;
	int lengthRight = R - M + 1;

	//左边的数组的大小
	int leftArray[lengthLeft];
	memset(leftArray,0,lengthLeft*sizeof(int));

	//右边的数组大小
	int rightArray[lengthRight];
	memset(rightArray,0,lengthRight*sizeof(int));

	//往这两个数组填充数据
	for (int i = L; i < M; i++) {
		leftArray[i - L] = arrays[i];
	}
	for (int i = M; i <= R; i++) {
		rightArray[i - M] = arrays[i];
	}

	int i = 0, j = 0;
	//arrays数组的第一个元素
	int k = L;

	//比较这两个数组的值，哪个小，就往数组上放
	while (i < lengthLeft && j < lengthRight) {
		
		//谁比较小，谁将元素放入大数组中,移动指针，继续比较下一个
		if (leftArray[i] < rightArray[j]) {
			arrays[k++] = leftArray[i++];
		} else {
			arrays[k++] = rightArray[j++];
		}
	}

	//如果左边的数组还没比较完，右边的数都已经完了，那么将左边的数抄到大数组中(剩下的都是大数字)
	while (i < lengthLeft) {
		arrays[k++] = leftArray[i++];
	}
	//如果右边的数组还没比较完，左边的数都已经完了，那么将右边的数抄到大数组中(剩下的都是大数字)
	while (j < lengthRight) {
		arrays[k++] = rightArray[j++];
	}
}

/**
 * 归并排序
 *
 * @param arrays
 * @param L      指向数组第一个元素
 * @param R      指向数组最后一个元素
 */
void mergeSort(int arrays[], int L, int R) {

	//如果只有一个元素，那就不用排序了
	if (L == R) {
		return;
	} else {

		//取中间的数，进行拆分
		int M = (L + R) / 2;

		//左边的数不断进行拆分
		mergeSort(arrays, L, M);

		//右边的数不断进行拆分
		mergeSort(arrays, M + 1, R);

		//合并
		merge(arrays, L, M + 1, R);

	}
}
	
	
void main()
{
	int a[] = {5,2,3,4,9,8,7,1,6};
	int length = sizeof(a)/4;
	mergeSort(a,0,length-1);
	for(int i=0;i<length;i++)
	{   
		printf("a[%d] = %d\n",i,a[i]);
	}   
}
```

#### 参考资料  
1. [归并排序就这么简单](https://juejin.im/post/5ab4c7566fb9a028cb2d9126)  
