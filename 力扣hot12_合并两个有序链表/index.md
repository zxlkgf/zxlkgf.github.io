# 力扣hot100_12_合并两个有序链表


# 合并两个有序链表
[合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的

示例1
```text
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

## 题解和思路
循环遍历两个链表，对比大小，小的插入到新链表 直到某个链表为空    

继续循环不为空的链表插入到新链表

返回新链表

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
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode* head = new ListNode(0);
        ListNode* tail = head;

        while(list1&&list2){
            int v1 = list1->val;
            int v2 = list2->val;
            if(v1>v2){
                tail->next=new ListNode(v2);
                tail = tail->next;
                list2 = list2->next;
            }else{
                tail->next=new ListNode(v1);
                tail = tail->next;
                list1 = list1->next;
            }
            
        }
        while(list1){
            tail->next=new ListNode(list1->val);
            tail = tail->next;
            list1 = list1->next;
        }
        while(list2){
            tail->next=new ListNode(list2->val);
            tail = tail->next;
            list2 = list2->next;
        }
        return head->next;
    }
};
```
