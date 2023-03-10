# 力扣hot100_13_括号生成


# 括号生成
[括号生成](https://leetcode.cn/problems/generate-parentheses/?favorite=2cktkvj)

数字 n 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。

示例1
```text
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

## 题解和思路
使用回溯算法  

可以按照左右括号的匹配情况判断  

优先使用左括号，如果左括号小于右括号，则使用右括号  


```c++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        if(n<=0){
            return ans;
        }
        dfs("",n,n);
        return ans;
    }
private:
    vector<string> ans;
    void dfs(const string &str,int left,int right){
        if(left==0&&right==0){
            ans.push_back(str);
            return;
        }
        //优先使用左括号
        if(left>0){
            dfs(str+'(',left-1,right);
        }
        if(left<right){
            dfs(str+')',left,right-1);
        }
    }
};

```
