# 力扣hot100_18_在数组中查找元素的第一个和最后一个位置


# 在数组中查找元素的第一个和最后一个位置
[在数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/?favorite=2cktkvj)

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。


示例1
```text
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]

```

## 题解和思路
使用两个二分  

第一个二分将r往target处移动，如果有target，则l和r会逐渐重合，找出第一个target的下标  

第二个二分将l往最后的target处移动，如果有target，则l和r会逐渐重合，找出最后一个target的下标   


  
```c++
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> ans = {-1,-1};
        int len = nums.size();
        if(len == 0)return ans;
        int l = 0;
        int r = len - 1;
        while(l<r){
            int mid = ( l + r ) >> 1;
            if(nums[mid]>=target){
                r = mid;
            }else{
                l = mid + 1;
            }
        }
        if(nums[l]!=target)return ans;
        ans[0] = l;
        l = 0;
        r = len - 1;
        while(l<r){
            int mid = (l + r + 1) >> 1;
            if(nums[mid]<=target){
                l = mid;
            }else{
                r = mid - 1;
            }
        }
        ans[1] = l;
        return ans;
    }
};

```
