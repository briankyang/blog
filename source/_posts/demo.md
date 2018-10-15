---
title: demo
date: 2017-05-21 12:43:18
tags: 测试
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=29454566&auto=0&height=66"></iframe>

---
### 归并排序
<!-- more -->

```java
package sort_related.merge_sort;

import sort_related.common.SortProcessor;

import java.util.Arrays;

/**
 * Created by yangkang0121 on 2/23/17.
 */
public class MergeSortProcessor implements SortProcessor{
    public <T extends Comparable<? super T>> void sort(T[] arr, int start, int end) {
        if (start < 0 || start > end - 1 || end > arr.length || end - start < 2)
            return;

        for (int gap = 1; gap < end - start;gap *= 2)
            mergeSort(arr,start,gap,end - start);
    }

    private static <T extends Comparable<? super T>> void mergeSort(T[] arr,int start,int gap,int length){
        int i;
        for (i = start; i + gap * 2 - 1 < start + length; i += gap * 2){
            merge(arr,i,i + gap - 1,i + gap * 2);
        }
        if (i + gap < start + length)
            merge(arr,i,i + gap - 1,start + length);
    }

    @SuppressWarnings("Since15")
    private static <T extends Comparable<? super T>> void merge(T[] arr, int start, int mid, int end){
        int i = start,j = mid + 1,k = 0;

        T[] arr2 = Arrays.copyOfRange(arr,start,end);

        while (i <= mid && j < end){
            if (arr[i].compareTo(arr[j]) < 0){
                arr2[k++] = arr[i++];
            }else
                arr2[k++] = arr[j++];
        }

        while (i <= mid)
            arr2[k++] = arr[i++];
        while (j < end)
            arr2[k++] = arr[j++];

        System.arraycopy(arr2,0,arr,start,end - start);
    }
}


```
