# 力扣hot100_8_三数之和


# 三数之和
[三数之和](https://leetcode.cn/problems/3sum/?favorite=2cktkvj)

题目:  
给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请

你返回所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例1
```text
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。

```

## 题解和思路
三个数之和，可以先固定一个数，再在内层使用双指针从除i外的两端开始向内查找，
并且，需要每次都对三个数进行查重。

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        //创建答案数组
         vector<vector<int>> ans;
        //判断长度
        if(nums.size()<3||nums.empty()) return ans;
        //排序
        sort(nums.begin(),nums.end());
        //从头循环
        int i = 0 ;
        while(i<nums.size()){
            if(nums[i]>0)break;//提前终止
            int left = i+1;
            int right = nums.size()-1;
            while(left<right){
                int x = nums[i];
                int y = nums[left];
                int z = nums[right];
                if(x + y >0 - z){
                    right-- ;
                }else if(x + y < 0 - z){
                    left ++;
                }else{
                    ans.push_back({nums[i],nums[left],nums[right]});
                    //不允许重复
                    while(left<right&&nums[left] == nums[left+1]){
                        left++;
                    }
                    while(left<right&&nums[right] == nums[right-1]){
                        right--;
                    }
                    left++;
                    right--;
                }
            }
            //避免nums[i]重复
            while(i+1<nums.size()&&nums[i]==nums[i+1]){
                i++;
            }
            i++;
        }
        return ans;
    }
};
```
