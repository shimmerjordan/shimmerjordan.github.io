---
title: Add Two Numbers
date: 2020-12-08 20:42:10
tags: 
	- Algorithm
	- Data structures
categories: ['Algorithm']
id: addTwoNumbers
---
# What is  the best solution to Add Two Numbers

Currently I encounter a problem about adding two numbers, based on the bias between my initial solution and the official solution, I am able to generate a better way to add two numbers.

<!-- more -->

# Add Two Numbers

Detailed information can be found on [(LeetCode) Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

> You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.
>
> You may assume the two numbers do not contain any leading zero, except the number 0 itself.
>
> **Example 1:**
>
>  <center>
>      <img style="border-radius: 0.3125em;
>      box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
>      src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/09/r9yrAs.jpg" width='50%'>
>      <br>
>      <div style="color:orange; border-bottom: 1px solid #d9d9d9;
>      display: inline-block;
>      color: #999;
>      padding: 2px;">Example 1</div>
>  </center>
>
>
> > **Input:** 	l1 = [2, 4, 3],	l2 = [5, 6, 4]
> >
> > **Output:**	[7, 0, 8]
> >
> > **Explanation:**	342 + 465 = 807
>
> **Example 2:**
>
> > **Input:**	l1 = [0],	l2 = [0]
> >
> > **Output:**	[0]
>
> **Example 3:**
>
> > **Input:**	l1 = [9, 9, 9, 9, 9, 9, 9],	l2 = [9, 9, 9, 9]
> >
> > **Output:**	[8, 9, 9, 9, 0, 0, 0, 1]

Constraints:

- The number of nodes in each linked list is in the range `[1, 100]`
- `0 <= Node.val <= 9`
- It is guaranteed that the list represents a number that does not have leading zeros.

# Initial solution

First of all, this solution was my first attempt and failed in some test  cases after submission. 

We know that the digits are stored in reverse order in each linked list. So it is possible to transfer the linked list into integer first, and then store the sum digit by digit into a result linked list. The solution for this route is as follows:

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode curr = new ListNode(0);
        ListNode flag = curr;
        int digit = 1;
        int num1 = 0;
        int num2 = 0;
        
        while(l1 != null || l2 != null){
            long x = (l1 != null) ? l1.val : 0;
            long y = (l2 != null) ? l2.val : 0;
            
            num1 += x * digit;
            num2 += y * digit;
            
            if(l1 != null)
                l1 = l1.next;
            if(l2 != null)
                l2 = l2.next;
            digit *= 10;
        }
        
        long temp = num1 + num2;
        
        if(temp == 0){
             curr.next = new ListNode(0);
        }
        
        while(temp > 0){
            curr.next = new ListNode((int)temp % 10);
            curr = curr.next;
            temp /= 10;
        }
        return flag.next;
    }
}
```

It Seems that the solution have passed most of the test cases, but it **failed** for the case as follows:

> **Input:**	l1 = [9],	l2 = [1, 9, 9, 9, 9, 9, 9, 9, 9, 9]
>
> **Output:**	sum = [8, 0, 4, 5, 6, 0, 0, 1, 4, 1]
>
> **Expected:**	sum = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1]

It follows that the range of integer data types is not taken into account. Therefore, the out-of-bounds error is generated as a result that the wrong result is obtained at the sum calculation stage. Therefore, this approach is not feasible, and in fact only solves the problem of adding or subtracting a small number of small integers.

Then we need to sum directly from the original chain table and synchronize to the result chain table to avoid the data overflow problem.

# Correct Thinking

Create a new chain table by setting up a variable called carried, which represents a carryover. Process the two input links simultaneously from the beginning to the end, adding up every two, and then add the result plus the value of carried to the new table as a new node.

The process can be seen in the following diagram:

<center>
     <img style="border-radius: 0.3125em;
     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
     src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/09/rC6Vw4.gif" width="60%">
     <br>
     <div style="color:orange; border-bottom: 1px solid #d9d9d9;
     display: inline-block;
     color: #999;
     padding: 2px;">Adding process</div>
 </center>


 # Code Implementation

 ```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode(0);
        ListNode curr = dummyHead;
        int carried = 0;

        while(l1 != null || l2 != null || carried != 0){
            int l1Val = (l1 != null) ? l1.val : 0;
            int l2Val = (l2 != null) ? l2.val : 0;
            int sumVal = l1Val + l2Val + carried;
            carried = sumVal / 10;

            ListNode sumNode = new ListNode(sumVal % 10);
            curr.next = sumNode;
            curr = sumNode;

            if(l1 != null) l1 = l1.next;
            if(l2 != null) l2 = l2.next;
        }
        return dummyHead.next;
    }
}
 ```