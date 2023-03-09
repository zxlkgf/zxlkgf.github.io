# 力扣hot100_10_删除链表倒数第N个节点


# 删除链表倒数第N个节点
[删除链表倒数第N个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

题目:  
给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。
示例1
```text
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]

```

## 题解和思路
使用快慢指针   

定义一个fast指向head  

定义一个新的ListNode节点dummy，并将其next指向head，让slow指针指向dummy  

对fast进行遍历 遍历次数为n(倒数节点的值);  

然后我们对slow和fast都进行遍历，直到fast节点指向null。这样slow就指向了倒数第n个节点。  

那么如果我们让slow指向dummy，再进行遍历，就可以得到需要删除的节点的前置节点，就便于删除  

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        //快慢指针
        ListNode *dummy = new ListNode(0,head);
        ListNode *fast = head;
        ListNode *slow = dummy;

        for(int i = 0;i<n;i++){
            fast = fast->next;
        }
        while(fast){
            fast=fast->next;
            slow=slow->next;
        }
        slow->next = slow->next->next;
        return dummy->next;
    }
};
```
