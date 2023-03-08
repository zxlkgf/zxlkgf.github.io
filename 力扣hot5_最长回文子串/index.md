# 力扣hot100_5_最长回文子串


# 最长回文子串
[最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/?favorite=2cktkvj)

题目:  
给你一个字符串 s，找到 s 中最长的回文子串。

如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。

示例1
```text
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案
```

## 题解和思路
//中心扩散法  
从字符串的每一个index开始操作;  
如果往左边字符串相同，则长度+1 直到没有重复  
如果往右边字符串相同，则长度+1 直到没有重复  
然后开始 向两边扩张，left的字母等于right 的字母，则长度+2;
进行一轮判断之后，如果长度比maxlen大，将left赋值给maxleft  right赋值给maxright    
重复  
```c++
class Solution {
public:
    string longestPalindrome(string s) {
        //使用中心扩散方法
        int maxleft = 0;
        int maxright = 0;
        int maxlen = 0;
        int len = 1 ;

        for(int mid = 0; mid < s.size() ;mid++){
            int left = mid - 1; //重复字符串左边界
            int right = mid + 1;    //重复字符串右边界限

            while( left >= 0 && s[left] == s[mid]){
                left --;
                len++;
            }
            while(right <= s.size()-1&&s[right]==s[mid]){
                right++;
                len++;
            }

            while(left >=0 && right <=s.size()-1 && s[left] == s[right]){
                left -- ;
                right ++ ;
                len +=2;
            }

            if(len > maxlen){
                maxleft = left;
                maxright = right;
                maxlen = len;
            }
            len = 1;
        }
        return s.substr(maxleft + 1,maxlen);
    }
};
```
