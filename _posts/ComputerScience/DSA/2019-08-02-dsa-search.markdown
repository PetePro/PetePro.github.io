---
layout: post
title:  "【DSA】数据结构与算法 02 查找"
crawlertitle: "查找"
summary: "Data Structures and Algorithm —— Search"
date:   2019-08-02 10:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

高效检索信息是处理它们的前提，`查找`则是在广泛应用中经过实践检验的经典算法。

### 0 查找
Java中的实现：
- java.util.TreeMap：红黑树
- java.util.HashMap：拉链法的散列表

符号表：将一个键和一个值联系起来。

符号表的实现及优缺点

数据结构|实现|优点|缺点
:-:|:-:|:-:|:-:
无序链表<br/>（顺序查找）|SequentialSearchST|适用于小型问题|对于大型符号表很慢
有序数组<br/>（二分查找）|BinarySearchST|最优的查找效率和空间需求<br/>能够进行有序性相关操作|插入操作很慢
二叉查找树|BST|实现简单<br/>能够进行有序性相关操作|没有性能上界保证<br/>链接需要额外空间
平衡查找树|RedBlackST|最优的查找和插入效率<br/>能够进行有序性相关的操作|链接需要额外的空间
散列表|SeparateChainHashST<br/>LinearProbingHashST|能够快速查找和插入常见类型数据|需要计算每种类型数据的散列<br/>无法进行有序性相关的操作<br/>链接和空结点需要额外的空间




### 1 二叉查找树
二叉查找树（BST）是一颗二叉树，并且每个结点的值都大于等于其左子树中的所有结点的值而小于等于右子树的所有结点的值。BST 有一个重要性质，就是它的中序遍历结果递增排序。

***删除***：如果待删除的结点只有一个子树，那么只需要让指向待删除结点的链接指向唯一的子树即可；否则，让右子树的最小结点替换该结点。
```java
private Node min(Node x) {
    if (x.left == null)
        return x;
    return min(x.left);
}
private Node deleteMin(Node x) {
    if (x.left == null)
        return x.right;
    x.left = deleteMin(x.left);
    x.N = size(x.left) + size(x.right) + 1;
    return x;
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
        x = min(t.right);   // 后继结点
        x.right = deleteMin(t.right);
        x.left = t.left;
    }
    x.N = size(x.left) + size(x.right) + 1;
    return x;
}
```



### 2 平衡二叉树
平衡二叉树（Balanced Binary Tree）具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过 1，并且左右两个子树都是一棵平衡二叉树。常用算法有红黑树、AVL、Treap、伸展树等。在平衡二叉搜索树中，我们可以看到，其高度一般都良好地维持在 O(log(n))，大大降低了操作的时间复杂度。最小二叉平衡树的结点的公式：`F(n) = F(n - 1) + F(n - 2) + 1`。

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

#### 2.1 2-3查找树
- 2- 结点，含有一个键和两条链接
- 3- 结点，含有两个键和三条链接

#### 2.2 红黑树
红黑树是每个结点都带有颜色属性的二叉查找树。典型的用途是实现关联数组。将红黑树的红链接画平即为 2-3 树。
- 红链接：将两个 2- 结点连接起来构成一个 3- 结点
- 黑链接：普通链接

性质：
1. 红链接均为左链接
2. 没有任何一个结点同时和两条红链接相连
3. 完美黑色平衡：任意空链接到根结点的路径上的黑链接数量相同

结点表示
```java
private class Node {
    Key key;            // 键
    Value val;          // 值
    Node left, right;   // 左右子树
    int N;              // 子树中结点的个数
    boolean color;      // 父结点指向它的链接的颜色
}
```

主要操作
+ 旋转
  + 左旋转：将一条红色的右链接转化为左链接 —— 将两个键中较大者作为根结点，并交换颜色。
  + 右旋转：将一条红色的左链接转化为右链接 —— 将两个键中较小者作为根结点，并交换颜色。
+ 颜色变换：将子结点颜色由红变黑后，将父结点颜色由黑变红
  + 根结点总是黑色：每当根结点由红变黑后，树的黑链接高度加 1
+ 规则
  + 如果右子结点是红色而左子结点是黑色，进行左旋转
  + 如果左子结点是红色且它的左子结点也是红色，进行右旋转
  + 如果左右子结点均为红色，进行颜色转换
+ 插入
  + 向 2- 结点插入
    + 左插：用红链接和新结点连接，2- 结点变成 3- 结点
    + 右插：用红链接和新结点连接，左旋转后变成 3- 结点
  + 向 3- 结点插入
    + 新键大于原树中两个键：用红链接和新结点连接，将两条链接颜色都由红变黑
    + 新键小于原树中两个键：用红链接和新结点连接，旋转后出现红色右链接，将两条链接颜色都由红变黑
    + 新键介于原树两键之间：用红链接和新结点连接，旋转两次后出现红色右链接，将两条链接颜色都由红变黑
+ 删除

#### 2.3 B树与B+树
B 树是一种平衡的多路查找树。普遍用在数据库和文件系统。但是它不是二叉树。

一棵 `M` 阶的B树，或为空树，或为满足下列特征的 `M` 叉树：
1. 定义任意非叶子结点最多只有 `M, M>2` 个儿子；
2. 根结点的儿子数为 `[2, M]`；
3. 除根结点以外的非叶子结点的儿子数为 `[M/2, M]`；
4. 每个结点存放至少 `M/2-1（取上整）`和至多 `M-1` 个关键字；（至少2个关键字）
5. 非叶子结点的关键字个数 = 指向儿子的指针个数 - 1；
6. 非叶子结点的关键字：`K[1], K[2], …, K[M - 1]`；且 `K[i] < K[i + 1]`；
7. 所有叶子结点位于同一层；

B+ 树是对 B 树的一种变形树，它与 B 树的差异在于：
1. 非叶子结点的子树指针与关键字个数相同；
2. 非叶子结点只包含导航信息，不包含实际的值；
3. 所有的叶子结点和相连的结点使用链表相连；
4. 所有关键字都在叶子结点出现；

B+ 树的优点在于：
- 由于 B+ 树在内部结点上不好含数据信息，因此在内存页中能够存放更多的key。 数据存放的更加紧密，具有更好的空间局部性。因此访问叶子几点上关联的数据也具有更好的缓存命中率。
- B+ 树的叶子结点都是相链的，因此对整棵树的遍历只需要一次线性遍历叶子结点即可。而且由于数据顺序排列并且相连，所以便于区间查找和搜索。而 B 树则需要进行每一层的递归遍历。相邻的元素可能在内存中不相邻，所以缓存命中性没有 B+ 树好。




### 3 散列表
散列函数：直接定址法、除留余数法、数字分析法、平方取中法、折叠法。

冲突处理：
1. 开放地址法
   1. 线性探测法：当冲突发生时，向前探测一个空位来存储冲突的键。
   2. 二次探测法。
   3. 双重散列法。
2. 链接法
   1. 拉链法：首先查找 Key 所在的链表，然后在链表中顺序查找。

再散列（rehashing）：如果散列表满了，再往散列表中插入新的元素时候就会失败。这个时候可以通过创建另外一个散列表，使得新的散列表的长度是当前散列表的2倍多一些，重新计算各个元素的hash值，插入到新的散列表中。




### 4 总结
各种符号表实现的性能总结
<table align="center" border="1">
    <thead>
        <tr class="firstHead">
            <th colspan="1" rowspan="2">算法<br/>（数据结构）</th>
            <th colspan="2">最坏情况下的运行时间的增长数量级<br/>（N次插入之后）</th>
            <th colspan="2">平均情况下的运行时间的增长数量级<br/>（N次随机插入之后）</th>
            <th colspan="1" rowspan="2">关键接口</th>
            <th colspan="1" rowspan="2">内存使用</th>
            <th colspan="1" rowspan="2">是否支持有序性相关操作</th>
        </tr>
        <tr class="twoHead">
            <th>查找</th>
            <th>插入</th>
            <th>查找命中</th>
            <th>插入</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>顺序查找<br/>（无序链表）</td>
            <td>N</td>
            <td>N</td>
            <td>N/2</td>
            <td>N</td>
            <td>equals()</td>
            <td>48N</td>
            <td>否</td>
        </tr>
        <tr>
            <td>二分查找<br/>（有序数组）</td>
            <td>lgN</td>
            <td>N</td>
            <td>lgN</td>
            <td>N/2</td>
            <td>compareTo()</td>
            <td>16N</td>
            <td>是</td>
        </tr>
        <tr>
            <td>二叉树查找<br/>（二叉查找树）</td>
            <td>N</td>
            <td>N</td>
            <td>1.39lgN</td>
            <td>1.39lgN</td>
            <td>compareTo()</td>
            <td>64N</td>
            <td>是</td>
        </tr>
        <tr>
            <td>2-3查找树<br/>（红黑树）</td>
            <td>2lgN</td>
            <td>2lgN</td>
            <td>lgN</td>
            <td>lgN</td>
            <td>compareTo()</td>
            <td>64N</td>
            <td>是</td>
        </tr>
        <tr>
            <td>拉链法<br/>（链表数组）</td>
            <td>&lt;lgN</td>
            <td>&lt;lgN</td>
            <td>N/(2M)</td>
            <td>N/M</td>
            <td>equals()<br/>hashCode()</td>
            <td>48N+32M</td>
            <td>否</td>
        </tr>
        <tr>
            <td>现象探测法<br/>（并行数组）</td>
            <td>clgN</td>
            <td>clgN</td>
            <td>&lt;1.5</td>
            <td>&lt;2.5</td>
            <td>equals()<br/>hashCode()</td>
            <td>32N~128N</td>
            <td>否</td>
        </tr>
    </tbody>
</table>

### 5 并查集
并查集是一种树型的数据结构，用于处理一些不相交集合的合并及查询问题。常常在使用中以森林来表示。

**合并（Union / Merge）**：合并两个集合。
```java
    int Find(int x) {
        if(x == pre[x])
            return x;
        return pre[x] = Find(pre[x]);
    }
```

**查询（Find / Get）**：查询元素所属集合。
```java
    void merge(int x, int y) {
        int fx = Find(x), fy = Find(y);
        if(fx != fy)
            pre[fx] = fy;
    }
```

### 6 Trie（前缀树）
**Trie 树的结点结构**：Trie 树是一个有根的树，其结点具有以下字段：
+ 最多 R 个指向子结点的链接，其中每个链接对应字母表数据集中的一个字母。假定 R 为 26，小写拉丁字母的数量。
+ 布尔字段，以指定节点是对应键的结尾还是只是键前缀。

```java
class TrieNode {
    private TrieNode[] links;
    private final int R = 26;
    private boolean isEnd;
}
```

**插入**：从根开始搜索它对应于第一个键字符的链接。有两种情况：
- 链接存在。沿着链接移动到树的下一个子层。算法继续搜索下一个键字符。
- 链接不存在。创建一个新的节点，并将它与父节点的链接相连，该链接与当前的键字符相匹配。

重复以上步骤，直到到达键的最后一个字符，然后将当前节点标记为结束节点，算法完成。

```java
	public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char currentChar = word.charAt(i);
            if (!node.containsKey(currentChar)) {
                node.put(currentChar, new TrieNode());
            }
            node = node.get(currentChar);
        }
        node.setEnd();
    }
```

**查找单词和前缀匹配**：检查当前节点中与键字符对应的链接。有两种情况：
- 存在链接。我们移动到该链接后面路径中的下一个节点，并继续搜索下一个键字符。
- 不存在链接。若已无键字符，且当前结点标记为 isEnd，则返回 true。否则有两种可能，均返回 false :
  - 还有键字符剩余，但无法跟随 Trie 树的键路径，找不到键。
  - 没有键字符剩余，但当前结点没有标记为 isEnd。也就是说，待查找键只是Trie树中另一个键的前缀。

前缀匹配与在 Trie 树中搜索键时使用的方法非常相似，因为我们搜索的是键的前缀，而不是整个键。

```java
	private TrieNode searchPrefix(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
           char curLetter = word.charAt(i);
           if (node.containsKey(curLetter)) {
               node = node.get(curLetter);
           } else {
               return null;
           }
        }
        return node;
    }

    public boolean search(String word) {
       TrieNode node = searchPrefix(word);
       return node != null && node.isEnd();
    }

	public boolean startsWith(String prefix) {
        TrieNode node = searchPrefix(prefix);
        return node != null;
    }
```