# BFS 算法秒杀各种智力题

| 牛客网 |                           LeetCode                           |                             力扣                             | 难度 |
| :----: | :----------------------------------------------------------: | :----------------------------------------------------------: | :--: |
|   -    | [773. Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle) | [773. 滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle) |  🔴   |

滑动拼图游戏大家应该都玩过，下图是一个 4x4 的滑动拼图：

![img](img/1-20220412231905888.jpeg)

拼图中有一个格子是空的，可以利用这个空着的格子移动其他数字。你需要通过移动这些数字，得到某个特定排列顺序，这样就算赢了。

还有「华容道」的益智游戏，也和滑动拼图比较类似：

![img](img/2-20220412231934716.jpeg)

**这些益智游戏通通可以用暴力搜索算法解决，所以今天我们就学以致用，用 BFS 算法框架来秒杀这些游戏**。

## 1. 题目解析

力扣第 773 题「 [滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle)」就是这个问题，题目的要求如下：

给你一个 2x3 的滑动拼图，用一个 2x3 的数组 `board` 表示。拼图中有数字 0~5 六个数，其中**数字 0 就表示那个空着的格子**，你可以移动其中的数字，当 `board` 变为 `[[1,2,3],[4,5,0]]` 时，赢得游戏。

请你写一个算法，计算赢得游戏需要的最少移动次数，如果不能赢得游戏，返回 -1。

比如说输入的二维数组 `board = [[4,1,2],[5,0,3]]`，算法应该返回 5：

![img](img/5.jpeg)

## 2. 思路分析

这种计算最小步数的问题，我们就要敏感地想到 BFS 算法。

这个题目转化成 BFS 问题是有一些技巧的，我们面临如下问题：

1. 一般的 BFS 算法，是从一个起点 `start` 开始，向终点 `target` 进行寻路，但是拼图问题不是在寻路，而是在不断交换数字，这应该怎么转化成 BFS 算法问题呢？

2. 即便这个问题能够转化成 BFS 问题，如何处理起点 `start` 和终点 `target`？它们都是数组哎，把数组放进队列，套 BFS 框架，想想就比较麻烦且低效。

本质上**BFS 算法并不只是一个寻路算法，而是一种暴力搜索算法**，只要涉及暴力穷举的问题，BFS 就可以用，而且可以最快地找到答案。

本质上就是把所有可行解暴力穷举出来，然后从中找到一个最优解罢了。

这样我们的问题就转化成了：**如何穷举出 `board` 当前局面下可能衍生出的所有局面**？这就简单了，看数字 0 的位置呗，和上下左右的数字进行交换就行了：

![img](img/3.jpeg)

这样其实就是一个 BFS 问题，每次先找到数字 0，然后和周围的数字进行交换，形成新的局面加入队列…… 当第一次到达 `target` 时，就得到了赢得游戏的最少步数。

对于第二个问题，我们这里的 `board` 仅仅是 2x3 的二维数组，所以可以压缩成一个一维字符串。**其中比较有技巧性的点在于，二维数组有「上下左右」的概念，压缩成一维后，如何得到某一个索引上下左右的索引**？

很简单，我们只要手动写出来这个映射就行了：

```java
// 记录一维字符串的相邻索引
int[][] neighbor = new int[][]{
        {1, 3},
        {0, 4, 2},
        {1, 5},
        {0, 4},
        {3, 1, 5},
        {4, 2}
};
```

**这个含义就是，在一维字符串中，索引 `i` 在二维数组中的的相邻索引为 `neighbor[i]`**：

![img](img/4.jpeg)

至此，我们就把这个问题完全转化成标准的 BFS 问题了，借助前文 BFS 算法框架 的代码框架，直接就可以套出解法代码了：

```java
class Solution {
    public int slidingPuzzle(int[][] board) {
        StringBuilder sb = new StringBuilder();
        for(int i=0; i<2; ++i){
            for(int j=0; j<3; ++j){
                sb.append(board[i][j]);
            }
        }
        String start = sb.toString();

        String target="123450";
        
        // 记录每一个位置的邻居
        int[][] nei = {
            {1, 3},
            {0, 2, 4},
            {1, 5},
            {0, 4},
            {1, 3, 5},
            {2, 4}
        };
        
        /******* BFS 算法框架开始 *******/
        Queue<String> q = new LinkedList<>();
        HashSet<String> visited = new HashSet<>();
        int step = 0;
        // 从起点开始 BFS 搜索
        q.offer(start);
        while(!q.isEmpty()){
            int sz = q.size();
            for(int i=0; i<sz; ++i){
                String cur = q.poll();
                if(cur.equals(target)) return step;
                //加入visited 防止回头遍历
                visited.add(cur);
                //找到0的idx
                int idx = cur.indexOf('0');
                for(int j: nei[idx]){
                    char[] ch = cur.toCharArray();
                    String next = getNext(ch, idx, j);
                    if(visited.contains(next)) continue;
                    q.offer(next);
                }
            }
            ++step;
        }
        return -1;
    }

    String getNext(char[] ch, int idx, int j){
        char temp = ch[idx];
        ch[idx] = ch[j];
        ch[j] = temp;
        return new String(ch);        
    }
}
```

