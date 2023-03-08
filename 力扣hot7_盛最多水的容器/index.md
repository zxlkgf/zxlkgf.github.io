# 力扣hot100_7_盛最多水的容器


# 盛最多水的容器
[盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/?favorite=2cktkvj)

题目:  
给定一个长度为 n 的整数数组 height 。有 n 条垂线，第 i 条线的两个端点是 (i, 0) 和 (i, height[i]) 。

找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

说明：你不能倾斜容器

示例1
```text
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

## 题解和思路
双指针  
双向向中间遍历  
判断哪边最短，选取最短的边的计算结果  
最短边向中间移动一格  
返回结果  

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;
        int right = height.size()-1;
        int ans = 0;
        while(left < right){
            if(height[right]>height[left]){
                int area = height[left] *(right - left );
                ans = max(ans,area);
                left++;
            }else{
                int area = height[right] *(right - left );
                ans = max(ans,area);
                right--;
            }
            
        }
        return ans;
    }
};
```
