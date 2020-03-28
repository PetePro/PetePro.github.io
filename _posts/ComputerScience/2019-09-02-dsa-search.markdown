---
layout: post
title:  "【DSA】数据结构与算法 总结 02 查找"
crawlertitle: "查找"
summary: "Data Structures and Algorithm —— Search"
date:   2019-09-02 10:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

高效检索信息是处理它们的前提，`查找`则是在广泛应用中经过实践检验的经典算法。

### 查找
Java中的实现：
- java.util.TreeMap：红黑树
- java.util.HashMap：拉链法的散列表

#### 二叉查找树
二叉查找树（BST）是一颗二叉树，并且每个节点的值都大于等于其左子树中的所有节点的值而小于等于右子树的所有节点的值。BST 有一个重要性质，就是它的中序遍历结果递增排序。

***删除***：如果待删除的节点只有一个子树，那么只需要让指向待删除节点的链接指向唯一的子树即可；否则，让右子树的最小节点替换该节点。
```java
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
平衡二叉树（Balanced Binary Tree）具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过 1，并且左右两个子树都是一棵平衡二叉树。常用算法有红黑树、AVL、Treap、伸展树等。在平衡二叉搜索树中，我们可以看到，其高度一般都良好地维持在 O(log(n))，大大降低了操作的时间复杂度。最小二叉平衡树的节点的公式：`F(n) = F(n - 1) + F(n - 2) + 1`。

***旋转***
1. 单向右旋平衡处理 LL。
2. 单向左旋平衡处理 RR。
3. 双向旋转（先左后右）平衡处理 LR。
4. 双向旋转（先右后左）平衡处理 RL。

```java
// 左旋转函数
void left_rotation(Node node) {
    Node tmp;
    Key tmp_key;
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
    Key tmp_key;
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

##### 红黑树
红黑树是每个节点都带有颜色属性的二叉查找树。典型的用途是实现关联数组。

性质：
1. 每个节点要么是黑色，要么是红色。
2. 根节点是黑色。
3. 每个叶子节点（NIL）是黑色。
4. 每个红色结点的两个子结点一定都是黑色。
5. 任意一结点到每个叶子结点的路径都包含数量相同的黑结点。

红黑树自平衡的三种操作：
1. 左旋：以某个结点作为支点（旋转结点），其右子结点变为旋转结点的父结点，右子结点的左子结点变为旋转结点的右子结点，左子结点保持不变。
2. 右旋：以某个结点作为支点（旋转结点），其左子结点变为旋转结点的父结点，左子结点的右子结点变为旋转结点的左子结点，右子结点保持不变。
3. 变色：结点的颜色由红变黑或由黑变红。

#### B树与B+树
B树是一种平衡的多路查找树。普遍用在数据库和文件系统。但是它不是二叉树。

一棵 `M` 阶的B树，或为空树，或为满足下列特征的 `M` 叉树：
1. 定义任意非叶子结点最多只有 `M, M>2` 个儿子；
2. 根结点的儿子数为 `[2, M]`；
3. 除根结点以外的非叶子结点的儿子数为 `[M/2, M]`；
4. 每个结点存放至少 `M/2-1（取上整）`和至多 `M-1` 个关键字；（至少2个关键字）
5. 非叶子结点的关键字个数 = 指向儿子的指针个数 - 1；
6. 非叶子结点的关键字：`K[1], K[2], …, K[M - 1]`；且 `K[i] < K[i + 1]`；
7. 所有叶子结点位于同一层；

B+树是对B树的一种变形树，它与B树的差异在于：
1. 非叶子结点的子树指针与关键字个数相同；
2. 为所有叶子结点增加一个链指针；
3. 所有关键字都在叶子结点出现；

B和B+树的区别在于，B+树的非叶子结点只包含导航信息，不包含实际的值，所有的叶子结点和相连的节点使用链表相连，便于区间查找和遍历。B+ 树的优点在于：
- 由于B+树在内部节点上不好含数据信息，因此在内存页中能够存放更多的key。 数据存放的更加紧密，具有更好的空间局部性。因此访问叶子几点上关联的数据也具有更好的缓存命中率。
- B+树的叶子结点都是相链的，因此对整棵树的遍历只需要一次线性遍历叶子结点即可。而且由于数据顺序排列并且相连，所以便于区间查找和搜索。而B树则需要进行每一层的递归遍历。相邻的元素可能在内存中不相邻，所以缓存命中性没有B+树好。


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
    ```java
    int Find(int x) {
        if(x == pre[x])
            return x;
        return pre[x] = Find(pre[x]);
    }
    ```
2. 查询(Find/Get)：查询元素所属集合。
    ```java
    void merge(int x, int y) {
        int fx = Find(x), fy = Find(y);
        if(fx != fy)
            pre[fx] = fy;
    }
    ```

