# 力扣hot100_9_电话号码的字母组合


# 电话号码的字母组合
[电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

题目:  
给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。  


""," ","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"
示例1
```text
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

```

## 题解和思路
定义一个全局number变量去存储电话号码按键。  

使用回溯法去回溯。  


```c++
class Solution {
public:
    vector<string> number = {""," ","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};

    void dfs(vector<string> &ans,const string &digits,int pos,string& temp){
        if(pos==digits.size()){
            ans.push_back(temp);
            return;
        }
        int num = digits[pos] - '0';
        for(int i=0;i<number[num].size();i++){
            char ch = number[num][i];
            temp.push_back(ch);
            dfs(ans,digits,pos+1,temp);
            temp.pop_back();
        }
    }

    vector<string> letterCombinations(string digits) {
        vector<string> ans;
        string temp;
        if(digits.size()<1)return ans;
        dfs(ans,digits,0,temp);
        return ans;
    }
};
```
