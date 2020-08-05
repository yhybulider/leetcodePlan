# leetcode39-组合总和I

这个题一开始拿到的时候，是想往三数之和那种去靠

![image-20200805220151028](https://gitee.com/yhycoder/photo/raw/master/image-20200805220151028.png)

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
// 使用回溯的方法，回溯完之后要去掉当前的元素，遍历下一个元素
    List<List<Integer>> list = new ArrayList<>();
    // 先数组排序
    Arrays.sort(candidates);
    backtracking(0,candidates,target,new ArrayList<>(),list);
    return list;

    }

    public void backtracking(int start,int[] nums,int remain,List<Integer> templist, List<List<Integer>> list){
        if(remain < 0){
            return;
        }else if(remain == 0){
            list.add(new ArrayList<>(templist));
        }else{
            // 添加参数start，将去掉重复的结果
            for(int i = start; i < nums.length;i++){
                templist.add(nums[i]);
                backtracking(i,nums,remain-nums[i],templist,list);
                // 减掉访问过的枝叶，进行回溯
                templist.remove(templist.size() - 1);
            }
        }

        

    }
}
```

