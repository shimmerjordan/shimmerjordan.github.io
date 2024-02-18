---
title: 八大排序算法笔记
date: 2022-02-25 11:16:10
tags: 
    - Algorithm
categories: ['Algorithm']
id: 8SortingAlgorithms

---

# 概述

排序有内部排序和外部排序，内部排序是数据记录在内存中进行排序，而外部排序是因排序的数据很大，一次不能容纳全部的排序记录，在排序过程中需要访问外存。这里介绍的是内部排序中的八大主流排序算法：

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/1.jpg" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">算法结构</div>
</center>

当$n$较大，则应采用时间复杂度为$O(nlog2n)$的排序方法：快速排序、堆排序或归并排序序。其中快速排序是目前基于比较的内部排序中被认为是最好的方法，当待排序的关键字是随机分布时，快速排序的**平均**时间最短**但不稳定**。

<!--more-->

# <a id="insertSort">插入排序—直接插入排序</a>

Note：设立一个作为临时存储和判断数组边界的哨兵。

**基本思想**

将一个记录插入到已排序好的有序表中，从而得到一个有序子表。即：先将序列的第1个记录看成是一个有序的子序列，然后从第2个记录逐个进行插入，直至整个序列有序为止（$n-1$次操作）。

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/2.jpg" width='50%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">直接插入排序</div>
</center>

Tip：如果碰见一个和插入元素相等的，那么插入元素把想插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序就是排好序后的顺序，**所以插入排序是稳定的。**

**算法实现**

```python
def insertSort(target):
    for i in range(1, len(target)):
        if target[i] < target[i-1]:     # 若第i个元素大于i-1元素，直接插入。小于的话，移动有序表后插入
            temp = target[i]            # 复制为哨兵，即存储待排序元素
            target[i] = target[i-1]
            j = i - 1
            while temp < target[j]:     # 查找在有序表的插入位置
                target[j+1] = target[j]
                j -= 1                  # 元素后移
                if j < 0: break
            target[j+1] = temp          # 插入到正确位置
    return target

target = [3,1,5,7,2,4,9,6]
print(insertSort(target))
```

时间复杂度为$O(n^2)$，其它的插入排序有二分插入排序，2-路插入排序。

# 插入排序—希尔排序

1959 年由D.L.Shell 提出来的，相对直接排序有较大的改进，又称**缩小增量排序**

**基本思想**

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录“基本有序”时，再对全体记录进行依次直接插入排序。

1. 选择一个增量序列$t_1,t_2, \dots ,t_k$，其中$t_i>t_j$，$t_k=1$；

2. 按增量序列个数$k$，对序列进行$k$趟排序；

3. 每趟排序，根据对应的增量$t_i$，将待排序列分割成若干长度为$m$的子序列，分别对各子表进行直接插入排序。仅增量因子为$1$时，整个序列作为一个表来处理，表长度即为整个序列的长度。

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/3.jpg" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">希尔排序</div>
</center>

**算法实现**

我们简单处理增量序列：增量序列$d = \{n/2,n/4,n/8, \cdots, 1\}$，$n$为要排序数的个数。

```python
def shellInsertSort(target, dk):
    for i in range(dk, len(target)):
        if target[i] < target[i-dk]:
            j = i-dk
            temp = target[i]
            target[i] = target[i-dk]
            while temp < target[j]:
                target[j+dk] = target[j]
                j -= dk
                if j < 0: break
            target[j+dk] = temp
    return target

def shellSort(target):
    dk = len(target) // 2
    while dk >= 1:
        shellInsertSort(target, dk)
        dk //= 2
    return target

target = [3,1,5,7,2,4,9,6]
print(shellSort(target))
```

**复杂度**：希尔排序时效分析很难，关键码的比较次数与记录移动次数依赖于增量因子序列$d$的选取，特定情况下可以准确估算出关键码的比较次数和记录的移动次数。目前还没有人给出选取最好的增量因子序列的方法。增量因子序列可以有各种取法，有取奇数的，也有取质数的，但需要注意：增量因子中除$1$外没有公因子，且最后一个增量因子必须为$1$。希尔排序方法是一个**不稳定的排序方法**。

# 选择排序—简单选择排序

**基本思想**

在要排序的一组数中，选出最小（或者最大）的一个数与第$1$个位置的数交换；然后在剩下的数当中再找最小（或者最大）的与第$2$个位置的数交换，依次类推，直到第 $n-1$个元素（倒数第二个数）和第$n$个元素（最后一个数）比较为止。

第$1$趟，从$n$个记录中找出关键码最小的记录与第$1$个记录交换；

第$2$趟，从第$2$个记录开始的$n-1$个记录中再选出关键码最小的记录与第$2$个记录交换；

$\cdots$

第$i$趟，则从第$i$个记录开始的$n-i+1$个记录中选出关键码最小的记录与第$i$个记录交换，直到整个序列按关键码有序。
**算法实现**

```python
def minIndex(target, start):
    res = start 
    min_num = target[start]
    for i in range(start, len(target)):
        if target[i] < min_num:
            min_num = target[i]
            res = i
    return res

def selectSort(target):
    for i in range(len(target)):
        key = minIndex(target, i)
        if key != i:
            target[i], target[key] = target[key], target[i]
    return target

target = [3,1,5,7,2,4,9,6]
print(selectSort(target))
```

**简单选择排序的改进——二元选择排序**

简单选择排序，每趟循环只能确定一个元素排序后的定位。我们可以考虑改进为每趟循环确定两个元素（当前趟最大和最小记录）的位置,从而减少排序所需的循环次数。改进后对$n$个数据进行排序，最多只需进行$[n/2]$趟循环即可。具体实现不再赘述。

# 选择排序—堆排序

堆排序是一种树形选择排序，是对直接选择排序的有效改进。`python`内部实现了最小堆[heapq优先队列](https://docs.python.org/zh-cn/3/library/heapq.html)

**基本思想**

初始时把要排序的$n$个数的序列看作是一棵顺序存储的二叉树（一维数组存储二叉树），调整它们的存储序，使之成为一个堆，将堆顶元素输出，得到$n$个元素中最小(或最大)的元素，这时堆的根节点的数最小（或者最大）。然后对前面$n-1$个元素重新调整使之成为堆，输出堆顶元素，得到$n$个元素中次小(或次大)的元素。依此类推，直到只有两个节点的堆，并对它们作交换，最后得到有$n$个节点的有序序列。称这个过程为**堆排序**。
因此，实现堆排序需解决两个问题：  

1. 如何将$n$个待排序的数建成堆；  
2. 输出堆顶元素后，怎样调整剩余$n-1$个元素，使其成为一个新堆。

首先讨论第二个问题：输出堆顶元素后，对剩余$n-1$元素重新建成堆的调整过程。
**调整小顶堆的方法：**

1) 设有$m$个元素的堆，输出堆顶元素后，剩下$m-1$个元素。将堆底元素送入堆顶（最后一个元素与堆顶进行交换），堆被破坏，其原因仅是根结点不满足堆的性质。

2) 将根结点与左、右子树中较小元素的进行交换。

3) 若与左子树交换：如果左子树堆被破坏，即左子树的根结点不满足堆的性质，则重复方法`2`.

4) 若与右子树交换：如果右子树堆被破坏，即右子树的根结点不 满足堆的性质。则重复方法`2`.

5) 继续对不满足堆性质的子树进行上述交换操作，直到叶子结点，堆被建成。

再讨论对$n$个元素初始建堆的过程。

**建堆方法：** 对初始序列建堆的过程，就是一个反复进行筛选的过程。

1. $n$个结点的完全二叉树，则最后一个结点是第$[n/2]$个结点的子树。

2. 筛选从第$[n/2]$个结点为根的子树开始，该子树成为堆。

3. 之后向前依次对各结点为根的子树进行筛选，使之成为堆，直到根结点。

**算法的实现：**

从算法描述来看，堆排序需要两个过程，一是建立堆，二是堆顶与堆的最后一个元素交换位置。所以堆排序有两个函数组成。一是建堆的渗透函数，二是反复调用渗透函数实现排序的函数。

```python
# 构造最小堆 
class MinHeap():
    def __init__(self, maxSize=None):
        self.maxSize = maxSize 
        self.array = [None] * maxSize 
        self._count = 0

    def length(self):
        return self._count 

    def show(self):
        if self._count <= 0:
            print('null')
        print(self.array[: self._count], end=', ')

    def add(self, value):
        # 增加元素
        if self._count >= self.maxSize:
            raise Exception('The array is Full')
        self.array[self._count] = value
        self._shift_up(self._count)
        self._count += 1

    def _shift_up(self, index):
        # 比较结点与根节点的大小， 较小的为根结点
        if index > 0:
            parent = (index - 1) // 2
            if self.array[parent] > self.array[index]:
                self.array[parent], self.array[index] = self.array[index], self.array[parent]
                self._shift_up(parent)

    def extract(self):           
        # 获取最小值，并更新数组
        if self._count <= 0:
            raise Exception('The array is Empty')
        value = self.array[0]
        self._count -= 1 # 更新数组的长度
        self.array[0] = self.array[self._count] # 将最后一个结点放在前面
        self._shift_down(0)

        return value 

    def _shift_down(self, index):  
        # 此时index 是根结点
        if index < self._count:
            left = 2 * index + 1
            right = 2 * index + 2
            # 判断左右结点是否越界，是否小于根结点，如果是就交换
            if left < self._count and right < self._count and self.array[left] < self.array[index] and self.array[left] < self.array[right]:
                self.array[index], self.array[left] = self.array[left], self.array[index] #交换得到较小的值
                self._shift_down(left)
            elif left < self._count and right < self._count and self.array[right] < self.array[left] and self.array[right] < self.array[index]:
                self.array[right], self.array[index] = self.array[index], self.array[right]
                self._shift_down(right)

            # 特殊情况： 如果只有做叶子结点
            if left < self._count and right > self._count and self.array[left] < self.array[index]:
                self.array[left], self.array[index] = self.array[index], self.array[left]
                self._shift_down(left)

mi = MinHeap(10)
num = [3,1,5,7,2,4,9,6,10,8]
print('---------小顶堆----------')
for i in num:
    mi.add(i)
res = []
for _ in range(len(num)):
    res.append(mi.extract())
print(res)
```

**复杂度：**

设树深度为$k=[log_2n]+1$，从根到叶的筛选，元素比较次数至多$2(k-1)$次，交换记录至多$k$次。所以，在建好堆后，排序过程中的筛选次数不超过:

$$
2(\lfloor log_2(n-1)\rfloor+\lfloor log_2(n-2)\rfloor+,\cdots,+\lfloor log_22\rfloor)<2n(\lfloor log_2n\rfloor)
$$

而建堆时的比较次数不超过$4n$次，因此堆排序最坏情况下，时间复杂度也为：$O(nlogn )$

# 交换排序—冒泡排序

**基本思想：**

在要排序的一组数中，对当前还未排好序的范围内的全部数，自上而下对相邻的两个数依次进行比较和调整，让较大的数往下沉，较小的往上冒。即：每当两相邻的数比较后发现它们的排序与排序要求相反时，就将它们互换。

**算法的实现：**

```python
def bubbleSort(target):
    for i in range(len(target)):
        for j in range(len(target)-i-1):
            if target[j] > target[j+1]:
                target[j], target[j+1] = target[j+1], target[j]
    return target  
target = [3,1,5,7,2,4,9,6,10,8]
print(bubbleSort(target))
```

**冒泡排序算法的改进:**

对冒泡排序常见的改进方法是加入一标志性变量`exchange`，用于标志某一趟排序过程中是否有数据交换，如果进行某一趟排序时并没有进行数据交换，则说明数据已经按要求排列好，可立即结束排序，避免不必要的比较过程。本文再提供以下两种改进算法：

1．设置一标志性变量`pos`,用于记录每趟排序中最后一次进行交换的位置。由于`pos`位置之后的记录均已交换到位,故在进行下一趟排序时只要扫描到`pos`位置即可。

```python
def bubbleSort_1(target):
    i = len(target) - 1
    while i > 0:
        pos = 0         # 每趟开始前，无记录交换
        for j in range(i):
            if target[j] > target[j+1]:
                pos = j
                target[j], target[j+1] = target[j+1], target[j]
        i = pos
    return target
```

2．传统冒泡排序中每一趟排序操作只能找到一个最大值或最小值,我们考虑利用在每趟排序中进行正向和反向两遍冒泡的方法一次可以得到两个最终值(最大者和最小者) , 从而使排序趟数几乎减少了一半。

```python
def bubbleSort_2(target):
    low, high = 0, len(target) - 1
    while low < high:
        for j in range(low, high):        # 正向冒泡寻找最大
            if target[j] > target[j+1]:
                target[j], target[j+1] = target[j+1], target[j]
        high -= 1
        for j in range(high, low, -1):    # 反向冒泡寻找最小
            if target[j] < target[j-1]:
                target[j], target[j-1] = target[j-1], target[j]
        low += 1
    return target
```

# 交换排序—快速排序

**基本思想：**

1. 选择一个基准元素,通常选择第一个元素或者最后一个元素；

2. 通过一趟排序讲待排序的记录分割成独立的两部分，其中一部分记录的元素值均比基准元素值小。另一部分记录的元素值比基准值大；

3. 此时基准元素在其排好序后的正确位置；

4. 然后分别对这两部分记录用同样的方法继续进行排序，直到整个序列有序。
   
   <center>
     <img style="border-radius: 0.3125em;
     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
     src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/1.gif">
     <br>
     <div style="color:orange; border-bottom: 1px solid #d9d9d9;
     display: inline-block;
     color: #999;
     padding: 2px;">快速排序</div>
   </center>

**算法实现：**

```python
def partition(target, low, high):
    privot = target[low]
    while low < high:
        while low < high and target[high] >= privot:
            high -= 1
        target[low], target[high] = target[high], target[low]
        while low < high and target[low] <= privot:
            low += 1
        target[low], target[high] = target[high], target[low]
    return low

def quickSort(target, low, high):
    if low < high:
        privotLoc = partition(target, low, high)
        quickSort(target, low, privotLoc-1)
        quickSort(target, privotLoc+1, high)
    return target

target = [3,1,5,7,2,4,9,6,10,8]
print(quickSort(target,0,len(target)-1))
```

**分析：**

快速排序是通常被认为在同数量级$O(nlog2n)$的排序方法中平均性能最好的。但若初始序列按关键码有序或基本有序时，快排序反而蜕化为冒泡排序。为改进之，通常以“三者取中法”来选取基准记录，即将排序区间的两个端点与中点三个记录关键码居中的调整为支点记录。快速排序是一个**不稳定**的排序方法。

**快速排序的改进：**

在本改进算法中，只对长度大于$k$的子序列递归调用快速排序，让原序列基本有序，然后再对整个基本有序序列用<a href="#insertSort">插入排序算法</a>排序：

```python
def quickSort_improve(target, low, high, k):
    if high - low > k:
        privotLoc = partition(target, low, high)
        quickSort_improve(target, low, privotLoc-1, k)
        quickSort_improve(target, privotLoc+1, high, k)
    return target

def quickSort(target, k):
    target = quickSort_improve(target, 0, len(target)-1, k)
    return insertSort(target)

target = [3,1,5,7,2,4,9,6,10,8]
print(quickSort(target,8))
```

# 归并排序

**基本思想：**

归并（Merge）排序法是将两个（或两个以上）有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列是有序的。然后再把有序子序列合并为整体有序序列。

**合并方法：**

设`r[i…n]`由两个有序子表`r[i…m]`和`r[m+1…n]`组成，两个子表长度分别为`n-i +1`、`n-m`。

1. `j=m+1`；`k=i`；`i=i`；\# 置两个子表的起始下标及辅助数组的起始下标；

2. 若`i>m` 或`j>n`，转⑷；\# 其中一个子表已合并完，比较选取结束；

3. \# 选取`r[i]`和`r[j]`较小的存入辅助数组`rf`
   如果`r[i]<r[j]`，`rf[k]=r[i]`；`i++`； `k++`； 转⑵
   否则，`rf[k]=r[j]`；` j++`；` k++`； 转⑵

4. \# 将尚未处理完的子表中元素存入rf
   如果`i<=m`，将`r[i…m]`存入`rf[k…n]`  \# 前一子表非空
   如果`j<=n` ,  将`r[j…n]` 存入`rf[k…n]` \# 后一子表非空

5. 合并结束。

**算法实现：**

这里直接实现两路归并的迭代版本：

```python
def merge(a, b):
    c = []
    h = j = 0
    while j < len(a) and h < len(b):
        if a[j] < b[h]:
            c.append(a[j])
            j += 1
        else:
            c.append(b[h])
            h += 1
    if j == len(a):
        for i in b[h:]: c.append(i)
    else:
        for i in a[j:]: c.append(i)
    return c

def merge_sort(target):
    if len(target) <= 1:
        return target
    middle = len(target) // 2
    left = merge_sort(target[:middle])
    right = merge_sort(target[middle:])
    return merge(left, right)

target = [3,1,5,7,2,4,9,6,10,8]
print(merge_sort(target))
```

# 桶排序/基数排序

说基数排序之前，我们先说**桶排序**：

**基本思想：** 是将阵列分到有限数量的桶子里。每个桶子再个别排序（有可能再使用别的排序算法或是以递回方式继续使用桶排序进行排序）。桶排序是鸽巢排序的一种归纳结果。当要被排序的阵列内的数值是均匀分配的时候，桶排序使用线性时间$Θ(n)$。但桶排序并不是比较排序，他不受到$Θ(nlogn)$下限的影响。

简单来说，就是把数据分组，放在一个个的桶中，然后对每个桶里面的在进行排序。  

例如要对大小为$[1,1000]$范围内的$n$个整数`A[1..n]`排序。首先，可以把桶设为大小为`10`的范围，具体而言，设集合`B[1]`存储$[1,10]$的整数，集合`B[2]`存储 $(10,20]$的整数，……集合`B[i]`存储$((i-1) \times 10, i \times 10]$的整数，$i=1,2,..100$。总共有$100$个桶。  

然后，对`A[1..n]`从头到尾扫描一遍，把每个`A[i]`放入对应的桶`B[j]`中。  再对这$100$个桶中每个桶里的数字排序，这时可用冒泡，选择，乃至快排，一般来说任何排序法都可以。

最后，依次输出每个桶里面的数字，且每个桶中的数字从小到大输出，这样就得到所有数字排好序的一个序列了。  

假设有$n$个数字，有$m$个桶，如果数字是平均分布的，则每个桶里面平均有$n/m$个数字。如果对每个桶中的数字采用快速排序，那么整个算法的复杂度是  

$$
O(n + m * n/m*log(n/m)) = O(n + nlogn - nlogm)
$$

$$

从上式看出，当$m$接近$n$的时候，桶排序复杂度接近$O(n)$。当然，以上复杂度的计算是基于输入的$n$个数字是平均分布这个假设的。这个假设是很强的  ，实际应用中效果并没有这么好。如果所有的数字都落在同一个桶中，那就退化成一般的排序了。  

**两种多关键码排序方法：**

多关键码排序按照从最主位关键码到最次位关键码或从最次位到最主位关键码的顺序逐次排序，分两种方法：

最高位优先(Most Significant Digit first)法，简称**MSD**法：

1. 先按`k1`排序分组，将序列分成若干子序列，同一组序列的记录中，关键码`k1`相等。

2. 再对各组按`k2`排序分成子组，之后，对后面的关键码继续这样的排序分组，直到按最次位关键码`kd`对各子组排序后。

3. 再将各组连接起来，便得到一个有序序列。扑克牌按花色、面值排序中介绍的方法一即是MSD法。

最低位优先(Least Significant Digit first)法，简称**LSD**法：

1) 先从`kd`开始排序，再对`kd-1`进行排序，依次重复，直到按`k1`排序分组分成最小的子序列后。

2) 最后将各个子序列连接起来，便可得到一个有序的序列，扑克牌按花色、面值排序中介绍的方法二即是LSD法。

**基于LSD的基数排序:**

按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。有时候有些属性是有优先级顺序的，先按低优先级排序，再按高优先级排序。最后的次序就是高优先级高的在前，高优先级相同的低优先级高的在前。基数排序基于分别排序，分别收集，所以是稳定的。

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/5.jpg" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">基数排序</div>
</center>

**算法实现：**

```python
def radix_sort(s):
    """基数排序"""
    i = 0 # 记录当前正在排拿一位，最低位为1
    max_num = max(s)  # 最大值
    j = len(str(max_num))  # 记录最大值的位数
    while i < j:
        bucket_list =[[] for _ in range(10)] #初始化桶数组
        for x in s:
            bucket_list[int(x / (10**i)) % 10].append(x) # 找到位置放入桶数组
        print(bucket_list)
        s.clear()
        for x in bucket_list:   # 放回原序列
            for y in x:
                s.append(y)
        i += 1

if __name__ == '__main__':
    a = [334,5,67,345,7,345345,99,4,23,78,45,1,3453,23424]
    radix_sort(a)
    print(a)
```

# Summary

**各种排序的稳定性，时间复杂度和空间复杂度对比：**

<center>
  <img style="border-radius: 0.3125em;
  box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
  src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed/blog/8SortingAlg/6.jpg" width='70%'>
  <br>
  <div style="color:orange; border-bottom: 1px solid #d9d9d9;
  display: inline-block;
  color: #999;
  padding: 2px;">算法比较</div>
</center>

**说明：**

- 当原表有序或基本有序时，直接插入排序和冒泡排序将大大减少比较次数和移动记录的次数，时间复杂度可降至$O(n)$；

- 而快速排序则相反，当原表基本有序时，将退化为冒泡排序，时间复杂度提高为$O(n^2)$；

- 原表是否有序，对简单选择排序、堆排序、归并排序和基数排序的时间复杂度影响不大。

**稳定性：**

排序算法的稳定性：若待排序的序列中，存在多个具有相同关键字的记录，经过排序， 这些记录的相对次序保持不变，则称该算法是稳定的；否则不稳定。 

排序算法如果是稳定的，那么从一个键上排序，然后再从另一个键上排序，第一个键排序的结果可以为第二个键排序所用。基数排序就是这样，先按低位排序，逐次按高位排序，低位相同的元素其顺序再高位也相同时是不会改变的。另外，如果排序算法稳定，可以避免多余的比较；

**稳定的排序算法：** 冒泡排序、插入排序、归并排序和基数排序

**非稳定的排序算法：** 选择排序、快速排序、希尔排序、堆排序