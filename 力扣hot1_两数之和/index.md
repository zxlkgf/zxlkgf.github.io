# 力扣hot100_1_两数之和


# 两数之和
[两数之和](https://leetcode.cn/problems/two-sum/?favorite=2cktkvj)


题目：  
给定一个整数数组nums和一个整数目标值target，请你在该数组中找出和为目标值 target 的那两个整数,并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

示例1
```text
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。

```


## 思路和题解
创建一个ans的vector容器  
创建一个map的哈希表  
遍历nums内的元素，并查找哈希表内target - nums[i]的值是否在哈希表内，不在则将键值对为key:nums[i],value:i 加入到哈希表中，如果target - nums[i]的值在哈希表内，则将当前值的index和目标值的index加入到ans中，并返回即可。  

单次hash遍历的时间复杂度为O(n).


```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> ans;
        map<int,int> map;
        ans.push_back(-1);
        ans.push_back(-1);

        for(int i = 0;i < nums.size();i++){
            if(map.count(target - nums[i])>0){
                ans[0] = (map[target - nums[i]]);
                ans[1] =  i;
                return ans;
            }
            map[nums[i]] = i;
        }
        return ans;
    }
};

```

