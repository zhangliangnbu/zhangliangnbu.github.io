---
title: 算法-排序
date: 2020-01-22 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

排序算法是《数据结构与算法》中最基本的算法之一。

排序算法可以分为内部排序和外部排序，**内部排序**是数据记录在内存中进行排序，而**外部排序**是因排序的数据很大，一次不能容纳全部的排序记录，在排序过程中需要访问外存。常见的内部排序算法有，分类如下：

- 选择排序、冒泡排序
- 归并排序、快速排序
- 插入排序、希尔排序
- 计数排序、桶排序、基数排序
- 堆排序

![algorithm-10-sort](/images/algorithm-10-sort-2326622.png)



<!--more-->

# 冒泡&选择

## **冒泡**

大数沉底

- 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
- 针对所有的元素重复以上的步骤，除了最后一个。
- 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

```java
public void sort(int[] nums) {
    for (int i = nums.length - 1; i >= 0; i --) {
        // i 表示无序区的最后一个数的索引
        // 对无序区遍历，将大数沉底
        for (int j = 0; j < i; j ++) {
            if (nums[j] > nums[j + 1]) {
                swap(nums, j, j + 1);
            }
        }
    }
}
```

## 选择

**找出最小值，放在无序区最前面**

- 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
- 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
- 重复第二步，直到所有元素均排序完毕。

```java
public void sort(int[] nums) {
    for (int i = 0; i < nums.length; i ++) {
        // i 表示无序区第一个数
        // 遍历无序区，找到最小值
        int minIndex = i;
        for (int j = i; j < nums.length; j ++) {
            if (nums[minIndex] > nums[j]) {
                minIndex = j;
            }
        }
        // 将最小数与无序区第一个数交换
        int temp = nums[minIndex];
        nums[minIndex] = nums[i];
        nums[i] = temp;
    }
}
```



# 归并&快速

 分治递归

## 归并排序

归并排序（Merge sort）是建立在归并操作上的一种有效的排序算法。该算法是采用**分治法（Divide and Conquer）**的一个非常典型的应用。

作为一种典型的分而治之思想的算法应用，归并排序的实现由两种方法：

- 自上而下的递归（所有递归的方法都可以用迭代重写，所以就有了第 2 种方法）；
- 自下而上的迭代；

**算法步骤**

- 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列；
- 设定两个指针，最初位置分别为两个已经排序序列的起始位置；
- 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置；
- 重复步骤 3 直到某一指针达到序列尾；
- 将另一序列剩下的所有元素直接复制到合并序列尾。

```java
public int[] sort(int[] nums) {
    if (nums == null || nums.length < 2) {
        return nums;
    }
    int midIndex = nums.length / 2;
    int[] left = Arrays.copyOfRange(nums, 0, midIndex);
    int[] right = Arrays.copyOfRange(nums, midIndex, nums.length);
    return merge(sort(left), sort(right));
}

public int[] merge(int[] left, int[] right) {
    int[] arr = new int[left.length + right.length];
    int l = 0, r = 0;
    while (l < left.length && r < right.length) {
        if (left[l] < right[r]) {
            arr[l+r] = left[l];
            l ++;
        } else {
            arr[l+r] = right[r];
            r ++;
        }
    }
    if (l >= left.length) {
        System.arraycopy(right, r, arr, left.length + r, right.length - r);
    } else {
        System.arraycopy(left, l, arr, l + right.length, left.length - l);
    }
    return arr;
}
```

## 快速排序

快速排序使用**分治法（Divide and conquer）策略**来把一个串行（list）分为两个子串行（sub-lists）。

快速排序的最坏运行情况是 O(n²)，比如说顺序数列的快排。但它的平摊期望时间是 O(nlogn)，且 O(nlogn) 记号中隐含的常数因子很小，比复杂度稳定等于 O(nlogn) 的归并排序要小很多。所以，**对绝大多数顺序性较弱的随机数列而言，快速排序总是优于归并排序**。

**算法步骤：**

- 从数列中挑出一个元素，称为 "基准"（pivot）;
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

```java
private void quickSort(int[] nums, int left, int right) {
    if (left >= right) {
        return;
    }
    int pivot = portion(nums, left, right);
    quickSort(nums, left, pivot);
    quickSort(nums, pivot + 1, right);
}

// 分配并返回基准点 pivot
private int portion(int[] nums, int left, int right) {
    int pivot = left;
    // 待交换的元素
    int index = left + 1;
    for (int i = index; i <= right; i++) {
        if (nums[i] < nums[pivot]) {
            swap(nums, i, index);
            index ++;
        }
    }
    // 将基准值与最后一个小于基准值的元素进行交换
    swap(nums, pivot, index - 1);
    return index - 1;
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

# 插入&希尔

## 插入排序

复杂度： 平均时间复杂度：O(n^2) ； 平均空间复杂度O(1)

适用于排序性较好的序列！

原理：通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

步骤：

- 将待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列。
- 从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面。

```java
// 通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。
public void sort(int[] nums) {
    int n = nums.length;
    for (int i = 1; i < n; i ++) {
        // 将无序区元素nums[i]插入到0~i-1的有序区域里
        int temp = nums[i], j = i;
        while (j > 0 && nums[j-1] > temp) {
            nums[j] = nums[j-1];
            j--;
        }
        if (j != i) {
            nums[j] = temp;
        }
    }
}
```

## 希尔排序

复杂度： 平均时间复杂度：O(nlogn) ； 平均空间复杂度O(1)

希尔排序，也称递减增量排序算法，是**插入排序的一种更高效的改进版本**。但希尔排序是非稳定排序算法。

希尔排序是基于插入排序的以下两点性质而提出改进方法的：

- 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
- 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位；

希尔排序的基本思想是：**先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录"基本有序"时，再对全体记录进行依次直接插入排序。**

算法步骤：

- 选择一个增量序列 t1，t2，…ti…,...tj...，tk，其中 ti > tj, tk = 1；
- 按增量序列个数 k，对序列进行 k 趟排序；
- 每趟排序，根据对应的增量 ti，将待排序列分割成若干长度为 m 的子序列，分别对各子表进行直接插入排序。仅增量因子为 1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

```java
public void sort(int[] nums) {
    int n = nums.length;
    int temp, j;
    for (int step = n / 2; step >= 1; step /= 2) {
        for (int i = step; i < n; i ++) {
            temp = nums[i];
            j = i;
            while (j-step >= 0 && nums[j-step] > temp) {
                nums[j] = nums[j-step];
                j -= step;
            }
            if (j != i) {
                nums[j] = temp;
            }
        }
    }
}
```

# 计数&桶&基数

## **计数排序**

计算排序适用场景：数组范围已知且不大，会增加额为的空间O(数组范围)

计数排序的核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。

作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

计数排序的特征：当输入的元素是 n 个 0 到 k 之间的整数时，它的运行时间是 Θ(n + k)。计数排序不是比较排序，排序的速度快于任何比较排序算法。

由于用来计数的数组C的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上1），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。例如：计数排序是用来排序0到100之间的数字的最好的算法，但是它不适合按字母顺序排序人名。但是，计数排序可以用在基数排序中的算法来排序数据范围很大的数组。

通俗地理解，例如有 10 个年龄不同的人，统计出有 8 个人的年龄比 A 小，那 A 的年龄就排在第 9 位，用这个方法可以得到其他每个人的位置，也就排好了序。当然，年龄有重复时需要特殊处理（保证稳定性），这就是为什么最后要反向填充目标数组，以及将每个数字的统计减去 1 的原因

算法的步骤如下：

- 找出待排序的数组中最大和最小的元素
- 统计数组中每个值为i的元素出现的次数，存入数组C的第i项
- 填充目标数组：根据统计数组，依次填充i, 并移动索引。

```java
public void sort(int[] nums) {
    int len = nums.length;

    // find min and max
    int min = nums[0], max = nums[0];
    for (int i = 1; i < len; i ++) {
        if (nums[i] < min) {
            min = nums[i];
        } else if (nums[i] > max) {
            max = nums[i];
        }
    }

    // statistics
    int cLen = max - min + 1;
    int[] bucket = new int[cLen];
    for (int n : nums) {
        bucket[n - min] ++;
    }

    // fill
    // 待填充的索引，从0开始
    int index = 0;
    for (int i = 0; i < cLen; i ++) {
        // 从小到大，逐个将bucket的索引即nums的值填入nums中
        while (bucket[i] > 0) {
            nums[index] = i + min;
            bucket[i] --;
            index ++;
        }
    }
}
```

## 桶排序

计数排序的改进。计数排序中，如果数分布不均匀，会导致，空间极其浪费，比如[1,3, 12345]。这回浪费极大的空间，桶排序可一定程度上减少空间的浪费。

算法过程

- 根据待排序集合中最大元素和最小元素的差值范围和映射规则，确定申请的桶个数；
- 遍历待排序集合，将每一个元素移动到对应的桶中；
- 对每一个桶中元素进行排序，并移动到已排序集合中。

```java
public void sort(int[] nums, int bucketSize) {
    // 构建桶数组
    int[][] buckets = new int[bucketSize][0];
    int min = nums[0], max = nums[0];
    for (int i = 1; i < nums.length; i ++) {
        if (nums[i] < min) {
            min = nums[i];
        } else if (nums[i] > max) {
            max = nums[i];
        }
    }
    int bucketSpan = (max - min) / bucketSize + 1;
    for (int val : nums) {
        int index = (val - min) / bucketSpan;
        buckets[index] = arrAppend(buckets[index], val);
    }
    // 排序
    int index = 0;
    for (int[] bucket : buckets) {
        // 桶内排序
        if (bucket.length == 0) {
            continue;
        }
        Arrays.sort(bucket);
        for (int b : bucket) {
            nums[index] = b;
            index ++;
        }
    }
}

private int[] arrAppend(int[] arr, int val) {
    arr = Arrays.copyOf(arr, arr.length + 1);
    arr[arr.length - 1] = val;
    return arr;
}
```

## 基数排序

桶排序的改进，适用于整数和字符序列排序。

无论是最高位优先法还是最低位优先法进行遍历需要保证数字各位、十位、百位等对齐，这里我使用最低位优先法从个位开始向上。

数字类型的基数排序需要十个桶(0-9)，你可以使用二维数组，第一维度长度为10表示十个数字，第二个维度为数组长度，用来存储数字(因为最坏情况可能当前位数字一样)。但这样无疑太浪费内存空间了，你可以使用List或者Queue替代，这里就用List了。

具体实现要先找到最大值确定最高多少位，用来进行遍历时候确认。

收集的时候借助一个自增参数遍历收集。

每次收集完毕十个桶(bucket)需要清空待下次收集。

```java
public void sort(int[] nums) {
    // 以正整数为例

    // 最大值
    int max = nums[0];
    for (int val : nums) {
        if (max < val) {
            max = val;
        }
    }

    // 遍历排序
    int dev = 1;
    while (max > 0) {
        // 构造桶数组。因为桶要重复利用，所以每次都需要置空
        int[][] buckets = new int[10][0];
        // dev位数的入桶
        for (int val : nums) {
            int index = (val / dev) % 10;
            buckets[index] = arrAppend(buckets[index], val);
        }
        // 出桶排序
        int pos = 0;
        for (int[] bucket : buckets) {
            for (int b : bucket) {
                nums[pos] = b;
                pos ++;
            }
		    }

        dev *= 10;
        max /= 10;
    }
}

private int[] arrAppend(int[] arr, int val) {
    arr = Arrays.copyOf(arr, arr.length + 1);
    arr[arr.length - 1] = val;
    return arr;
}
```

# 堆排序

堆排序（Heapsort），是指利用堆这种数据结构所设计的一种排序算法。

堆积是一个**近似完全二叉树的结构**，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。堆排序可以说是一种利用堆的概念来排序的选择排序。分为两种方法：

- 大顶堆：每个节点的值都大于或等于其子节点的值，在堆排序算法中用于升序排列；
- 小顶堆：每个节点的值都小于或等于其子节点的值，在堆排序算法中用于降序排列；

堆排序的**平均时间复杂度为 Ο(nlogn)**。

算法步骤

- 创建一个堆 H[0……n-1]；
- 把堆首（最大值）和堆尾互换；
- 把堆的尺寸缩小 1，并调用 shift_down(0)，目的是把新的数组顶端数据调整到相应位置；
- 重复步骤 2，直到堆的尺寸为 1。

```java
public void sort(int[] arr) {
    int len = arr.length;
    // 构建最大堆
    buildMaxHeap(arr, len); // n * log(n)
    // 排序 最大索引的无序节点与根节点交换值 n * log(n)
    for (int i = len - 1; i > 0; i --) {
        swap(arr, 0, i);
        len --;
        heapify(arr, 0, len);
    }
}

private void buildMaxHeap(int[] arr, int len) {
    // 从倒数第二层节点开始向下处理
    for (int i = len / 2; i >= 0; i --) {
        heapify(arr, i, len);
    }
}

private void heapify(int[] arr, int i, int len) {
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    int largest = i;
    if (left < len && arr[left] > arr[largest]) {
        largest = left;
    }
    if (right < len && arr[right] > arr[largest]) {
        largest = right;
    }
    if (largest != i) {
        swap(arr, i, largest);
        // 由于larget(left or right)的值改变了，所以需要向下遍历
        heapify(arr, largest, len);
    }
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

