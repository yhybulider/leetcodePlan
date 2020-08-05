# leetcode40-组合总和II

这道题和39可以一起放置来做，题目如下

​	![image-20200805215809692](https://gitee.com/yhycoder/photo/raw/master/image-20200805215809692.png)

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        // 这里要实现一个组合总和里面的解不能有重复的组合
        // 做个标记，每次如果上一次都使用过了 就不再考虑使用
        // 最重要的是先排序
        Arrays.sort(candidates);
        List<List<Integer>> res = new ArrayList<>();
        // 单次结果
        List<Integer> path = new ArrayList();
        dfs(candidates,res,path,target,0);
        return res;
    }
    // 这里num是用来记录循环的步数到哪了，就是遍历走过的地方
    private void dfs(int[] nums,List<List<Integer>> res,  List<Integer> path, int target,int num){
        if(target == 0){
            res.add(new ArrayList<>(path));
            return;
        }
        int pre = -1;
        for(int i = num; i < nums.length; i++){
            // 将重复项给跳过
            if(pre == nums[i]){
                continue;
            }
            if(nums[i] > target){
                return;
            }//剪枝叶操作
            pre = nums[i];
            path.add(nums[i]);
            // i+1表示遍历下一个
            dfs(nums,res,path,target-nums[i],i+1);
            // 删掉最后一个，就是刚刚添加进去的
            path.remove(path.size()-1);

        }
    }
}
```

