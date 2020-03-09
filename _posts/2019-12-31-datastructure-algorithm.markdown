---
layout: post
title:  "【DSA】查找与排序"
crawlertitle: "查找与排序"
summary: "数据结构与算法——查找与排序"
date:   2019-12-31 09:00:00 +0800
categories: posts
tags: 'CSSE'
author: xusc
---

查找与排序算法

### 查找
Java中的实现：
- java.util.TreeMap：红黑树
- java.util.HashMap：拉链法的散列表

#### 二叉查找树
二叉查找树（BST）是一颗二叉树，并且每个节点的值都大于等于其左子树中的所有节点的值而小于等于右子树的所有节点的值。BST 有一个重要性质，就是它的中序遍历结果递增排序。

***删除***：如果待删除的节点只有一个子树，那么只需要让指向待删除节点的链接指向唯一的子树即可；否则，让右子树的最小节点替换该节点。
```Java
private Node min(Node x) {
    if (x == null)
        return null;
    if (x.left == null)
        return x;
    return min(x.left);
}
private Node delete(Node x, Key key) {
    if (x == null)
        return null;
    int cmp = key.compareTo(x.key);
    if (cmp < 0)
        x.left = delete(x.left, key);
    else if (cmp > 0)
        x.right = delete(x.right, key);
    else {
        if (x.right == null)
            return x.left;
        if (x.left == null)
            return x.right;
        Node t = x;
        x = min(t.right);
        x.right = deleteMin(t.right);
        x.left = t.left;
    }
    recalculateSize(x);
    return x;
}
```

#### 平衡二叉树
平衡二叉树（Balanced Binary Tree）具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过 1，并且左右两个子树都是一棵平衡二叉树。常用算法有红黑树、AVL、Treap、伸展树等。在平衡二叉搜索树中，我们可以看到，其高度一般都良好地维持在 O(log(n))，大大降低了操作的时间复杂度。

***旋转***
1. 单向右旋平衡处理 LL。
2. 单向左旋平衡处理 RR。
3. 双向旋转（先左后右）平衡处理 LR。
4. 双向旋转（先右后左）平衡处理 RL。

```Java
// 左旋转函数
void left_rotation(Node node) {
    Node tmp;
    int tmp_key;
    tmp = node.left;
    tmp_key = node.key;
    node.left = node.right;
    node.key = node.right.key;
    node.right = node.left.right;
    node.left.right = node.left.left;
    node.left.left = tmp;
    node.left.key = tmp_key;
}
// 右旋转函数
void right_rotation(Node node) {
    Node tmp;
    int tmp_key;
    tmp=node.right;
    tmp_key = node.key;
    node.right = node.left;
    node.key = node.left.key;
    node.left = node.right.left;
    node.right.left = node.right.right;
    node.right.right = tmp;
    node.right.key = tmp_key;
}
```

##### 2-3查找树
2-3 查找树引入了 2- 节点和 3- 节点，目的是为了让树平衡。一颗完美平衡的 2-3 查找树的所有空链接到根节点的距离应该是相同的。

##### 红黑树
红黑树是 2-3 查找树，但它不需要分别定义 2- 节点和 3- 节点，而是在普通的二叉查找树之上，为节点添加颜色。指向一个节点的链接颜色如果为红色，那么这个节点和上层节点表示的是一个 3- 节点，而黑色则是普通链接。

性质：
1. 每个节点要么是黑色，要么是红色。
2. 根节点是黑色。
3. 每个叶子节点（NIL）是黑色。
4. 每个红色结点的两个子结点一定都是黑色。
5. 任意一结点到每个叶子结点的路径都包含数量相同的黑结点。

#### 散列表
散列函数：直接定址法、除留余数法、数字分析法、平方取中法、折叠法。

冲突处理：
1. 开放寻址法
   1. 线性探测法：使用空位来解决冲突，当冲突发生时，向前探测一个空位来存储冲突的键。
   2. 二次探测法。
   3. 双重散列法。
2. 链接法
   1. 拉链法：首先查找 Key 所在的链表，然后在链表中顺序查找。

**再散列**（rehashing）：如果散列表满了，再往散列表中插入新的元素时候就会失败。这个时候可以通过创建另外一个散列表，使得新的散列表的长度是当前散列表的2倍多一些，重新计算各个元素的hash值，插入到新的散列表中。

#### 并查集
并查集是一种树型的数据结构，用于处理一些不相交集合的合并及查询问题。常常在使用中以森林来表示。
1. 合并(Union/Merge)：合并两个集合。
    ```Java
    int Find(int x) {
        if(x == pre[x])
            return x;
        return pre[x] = Find(pre[x]);
    }
    ```
2. 查询(Find/Get)：查询元素所属集合。
    ```Java
    void merge(int x, int y) {
        int fx = Find(x), fy = Find(y);
        if(fx != fy)
            pre[fx] = fy;
    }
    ```



### 排序

#### 选择排序
从数组中选择最小元素，将它与数组的第一个元素交换位置。再从数组剩下的元素中选择出最小的元素，将它与数组的第二个元素交换位置。不断进行这样的操作，直到将整个数组排序。

```Java
public void selectionSort(T[] nums) {
    int N = nums.length;
    for (int i = 0; i < N - 1; i++) {
        int min = i;
        for (int j = i + 1; j < N; j++)
            if (nums[j] < nums[min])
                min = j;
        swap(nums, i, min);
    }
}
```

#### 冒泡排序
从左到右不断交换相邻逆序的元素，在一轮的循环之后，可以让未排序的最大元素上浮到右侧。

```Java
public void bubbleSort(T[] nums) {
    int N = nums.length;
    boolean isSorted = false;
    for (int i = N - 1; i > 0 && !isSorted; i--) {
        isSorted = true;
        for (int j = 0; j < i; j++) {
            if (nums[j + 1] < nums[j)) {
                isSorted = false;
                swap(nums, j, j + 1);
            }
        }
    }
}
```

#### 插入排序
每次都将当前元素插入到左侧已经排序的数组中，使得插入之后左侧数组依然有序。

```Java
public void insertionSort(T[] nums) {
    int N = nums.length;
    for (int i = 1; i < N; i++) {
        for (int j = i; j > 0 && nums[j] < nums[j - 1]; j--)
            swap(nums, j, j - 1);
    }
}
```

#### 希尔排序
使用插入排序对间隔 h 的序列进行排序。通过不断减小 h，最后令 h=1，就可以使得整个数组是有序的。

```Java
public void shellSort(T[] nums) {
    int N = nums.length;
    int h = 1;
    while (h < N / 3)
        h = 3 * h + 1; // 1, 4, 13, 40, ...
    while (h >= 1) {
        for (int i = h; i < N; i++) {
            for (int j = i; j >= h && nums[j] < nums[j - h]; j -= h)
                swap(nums, j, j - h);
        }
        h = h / 3;
    }
}
```

#### 归并排序
将数组分成两部分，分别进行排序，然后归并起来。

```Java
private void merge(T[] nums, int l, int m, int h) {
    T[] aux;
    int i = l, j = m + 1;
    for (int k = l; k <= h; k++)
        aux[k] = nums[k]; // 将数据复制到辅助数组
    for (int k = l; k <= h; k++) {
        if (i > m) 
            nums[k] = aux[j++];
        else if (j > h)
            nums[k] = aux[i++];
        else if (aux[i] <= aux[j])
            nums[k] = aux[i++]; // 先进行这一步，保证稳定性
        else
            nums[k] = aux[j++];
    }
}
// 自顶向下
public void mergeSort(T[] nums) {
    aux = (T[]) new Comparable[nums.length];
    sort(nums, 0, nums.length - 1);
}
private void mergeSort(T[] nums, int l, int h) {
    if (h <= l)
        return;
    int mid = l + (h - l) / 2;
    sort(nums, l, mid);
    sort(nums, mid + 1, h);
    merge(nums, l, mid, h);
}
// 自底向上
public void mergeSort(T[] nums) {
    int N = nums.length;
    aux = (T[]) new Comparable[N];
    for (int sz = 1; sz < N; sz += sz)
        for (int lo = 0; lo < N - sz; lo += sz + sz)
            merge(nums, lo, lo + sz - 1, Math.min(lo + sz + sz - 1, N - 1));
}
```

#### 快速排序
通过一个切分元素将数组分为两个子数组，左子数组小于等于切分元素，右子数组大于等于切分元素，将这两个子数组排序也就将整个数组排序了。

```Java
private int partition(T[] nums, int l, int h) {
    int i = l, j = h + 1;
    T v = nums[l];
    while (true) {
        while (nums[++i] < v && i != h) ;
        while (v < nums[--j] && j != l) ;
        if (i >= j)
            break;
        swap(nums, i, j);
    }
    swap(nums, l, j);
    return j;
}
public void quickSort(T[] nums) {
    sort(nums, 0, nums.length - 1);
}
private void quickSort(T[] nums, int l, int h) {
    if (h <= l)
        return;
    int j = partition(nums, l, h);
    sort(nums, l, j - 1);
    sort(nums, j + 1, h);
}
```

##### 切分的改进
```Java
protected void sort(T[] nums, int l, int h) {
    if (h <= l)
        return;
    int lt = l, i = l + 1, gt = h;
    T v = nums[l];
    while (i <= gt) {
        int cmp = nums[i].compareTo(v);
        if (cmp < 0)
            swap(nums, lt++, i++);
        else if (cmp > 0)
            swap(nums, i, gt--);
        else
            i++;
    }
    sort(nums, l, lt - 1);
    sort(nums, gt + 1, h);
}
```

#### 堆排序
把最大元素和当前堆中数组的最后一个元素交换位置，并且不删除它，那么就可以得到一个从尾到头的递减序列，从正向来看就是一个递增序列，这就是堆排序。

```Java
public void sort(T[] nums) {
    int N = nums.length - 1;
    for (int k = N / 2; k >= 1; k--)
        sink(nums, k, N);
    while (N > 1) {
        swap(nums, 1, N--);
        sink(nums, 1, N);
    }
}
private void sink(T[] nums, int k, int N) {
    while (2 * k <= N) {
        int j = 2 * k;
        if (j < N && nums[j] < nums[j+1])
            j++;
        if (nums[k] < nums[j])
            break;
        swap(nums, k, j);
        k = j;
    }
}
```

#### 排序算法比较

算法|稳定性|时间复杂度|空间复杂度|备注
:-:|:-:|:-:|:-:|:-
选择排序|×|N2|1| 
冒泡排序|√|N2|1| 
插入排序|√|N ~ N2|1|时间复杂度和初始顺序有关
希尔排序|×|N 的若干倍乘于递增序列的长度|1|改进版插入排序
快速排序|×|NlogN|logN| 
三向切分快速排序|×|N ~ NlogN|logN|适用于有大量重复主键
归并排序|√|NlogN|N| 
堆排序|×|NlogN|1|无法利用局部性原理

Java 主要排序方法为 java.util.Arrays.sort()，对于原始数据类型使用三向切分的快速排序，对于引用类型使用归并排序。
