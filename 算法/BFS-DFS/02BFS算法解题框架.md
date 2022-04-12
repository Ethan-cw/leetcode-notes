# BFS算法解题框架

|                            牛客网                            |                           LeetCode                           |                             力扣                             | 难度 |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :--: |
|                              -                               | [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree) | [111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree) |  🟢   |
| [打开转盘锁](https://www.nowcoder.com/practice/e7cbabbf7e0a41ec98055ee5f3d33bbe?tpId=295&fromPut=pc_wzcpa_labuladong_sf) | [752. Open the Lock](https://leetcode.com/problems/open-the-lock) | [752. 打开转盘锁](https://leetcode-cn.com/problems/open-the-lock) |  🟠   |

**其实 DFS 算法就是回溯算法**。

BFS 的核心思想应该不难理解的，就是把一些问题抽象成图，从一个点开始，向四周开始扩散。一般来说，我们写 BFS 算法都是用「队列」这种数据结构，每次将一个节点周围的所有节点加入队列。

BFS 相对 DFS 的最主要的区别是：**BFS 找到的路径一定是最短的，但代价就是空间复杂度可能比 DFS 大很多**，之后介绍了框架就很容易看出来了。

本文就由浅入深学习两道 BFS 的典型题目，分别是「二叉树的最小高度」和「打开密码锁的最少步数」，学习如何写BFS 算法。

## 1. 算法框架

**问题的本质就是让你在一幅「图」中找到从起点 `start` 到终点 `target` 的最近距离，BFS 算法问题其实都是在干这个事儿**。

这个广义的描述可以有各种变体：

* 比如走迷宫，有的格子是围墙不能走，从起点到终点的最短距离是多少？如果这个迷宫带「传送门」可以瞬间传送呢？

* 再比如说两个单词，要求你通过某些替换，把其中一个变成另一个，每次只能替换一个字符，最少要替换几次？

* 再比如说连连看游戏，两个方块消除的条件不仅仅是图案相同，还得保证两个方块之间的最短连线不能多于两个拐点。

这些问题本质上就是一幅「图」，让你从一个起点，走到终点，问最短路径。

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路
    
    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj()) {
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
            }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

队列 `q` 是BFS 的核心数据结构；`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回头路就不需要 `visited`。

## 2. 二叉树的最小高度

判断一棵二叉树的**最小**高度，这也是力扣第 111 题「 [二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree)」：

![img](img/title1.jpg)

怎么套到 BFS 的框架里呢？

* 首先明确一下起点 `start` 和终点 `target` 是什么。

**显然起点就是 `root` 根节点，终点就是最靠近根节点的那个「叶子节点」嘛**，叶子节点就是两个子节点都是 `null` 的节点：

```java
if (cur.left == null && cur.right == null) 
    // 到达叶子节点
```

完整java代码：

```java
int minDepth(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    // root 本身就是一层，depth 初始化为 1
    int depth = 1;
    
    while (!q.isEmpty()) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            TreeNode cur = q.poll();
            /* 判断是否到达终点 */
            if (cur.left == null && cur.right == null) 
                return depth;
            /* 将 cur 的相邻节点加入队列 */
            if (cur.left != null)
                q.offer(cur.left);
            if (cur.right != null) 
                q.offer(cur.right);
        }
        /* 这里增加步数 */
        depth++;
    }
    return depth;
}
```

这里注意这个 `while` 循环和 `for` 循环的配合，**`while` 循环控制一层一层往下走，`for` 循环利用 `sz` 变量控制从左到右遍历每一层二叉树节点**：

![img](img/1.jpeg)

> 读完并理解本文后可以去看看 BFS 算法是如何演变成 Dijkstra 算法在加权图中寻找最短路径的。[Dijkstra 算法模板框架](https://labuladong.github.io/algo/2/20/55/) 

## 3. 解开密码锁的最少次数

力扣第 752 题「 [打开转盘锁](https://leetcode-cn.com/problems/open-the-lock)」:

![img](img/title2.jpg)

**第一步，我们不管所有的限制条件，不管 `deadends` 和 `target` 的限制，就思考一个问题：如果让你设计一个算法，穷举所有可能的密码组合，你怎么做**？

穷举呗，再简单一点，如果你只转一下锁，有几种可能？总共有 4 个位置，每个位置可以向上转，也可以向下转，也就是有 8 种可能对吧。

比如说从 `"0000"` 开始，转一次，可以穷举出 `"1000", "9000", "0100", "0900"...` 共 8 种密码。然后，再以这 8 种密码作为基础，对每个密码再转一下，穷举出所有可能…

**仔细想想，这就可以抽象成一幅图，每个节点有 8 个相邻的节点**，又让你求最短距离，这不就是典型的 BFS 嘛，框架就可以派上用场了，先写出一个「简陋」的 BFS 框架代码再说别的：

```java
// 将 s[j] 向上拨动一次
String plusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '9')
        ch[j] = '0';
    else
        ch[j] += 1;
    return new String(ch);
}
// 将 s[i] 向下拨动一次
String minusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '0')
        ch[j] = '9';
    else
        ch[j] -= 1;
    return new String(ch);
}

// BFS 框架，打印出所有可能的密码
void BFS(String target) {
    Queue<String> q = new LinkedList<>();
    q.offer("0000");
    
    while (!q.isEmpty()) {
        int sz = q.size();
        /* 将当前队列中的所有节点向周围扩散 */
        for (int i = 0; i < sz; i++) {
            String cur = q.poll();
            /* 判断是否到达终点 */
            System.out.println(cur);

            /* 将一个节点的相邻节点加入队列 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                String down = minusOne(cur, j);
                q.offer(up);
                q.offer(down);
            }
        }
        /* 在这里增加步数 */
    }
    return;
}
```

**这段 BFS 代码已经能够穷举所有可能的密码组合了，但是显然不能完成题目，有如下问题需要解决**：

1. 会走回头路。比如说我们从 `"0000"` 拨到 `"1000"`，但是等从队列拿出 `"1000"` 时，还会拨出一个 `"0000"`，这样的话会产生死循环。

2. 没有终止条件，按照题目要求，我们找到 `target` 就应该结束并返回拨动的次数。

3. 没有对 `deadends` 的处理，按道理这些「死亡密码」是不能出现的，也就是说你遇到这些密码的时候需要跳过。

```java
class Solution {
    public int openLock(String[] deadends, String target) {
        Set<String> deads = new HashSet();
        for(String s: deadends) deads.add(s);

        Queue<String> q = new LinkedList();
        q.offer(new String("0000"));
        int step = 0;

        //bfs 框架
        while(!q.isEmpty()){
            int sz = q.size();
          	/* 将一个节点的未遍历相邻节点加入队列 */
            for(int i=0; i<sz; ++i){
                String cur = q.poll();
                
                if(deads.contains(cur)) continue;
              	/* 判断是否到达终点 */
                if(cur.equals(target)) return step;

                deads.add(cur);
                for(int j=0; j<4; ++j){
                    String up = plusOne(cur, j);
                    String down = minusOne(cur, j);
                    q.offer(up);
                    q.offer(down);
                }
            }
            ++step;
        }
        return -1;
    }

    // 在j这个位置上旋一下
    String plusOne(String s, int j){
        char[] ch = s.toCharArray();
        if(ch[j]=='9'){
            ch[j]='0';
        }else{
            ch[j] += 1;
        }
        return new String(ch);
    }

    // 在j这个位置下旋一下
    String minusOne(String s, int j){
        char[] ch = s.toCharArray();
        if(ch[j]=='0'){
            ch[j]='9';
        }else{
            ch[j] -= 1;
        }
        return new String(ch);
    }
}
```

## 4. 双向BFS优化

**传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。

![img](img/2.jpeg)

**不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。

* 比如二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；
* 但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的。

```java
int openLock(String[] deadends, String target) {
    Set<String> deads = new HashSet<>();
    for (String s : deadends) deads.add(s);
    // 用集合不用队列，可以快速判断元素是否存在
    Set<String> q1 = new HashSet<>();
    Set<String> q2 = new HashSet<>();
    Set<String> visited = new HashSet<>();
    
    int step = 0;
    q1.add("0000");
    q2.add(target);
    
    while (!q1.isEmpty() && !q2.isEmpty()) {
        // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
        Set<String> temp = new HashSet<>();

        /* 将 q1 中的所有节点向周围扩散 */
        for (String cur : q1) {
            /* 判断是否到达终点 */
            if (deads.contains(cur))
                continue;
            if (q2.contains(cur))
                return step;
            
            visited.add(cur);

            /* 将一个节点的未遍历相邻节点加入集合 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (!visited.contains(up))
                    temp.add(up);
                String down = minusOne(cur, j);
                if (!visited.contains(down))
                    temp.add(down);
            }
        }
        /* 在这里增加步数 */
        step++;
        // temp 相当于 q1
        // 这里交换 q1 q2，下一轮 while 就是扩散 q2
        q1 = q2;
        q2 = temp;
    }
    return -1;
}
```

双向 BFS 还是遵循 BFS 算法框架的，只是**不再使用队列，而是使用 HashSet 方便快速判断两个集合是否有交集**。

另外的一个技巧点就是 **while 循环的最后交换 `q1` 和 `q2` 的内容**，所以只要默认扩散 `q1` 就相当于轮流扩散 `q1` 和 `q2`。

**无论传统 BFS 还是双向 BFS，无论做不做优化，从 Big O 衡量标准来看，时间复杂度都是一样的**，只能说双向 BFS 是一种 trick，算法运行的速度会相对快一点，掌握不掌握其实都无所谓。最关键的是把 BFS 通用框架记下来，反正所有 BFS 算法都可以用它套出解法。