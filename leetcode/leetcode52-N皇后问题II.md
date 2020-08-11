# leeetcode52-N皇后

## 题目



## 解答![image-20200808003531783](https://gitee.com/yhycoder/photo/raw/master/image-20200808003531783.png)![image-20200808003532834](https://gitee.com/yhycoder/photo/raw/master/image-20200808003532834.png)

这个题目相对于51题可能更好解答一点，因为这里不需要输出字符，只需将次数计算

```java
class Solution {
    private int n;
    private int count;
    private List<String> inres;
    public int totalNQueens(int n) {
        // 八皇后问题
        // 使用回溯算法来做
        // 根据输入来做
        // 初始化
        this.n = n;
		this.count = 0;
     
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
        return count;

    }
    // 回溯的主要方法 参数是从哪一个开始，从0个开始
    public void backtracking(int row){
        // 套路中先写结束条件，这里是N皇后问题，对应的是将N对应row
        if(row == n){
            count++;
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
        // 判断斜线上，可以用绝对值来判断 ,这个是左斜线上
        for(int i = row-1,j = column-1; i >= 0 && j >= 0; --i,--j){
            if(inres.get(i).charAt(j) == 'Q'){
                return false;
            }
        }
        // 判断右斜线上是否有冲突，这里的两次判断都是在前面去判断，没有去理后面的结果
        // 
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

```

这里采用皇后问题一的代码也可以解决，但是相对来说是繁琐的，因为这里不需要输出字符串，也就是不需要输出结果，我们可以直接将计数输出就可以了，相对会快一点。分析如下

![image.png](https://gitee.com/yhycoder/photo/raw/master/d83efa81a42afdbd267b8a4883955660d4f38922828f355a98f40629aae7a23c-image.png)