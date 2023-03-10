# 力扣hot100_11_有效括号


# 有效括号
[有效括号](https://leetcode.cn/problems/valid-parentheses/)
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
每个右括号都有一个对应的相同类型的左括号。
示例1
```text
输入：s = "()[]{}"
输出：true

```

## 题解和思路
使用栈  

将括号的组合放入哈希表中  

依次判断即可

```c++
class Solution {
public:
    map<char,char> map = {
            {')', '('},
            {']', '['},
            {'}', '{'}
    };

    bool isValid(string s1) {
        if(s1.size()%2 == 1)return false;
        stack<char> z;
        for(char ch : s1){
            if(map.count(ch)){
                if(z.empty()||z.top()!=map[ch]){
                    return false;
                }
                z.pop();
            }else{
                z.push(ch);
            }
        }
        return z.empty();
    }
};
```
