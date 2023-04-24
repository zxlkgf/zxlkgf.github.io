# 力扣hot100_19_组合总和


# 组合总和
[组合总和](https://leetcode.cn/problems/combination-sum/?favorite=2cktkvj)

给你一个 无重复元素 的整数数组 candidates 和一个目标整数 target ，找出 candidates 中可以使数字和为目标数 target 的 所有 不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。

candidates 中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 target 的不同组合数少于 150 个。



示例1
```text
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]
解释：
2 和 3 可以形成一组候选，2 + 2 + 3 = 7 。注意 2 可以使用多次。
7 也是一个候选， 7 = 7 。
仅有这两种组合。
```

## 题解和思路
注意:数字可被重复选，但是数字的个数的数量需要不同。

对数组先进性排序，其目的是如果target - candidates[i] < 0 则其后面的都小于0，可以直接break 

可以使用回溯算法，从数组的0-len开始进行回溯  

回溯算法内部可以是顺序遍历数组的begin 和 end 位置，判断target - candidates[i]是否小于0，如果小于零 则break

如果等于0 则将其加入ans

并将target - candidates[i]继续传入dfs函数内部，进行进一步判断。

```c++
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        int len = candidates.size();
        vector<vector<int>> ans;
        if(len == 0)return ans;
        sort(candidates.begin(),candidates.end());
        vector<int> path;
        dfs(candidates,0,len,target,path,ans);
        return ans;

    }
    void dfs(vector<int> &candidates,int begin,int end,int target,vector<int>&path,vector<vector<int>> &ans){
        if(target == 0){
            ans.push_back(path);
            return;
        }
        for(int i = begin;i < end;i ++){
            if(target-candidates[i]<0)break;
            path.push_back(candidates[i]);
            dfs(candidates,i,end,target-candidates[i],path,ans);
            path.pop_back();
        }
    }
};

```
