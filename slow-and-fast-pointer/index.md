# 快慢指针

> ### 什么是快慢指针  
> &emsp;&emsp;在链表中使用两个指针，其中一个指针的移动速度比另外一个指针移动的速度快，这就是快慢指针。借助两个指针产生的距离差值，快慢指针有妙用。 

> ### 链表结构  
```java
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { 
        this.val = val; 
    }
    ListNode(int val, ListNode next) { 
        this.val = val; 
        this.next = next; 
    }
}
```
> ### 快慢指针的妙用
#### 找n等分点  
&emsp;&emsp;n = 2是最常见的情况(即寻找链表的中点), 这是解决许多问题的第一步(如: [No.143](https://leetcode-cn.com/problems/reorder-list/)、[No.148](https://leetcode-cn.com/problems/sort-list/))。我常用的寻找2等分点代码如下(*java*):  
```java
ListNode slow = head, fast = head;
while(fast.next != tail && fast.next.next != tail){
    slow = slow.next;
    fast = fast.next.next;
}
```
&emsp;&emsp;在上述代码执行完成后, slow指针指向了链表的中间结点或中间两个结点的左结点。  
![奇数结点](SF_O.png)  
![偶数结点](SF_E.png)
#### 判环  
&emsp;&emsp;快慢指针的另一个重要用法就是判断链表中是否有环路(如: [No.141](https://leetcode-cn.com/problems/linked-list-cycle/))  
```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null)
            return false;
        ListNode slow = head, fast = head.next;
        while(fast != null){
            if(slow == fast)
                return true;
            else{
                slow = slow.next;
                fast = fast.next;
                if(fast != null)
                    fast = fast.next;
            }
        }
        return false;
    }
}
```
&emsp;&emsp;进一步的, 利用快慢指针还可以找到链表开始入环的第一个节点([N0.142](https://leetcode-cn.com/problems/linked-list-cycle-ii/))。
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if(head == null)
            return null;
        ListNode slow = head, fast = head;
        while(fast != null){
            slow = slow.next;
            if(fast.next != null){
                fast = fast.next.next;
            }
            else return null;
            if(slow == fast){
                ListNode ptr = head;
                while(ptr != slow){
                    ptr = ptr.next;
                    slow = slow.next;
                }
                return slow;
            }
        }
        return null;
    }
}
```
&emsp;&emsp;至于为什么这样可以找到入环的第一个结点, 这里不再详述。  
提醒两个关键点:
* **fast走过的路程是slow走过路程的两倍**  
* **在slow入环的第一圈两个指针相遇**
