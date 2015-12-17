---
layout: post
title: "归并排序"
date: 2014-12-03 12:26:58 +0800
comments: true
categories: 
---

归并排序比较占用内存，但却是一种效率高且稳定的算法。

实例
---
---

一,归并子集合<br>
假如有两个`已排序的集合`{1,3,5,7}和{2,4,6,8}。<br>首先,我们选取两个集合中的最小值1,将其从集合中剔除,并放入一个新的归并集合。<br>接着，我们再在剔除两个集合中的最小值2,放入归并集合{1,2}。<br>依次类推,我们接下来会依次剔取3、4、5、6、7、8<br>最后得到最终的归并集合 {1、2、3、4、5、6、7、8},`操作次数n-1`。

二,对集合{3、4、7、6、8、2、1、5、9}进行归并排序。<br>

1. 首先对集合{3、4、7、6、8}和{2、1、5、9}进行递归排序。
2. 将两个已排序的子集合归并在一起。

	![merge](/images/2014/12/mergesort.png) 

运行时间
---
---

观察归并前的最后两个子集 {3,4,6,7,8}和{1,2,5,9}的比较操作所需时间复杂度

	T(n) = T(n/2) + T(n/2);<br>
	
再往前推一层比较操作所需的时间

	T(n) =  T(n/4) + T(n/4) + T(n/4) + T(n/4);

所以,每层数所花费的比较时间是一个关于n的值Θ(n);(ps.最后一层不是Θ(n))<br>

树的层级数log(n)值应该是:<br>

	n -> n/2 -> n/4 -> ....... 1<br>
	
假设n是正好是2的整数幂,那么到Θ(lgn)。(`lgn即log2^n`,脑子里的数学知识都被猪吃了吧。)<br>

总时间T(n) = Θ(n*lgn),当归并排序在一个充分大的输入规模n下将优于插入算法。

<!--more-->
代码实现
---
---
	
	void merge(int a[],int f,int mid,int l)
	{
	    int n1 = mid - f +1;//前一部分的长度
	    int n2 = l - mid;   //后一部分的长度
	    
	    //申请两个空间存放排好的数组
	    int *front = (int*)malloc(n1*sizeof(int));
	    int *back = (int*)malloc(n2*sizeof(int));
	    
	    int i;
	    /*将数组转入两个新空间*/
	    for(i = 0; i < n1;i++)
	    {
	        front[i] = a[f+i];
	    }
	    
	    for(i = 0; i < n2;i++)
	    {
	        back[i] = a[mid+1+i];
	    }
	    
	    /*将元素合并*/
	    int j,k;
	    i = 0;
	    j = 0;
	    k = f;
	    
	    while (i < n1 && j<n2)
	    {
	        if(front[i] < back[j])
	        {
	            a[k++] = front[i++];
	        }else{
	            a[k++] = back[j++];
	        }
	    }
	    
	    /*将剩余的元素合并*/
	    while (i<n1) {
	        a[k++] = front[i++];
	    }
	    
	    while (j<n2) {
	        a[k++] = back[j++];
	    }
	}
	
	void merge_sort(int a[],int f,int l)
	{
	    if(f < l)
	    {
	        //中间元素索引
	        int mid = (f+l)/2;
	        
	        //排序左侧数组
	        merge_sort(a, f, mid);
	        //排序右侧数组
	        merge_sort(a, mid+1, l);
	        //合并两侧数组
	        merge(a,f,mid,l);
	    }
	}
	
	int main(int argc, const char * argv[])
	{
	    int a[9] = {3,4,7,6,8,2,1,5,9};
	    
	    merge_sort(a,0,8);
	    
	    for(int i=0;i<9;i++){
	        printf("%d ",a[i]);
	    }
	    
	    return 0;
	}