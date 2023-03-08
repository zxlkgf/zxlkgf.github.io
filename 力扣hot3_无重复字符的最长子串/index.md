# 力扣hot100_3_无重复字符的最长子串


# 无重复字符的最长子串
[无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/?favorite=2cktkvj)

题目:  
给定一个字符串 s ，请你找出其中不含有重复字符的最长子串的长度。


示例1
```text
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

## 题解和思路
创建一个桶，其内的元素为128个，并将其置为0;  
创建一个最大长度值maxvalue = 0;  
创建一个指向最左边的变量head = 0;  
循环访问字符串s内的每一个元素;  
head代表不重复字符串最左边的字符的下标，如果重复出现了a，则当前head的值为上一个v[a]内的值，而上一个v[a]内的值是上一个a的下标加1，这样可以让窗口往前移动。  
最大值为已经计算的最大值，和 当前不重复字符串的最大值的对比。  


```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        //判断是否为0
        if(s.size()==0)return 0;   
        // 滑动窗口
        vector<int> v(128,0);
        int head = 0;
        int maxvalue = 0;
        for(int i =0;i<s.size();i++){
            head = max(head,v[s[i]]);
            v[s[i]] = i + 1 ;
            maxvalue = max(maxvalue , i - head + 1);
        }
        return maxvalue;
    }
};
```
