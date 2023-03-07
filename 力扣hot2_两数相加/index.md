# 力扣hot100_2_两数相加


# 两数相加
[两数相加](https://leetcode.cn/problems/add-two-numbers/?favorite=2cktkvj)

题目:  
给你两个非空的链表，表示两个非负的整数。它们每位数字都是按照逆序的方式存储的，并且每个节点只能存储 一位数字。
请你将两个数相加，并以相同形式返回一个表示和的链表


示例1
```text
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

## 题解和思路
创建一个ListNode *head作为链表头  
创建一个ListNode *tail作为链表尾部.  
使用||符号，不去判断每个链表的节点是否为空，直到两个链表都到达链表最后。  
创建新链表内的值 为两个链表内值相加 / 10  
循环到最后的时候判断一下carry是否>0

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *head = new ListNode(0);
        ListNode *tail = head;
        int sum = 0;
        int carry = 0;
        while(l1 || l2){
            sum += (l1!=NULL?l1->val:0);
            sum += (l2!=NULL?l2->val:0);
            sum += carry;
            tail->next = new ListNode( sum % 10);
            tail = tail->next; 
            carry = sum / 10;
            sum = 0;
            l1 = l1->next;
            l2 = l2->next;
        }
        if(carry>0){
            tail->next = new ListNode( carry );
            tail = tail->next ;
        }
        return head->next;
    }
};

```
