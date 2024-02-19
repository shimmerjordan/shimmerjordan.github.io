---
title: TopK算法杂记
date: 2022-02-26 11:39:17
tags: 
    - Algorithm
categories: ['Algorithm']
id: topk-algo
---

# 概述

TopK的字面意思很简单，就是求给定序列的前$k$大（or小）的元素。本文由简单思路入手，对解决方案逐步发散与优化。为简化叙述，本文讨论的均为最大标准下的TopK（最小标准同理）。

<!-- more -->

# 1. 全局排序

这是最基础的想法，也是最容易实现的。只需要将给定序列进行全局排序后，取出最大的前$k$个即为所求。这里的排序算法选择可以参考[八大排序算法笔记](https://blog.shimmerjordan.eu.org/2022/02/25/8SortingAlgorithms/)，其最优的时间复杂度为$O(nlogn)$。

# 2. 局部排序

全局排序的时间复杂度之所以高，是因为对给定序列中除TopK的元素外进行了不必要的排序。那么我们就可以考虑使用局部排序的思想来优化算法。

我们需要尽可能只对前$k$个最大数进行排序操作，或者只进行找到前$k$个最大数的操作。显然冒泡是一个很好的选择，因为每一趟的冒泡都可以找到当前序列的子序列最大值，也就可以保证在$k$趟冒泡的时间内找出TopK。

**代码实现：**

```python
def bubbleTopk(target, k):
    for i in range(len(target)):
        if i == k-1:
            return target[-k:][::-1]
        for j in range(len(target)-i-1):
            if target[j] > target[j+1]:
                target[j], target[j+1] = target[j+1], target[j]

target = [3,1,5,7,2,4,9,6,10,8]
print(bubbleTopk(target,4))
```

**时间复杂度：**$O(nk)$

# 3. 堆

可以发现，在上面的冒泡过程中，我们还是对TopK进行了排序，但实际上只需要取出TopK而不需要进行TopK的排序。这就引出利用堆优化的TopK算法：

**基本思路：**

1. 先用前$k$个元素生成一个小顶堆，这个小顶堆用于存储，当前最大的$k$个元素；

2. 接着，从第$k+1$个元素开始扫描，和堆顶（堆中最小的元素）比较，如果被扫描的元素大于堆顶，则替换堆顶的元素，并调整堆，以保证堆内的$k$个元素，总是当前最大的$k$个元素；

3. 直到扫描完所有$n-k$个元素，最终堆中的$k$个元素，就是所需的TopK。

**代码实现：**

这里直接调用了python库[`heapq`](https://docs.python.org/zh-cn/3/library/heapq.html)，其中`replace`方法弹出并返回 `heap`中最小的一项，同时推入新的`item`。 堆的大小不变。

```python
import heapq
def heapqTopk(target, k):
    heap = []
    for i in range(k):
        heapq.heappush(heap, target[i])
    for j in range(k+1,len(target)):
        if target[j] > heap[0]:
            heapq.heapreplace(heap, target[j])
    return heap
```

**时间复杂度：**

$n$个元素扫一遍，假设运气很差，每次都入堆调整，调整时间复杂度为堆的高度，即$log(k)$，故整体时间复杂度是$O(nlogk)$。

# 4. 随机选择+partition

堆，将冒泡的TopK排序优化为了TopK不排序，节省了计算资源。堆，是求TopK的经典算法，那还有没有更快的方案呢？

我们知道快速排序的核心思想是**分治法**：

> 分治法（Divide&Conquer），把一个大的问题，转化为若干个子问题（Divide），每个子问题“都”解决，大的问题便随之解决（Conquer）。这里的关键词是“都”。从伪代码里可以看到，快速排序递归时，先通过partition把数组分隔为两个部分，两个部分“都”要再次递归。

分治法有一个特例，叫**减治法**:

> **减治法**（Reduce&Conquer），把一个大的问题，转化为若干个子问题（Reduce），这些子问题中“**只**”解决一个，大的问题便随之解决（Conquer）。这里的关键词是 **“只”** 。

通过分治法与减治法的描述，可以发现，分治法（快排$O(nlogn)$）的复杂度一般来说是大于减治法（二分$O(logn)$）的。我们或许可以利用利用减治法来简化分治法的终止边界条件，最终达到简化算法的目的。

我们知道，**快排**的算法核心`partition(arr, low, high)`会把序列`arr`分为两个部分（默认`privot = arr[low]`作为划分依据）：左半部分都比`privot`大，右半部分都比`privot`小。而且`partition`返回的是`privot`的最终位置`index`。

**`partition`和TopK问题有什么关系呢？**

TopK是希望求出`arr[1,n]`中最大的k个数，那如果找到了**第$k$大**的数，做一次`partition`，不就一次性找到最大的$k$个数了么？问题变成了`arr[1, n]`中找到第$k$大的数。

再回过头来看看**第一次**`partition`，划分之后：

`index = partition(arr, 1, n);`

如果`index`大于`k`，则说明`arr[index]`左边的元素都大于`k`，于是只递归`arr[1, index-1]`里第k大的元素即可；

如果`index`小于`k`，则说明说明第`k`大的元素在`arr[index]`的右边，于是只递归`arr[index+1, n]`里第`k-index`大的元素即可；

这就是**随机选择**算法Randomized Select，其**代码实现**如下：

```python
def partition(target, low, high):
    privot = target[low]
    i, j = low, high
    while i < j:
        while i < j and target[j] <= privot:
            j -= 1
        while i < j and target[i] >= privot:
            i += 1
        target[i], target[j] = target[j], target[i]
    target[i], target[low] = target[low], target[i]
    return i

def randomSelectTopk(target, k):
    left, right = 0, len(target)-1
    index = -1
    while k != index+1:
        index = partition(target, left, right)
        if index > k:
            right = index - 1
        elif index < k:
            left = index + 1
        else:
            return target[:k]

target = [3,1,5,7,2,4,9,6,10,8]
print(randomSelectTopk(target,4))
```

**时间复杂度：** 这是一个典型的减治算法，递归内的两个分支，最终只会执行一个，它的时间复杂度是$O(n)$。

# Summary

TopK不难；其思路优化过程不简单：知其然，知其所以然。思路比结论重要。

- 全局排序，$O(nlogn)$

- 局部排序，只排序TopK个数，$O(nk)$

- 堆，TopK个数也不排序了，$O(nlogk)$

- 随机选择+`partition`：利用分治和减治思想优化，$O(n)$