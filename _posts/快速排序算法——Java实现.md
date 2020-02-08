---
layout:     post
title:      快速排序算法——Java实现
subtitle:   快速排序算法——Java实现
date:       2020-01-09
author:     BY
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 算法
    - Java
---

@[TOC](快速排序算法——Java实现)
某公司第三轮面试，被要求手撕快速排序算法，当时因为没有准备，对算法这块忘记的比较多，所以当场蒙圈，回去后仔细研究了各种排序算法，以下是快排的代码。在这里插入代码片

```java
package com.jin.controller;

import java.util.Scanner;

public class QuickSort {
    public static void main(String[] args) {
        int[] arr={2,5,4,8,3,5,9};
        QuickSort(arr, 0, arr.length-1);
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

    }
    public static void QuickSort(int[] arr,int low,int high){
        int i,j,temp,t;
        if(low>high){
            return;
        }
        i=low;
        j=high;
        //temp就是基准位
        temp = arr[low];

        while (i<j) {
            //先看右边，依次往左递减
            while (temp<=arr[j]&&i<j) {
                j--;
            }
            //再看左边，依次往右递增
            while (temp>=arr[i]&&i<j) {
                i++;
            }
            //如果满足条件则交换
            if (i<j) {
                t = arr[j];
                arr[j] = arr[i];
                arr[i] = t;
            }

        }
        //最后将基准为与i和j相等位置的数字交换
        arr[low] = arr[i];
        arr[i] = temp;
        //递归调用左半数组
        QuickSort(arr, low, j-1);
        //递归调用右半数组
        QuickSort(arr, j+1, high);
    }
}

```
**下面简单介绍写快速排序的原理，以下简称快排**


 

> 快排的时间复杂度并不固定，如果在最坏情况下（元素刚好是反向的）速度比较慢，达到   
> O(n^2)（和选择排序一个效率），但是如果在比较理想的情况下时间复杂度 O(nlogn)。
> 
> 快排也是一个分治的算法，快排算法每次选择一个元素并且将整个数组以那个元素分为两部分，整个快速排序的核心是分区（partition），分区的目的是传入一个数组和选定的一个元素，把所有小于那个元素的其他元素放在左边，大于的放在右边。然后递归的将左边或者右边的数据进行排序

