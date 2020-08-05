# leetcode51-N皇后

这是经典的N皇后问题，经典的使用到了回溯法的思想。

1. 是什么
Backtracking:暴力搜索的一种，采用试错的思想，尝试分布得去解决一个问题，在分布解决问题的过程中，当它尝试发现现有的分步答案不能得到有效的正确的解答的时候，它将取消上一步或者上几万步的计算，再通过他的分步解答尝试再次寻找答案的答案。
回溯法可以看成是递归调用的一种特殊形式；简单来说也是一种DFS；从一条路往前走，能进则进，不进则退，换一条再试。
执行
保存当前步骤，如果是一个解就输出；维护状态，使搜索路径（含子路径）尽量不重复。必要时，应该对不可能为解的部分进行剪枝。--这一点常常用到


2. 怎么用
有套路
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择



这里最重要的是这个for循环中的递归怎么实现。

3. Leetcode
	a. 八皇后问题

## 题目

![image-20200805220440904](https://gitee.com/yhycoder/photo/raw/master/image-20200805220440904.png)

```java
class Solution {
    private int n;
    private List<List<String>> res;
    private List<String> inres;
    public List<List<String>> solveNQueens(int n) {
        // 八皇后问题
        // 使用回溯算法来做
        // 根据输入来做
        // 初始化
        this.n = n;
        res = new ArrayList<>();
        inres = new ArrayList<>();
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < n; ++i){
            sb.append('.');
        }
        for(int i = 0; i < n; ++i){
            // 每一行都有.....
            inres.add(sb.toString());
        }
        backtracking(0);
        // 返回结果
        return res;

    }
    // 回溯的主要方法 参数是从哪一个开始，从0个开始
    public void backtracking(int row){
        // 套路中先写结束条件，这里是N皇后问题，对应的是将N对应row
        if(row == n){
            res.add(new ArrayList<>(inres));
            return;
        }

        // 开始选择
        // 遍历row中的每一列
        for(int column = 0; column < n; ++column){
            // 判断皇后放置问题是否合法
            if(!isValid(row,column)) continue;//无效就跳过
            // 做选择，进行放置皇后
            setQueen(row,column,'Q');
            // 回溯下一个
            backtracking(row+1);
            // 撤销选择，回退回去
            setQueen(row,column,'.');
        }
    }

    private boolean isValid(int row,int column){
        // 判断同列
        for(int i = 0; i < row; ++i){
            if(inres.get(i).charAt(column) == 'Q'){
                return false;
            }
        }
        // 判断左斜线上，可以用绝对值来判断 ,这个是左斜线上
        for(int i = row-1,j = column-1; i >= 0 && j >= 0; --i,--j){
            if(inres.get(i).charAt(j) == 'Q'){
                return false;
            }
        }
        // 判断右斜线上是否有冲突，这里的两次判断都是在前面去判断，没有去理后面的结果
        // 注意这里的j填写等
         for(int i = row-1,j = column+1; i >= 0 && j<n;--i,++j){
            if(inres.get(i).charAt(j) == 'Q'){
                   return false;
            }  
        }
        
        // 全部符合条件后就开始返回的是true
        return true;
    }


    private void setQueen(int row,int column,char c){
        // 获得某行去进行修改
        StringBuilder sb = new StringBuilder(inres.get(row));
        // 给指定位置设置为某字符
        sb.setCharAt(column,c);
    // 将某行字符更新，设置好对应的字符。
        inres.set(row,sb.toString());
    }
}

作者：fire-20
链接：https://leetcode-cn.com/problems/n-queens/solution/huang-hou-wen-ti-by-fire-20/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

