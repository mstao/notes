[TOC]

# 链表问题 

链表的操作：

循转链表  
链表的反转，按照组进行反转，从指定位置进行反转  

链表对数据进行排序  

两个链表的处理  
有关递归的处理   

节点的更改操作，删除指定的节点，对节点进行排序，反转，交换  



常用链表的辅助操作  
找出链表中第 n 个节点、找出链表的倒数 nth 节点
获取链表长度  





常用思路:  
快慢指针  
递归  
虚拟头结点  
两条链表、单条链表拆分成两条链表  -- 考虑链表到底, 以及一条为 null, 一条不为 null
头插法逆序  



链表的添加 | 删除 | 交换 | 反转 | 查找 | 交换： 
链表元素的删除:  
删除链表中所有给定值的节点；  
删除链表中倒数的 kth 节点；  
删除排序的链表中重复元素节点      
删除排序的链表中重复元素节点并保留第一次出现的节点(去重)；     
删除链表中给定的节点   
在 O(1) 内删除给定的节点  



链表的判断和查找:   
判断链表是否有环；   
判断链表是否是回文链表；    
找到链表的倒数 kth 节点；   
找到链表的中间节点    
找到两条链表的交点；   
找出链表环的入口节点；  




链表的修改和构建:   
反转单链表；    
反转链表第 m,n 之间的节点；  
将链表旋转 k 位；  
交换相邻的两个节点形成新的链表；      
重建链表: 将链表按奇偶位置形成两条链表并拼接   
两条链表中的值相加     
反向存储的链表值相加；    

左右分割: 将链表分割成左半部分小于 x, 右半部分小于 大于x    
多条分割: 将一条链表分割成多条链表，有余时前面的个数大于后面的一个  

 


排序: 对单链表进行排序；   
链表的插入排序；   
按照指定规则排序原始链表；  
合并两个排序的链表    
合并 k 个已经排序的链表；    


根据有序链表构建平衡的二叉查找树；   
复杂链表的复制；  



链表的操作:

分割、合并、排序



复制、构建



## 性质


**删除倒数第 n 个节点**

[19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)

```
Given linked list: 1->2->3->4->5, and n = 2.
1->2->3->5.
```

思路一： 定位到第 n 个节点，同时记录 pre，用于删除

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode fast = head;
    while (n -- > 0)      // make fast is nth
        fast = fast.next;
    ListNode first = new ListNode(-1);
    first.next = head;
    ListNode pre = first;
    while (fast != null) {         // find nthFromEnd previous node
        pre = pre.next;
        fast = fast.next;
    }
    pre.next = pre.next.next;
    return first.next;
}
```

思路二： 通过 fast.next != null 进行两个指针的控制



**交换链表中的相邻结点**

[24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/description/)

```
Given 1->2->3->4, you should return the list as 2->1->4->3.
```

思路一： 确定之前节点，当前节点，当前节点的下一个节点，以及下一次的next 节点

```java
public ListNode swapPairs(ListNode head) {
    ListNode first = new ListNode(-1);
    first.next = head;
    ListNode pre = first;
    while (pre.next != null && pre.next.next != null) {    // note : must first judge pre.next != null
        ListNode n1 = pre.next, n2 = n1.next, next = n2.next;
        n1.next = next;
        n2.next = n1;
        pre.next = n2;
        pre = n1;
    }
    return first.next;
}
```

思路二： 递归控制

```java
public ListNode swapPairs(ListNode head) {
    if (head == null || head.next == null)
        return head;
    ListNode n = head.next;
    head.next = swapPairs(n.next);
    n.next = head;
    return n;
}
```



**按照组进行反转**



**将链表旋转给定 k 位**

```
Input: 1->2->3->4->5->NULL, k = 2
Output: 4->5->1->2->3->NULL
Explanation:
rotate 1 steps to the right: 5->1->2->3->4->NULL
rotate 2 steps to the right: 4->5->1->2->3->NULL
Example 2:

Input: 0->1->2->NULL, k = 4
Output: 2->0->1->NULL
Explanation:
rotate 1 steps to the right: 2->0->1->NULL
rotate 2 steps to the right: 1->2->0->NULL
rotate 3 steps to the right: 0->1->2->NULL
rotate 4 steps to the right: 2->0->1->NULL
```

思路一： 类似数组的处理方式，计算出链表总长度，通过一次遍历找到尾节点以及倒数第 n 个节点的前一个节点，之后重新拆分拼接实现；

```java
public ListNode rotateRight(ListNode head, int k) {
    if (head == null || head.next == null)
        return head;
    int len = length(head);
    k = k % len;                        // preHandle
    if (k == 0)
        return head;

    ListNode pre = null;
    ListNode cur = head;
    int cnt = 0;
    while (cur.next != null) {    // find pre node nthNodeFromEnd AND tail node
        cnt ++;
        if (cnt == len - k)       // count to calculate pre
            pre = cur;
        cur = cur.next;
    }
    ListNode newHead = pre.next;        // as head
    cur.next = head;
    pre.next = null;                  // as tail
    return newHead;
}

private int length(ListNode head) {
    ListNode cur = head;
    int len = 0;
    while (cur != null) {
        len ++;
        cur = cur.next;
    }
    return len;
}
```



**从已排序链表中删除所有的重复元素, 只留下不同的数组成的链表**

```
Input: 1->2->3->3->4->4->5
Output: 1->2->5
Example 2:

Input: 1->1->1->2->3
Output: 2->3
```

思路一： 递归找到第一个不等于当前值的节点，进行不断的递归

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null || head.next == null)
        return head;
    ListNode next = head.next;
    if (head.val == next.val) {
        while (next != null && head.val == next.val)  // head.val is excepted
            next = next.next;
        return deleteDuplicates(next);    // next is first not equal excepted value
    } else {
        head.next = deleteDuplicates(next);
        return head;
    }
}
```

思路二： 通过迭代指针 pre 进行控制

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode first = new ListNode(-1);
    first.next = head;
    ListNode pre = first;
    while (pre.next != null && pre.next.next != null) {
        if (pre.next.val == pre.next.next.val) {    // pre.next.val is expected
            ListNode cur = pre.next;
            while (cur.next != null && cur.val == cur.next.val)    // find last duplication
                cur = cur.next;
            pre.next = cur.next;
        } else {
            pre = pre.next;
        }
    }
    return first.next;
}
```



**从有序链表中删除重复节点，可保留1个节点**

[83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/)

```
Input: 1->1->2
Output: 1->2

Input: 1->1->2->3->3
Output: 1->2->3
```

思路一： 递归中找到最后一个等于当前节点的值的节点，递归向下进行删除

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode next = head.next;
    if (head.val == head.next.val) {
        while (next.next != null && head.val == next.next.val)
            next = next.next;   // next is last excepted
        return deleteDuplicates(next);
    }
    else {
        head.next = deleteDuplicates(next);
        return head;
    }
}
```

思路二： 通过迭代控制

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null || head.next == null)
        return head;
    ListNode pre = head;
    while (pre.next != null) {
        if (pre.val == pre.next.val) {    // can save pre
            ListNode cur = pre.next;
            while (cur.next != null && pre.val == cur.next.val)   // find last duplication
                cur = cur.next;
            pre.next = cur.next;
        } else  {
            pre = pre.next;
        }
    }
    return head;
}
```



**反转 m,n 位置之间的节点**

[92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

```
Input: 1->2->3->4->5->NULL, m = 2, n = 4
Output: 1->4->3->2->5->NULL
```

思路： 定位到第 m 个节点的前面，反转 m 到 n 的节点，同时将反转后的尾节点与原来的链表进行连接，之后与原来的链表进行连接

```java
public ListNode reverseBetween(ListNode head, int m, int n) {
    if(head == null) return null;
    ListNode first = new ListNode(-1);
    first.next = head;
    ListNode pre = first;
    for(int i = 0; i<m-1; i++)
        pre = pre.next;   // find mth node previous
    // link reverse list
    pre.next = reverseCount(pre.next, n - m);
    return first.next;
}

private ListNode reverseCount(ListNode head, int n) {
    ListNode reversedTail = head;
    ListNode pre = null;
    ListNode cur = head;
    ListNode next = null;
    for (int i = 0; i <= n; i ++) {   // Note: must <=n, because return is pre
        next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    reversedTail.next = next;
    return pre;
}
```



**判断链表是否有环**

[141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

```java
public boolean hasCycle(ListNode head) {
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) 
            return true;
    }
    return false;
}
```

**返回环的入口节点**

```
Input: head = [3,2,0,-4], pos = 1
Output: tail connects to node index 1

Input: head = [1,2], pos = 0
Output: tail connects to node index 0

Input: head = [1], pos = -1
Output: no cycle


Follow up:
Can you solve it without using extra space?
```

[142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

思路一：FastPointer的移动速度是SlowPointer的两倍。当SlowPointer走了k个结点进
入环路时，FastPointer已进入链表环路k个结点。也就是说FastPointer和SlowPointer相距
LOOP_SIZE - k个结点。
接下来，若SlowPointer每走一个结点，FastPointer就走两个结点，每走一次，两者的距离就会更近一个结点。
因此，在走了LOOP_SIZE - k次后，它们就会碰在一起。
这时两者距离环路起始处有k个结点。   即 LOOP_SIZE-k
从 entrance to merge 共 LOOP_SIZE-k
从 merge to entrance 共 LOOP_SIZE-(LOOP_SIZE-k)=k 步

```java
public ListNode detectCycle(ListNode head) {
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (fast == slow)
            break;
    }
    if (fast == null || fast.next == null)
        return null;   // have no loop
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow;
}
```

思路二： 遍历链表，确定是否有环
计算环中的节点个数
快指针走环的步数，之后快慢指针同时走碰撞处即为环的入口节点

```java
public ListNode detectCycle(ListNode head) {
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (fast == slow)
            break;
    }
    if (fast == null || fast.next == null)
        return null;   // no loop

    int loopCnt = lengthOfLoop(fast);
    fast = head;
    while (loopCnt -- > 0)
        fast = fast.next;
    slow = head;
    while (fast != slow) {
        slow = slow.next;
        fast = fast.next;
    }
    return fast;
}

private int lengthOfLoop(ListNode head) {
    int loopSize = 1;
    ListNode cur = head;
    cur = cur.next;
    while (cur != head) {
        loopSize ++;
        cur = cur.next;
    }
    return loopSize;
}
```





**两个链表的相交节点**

[160. Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/)

思路一： 两个模拟成环

```java
public ListNode getIntersectionNode(ListNode list1, ListNode list2) {
    ListNode cur1 = list1, cur2 = list2;
    while (cur1 != cur2) {
        cur1 = cur1 != null ? cur1.next : list2;
        cur2 = cur2 != null ? cur2.next : list1;
    }
    return cur1;
}
```



思路二： 两个指针指向的链表长度相同, 正向比较




思路三： 栈进行逆向比较



**删除链表中等于给定值的节点**

[203. Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/description/)

```
Input:  1->2->6->3->4->5->6, val = 6
Output: 1->2->3->4->5
```

思路一： 迭代控制

```java
public static ListNode removeElements(ListNode head, int val) {
    ListNode first = new ListNode(-1);
    first.next = head;
    ListNode pre = first;
    while (pre.next != null) {
        if (pre.next.val == val)
            pre.next = pre.next.next;     // val is excepted
        else
            pre = pre.next;
    }
    return first.next;
}
```

思路二： 递归控制

```java
public ListNode removeElements(ListNode head, int val) {
    if (head == null) return null;
    if (head.val == val)
        return removeElements(head.next, val);    // like traverse
    else {
        head.next = removeElements(head.next, val);
        return head;
    }
}
```



**反转链表**

[206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/description/)

思路一： 迭代控制

```java
public ListNode reverseList(ListNode head) {
    ListNode cur = head;
    ListNode pre = null;
    while (cur != null) {                   // can use for count reverse
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```

思路二： 递归控制

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null)
        return head;
    ListNode reverseHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;               // as tail
    return reverseHead;
}
```



**判断回文链表**

[234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

思路一： 无需额外空间实现

```java
public boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;
    ListNode fast = head, slow = head, pre = null;    // record slow previous node
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    pre.next = null;                // cut
    if (fast != null) slow = slow.next;

    ListNode l1 = head;
    ListNode l2 = reverse(slow);
    return isEqual(l1, l2);
}

private boolean isEqual(ListNode l1, ListNode l2) {
    while (l1 != null && l2 != null) {
        if (l1.val != l2.val) return false;
        l1 = l1.next;
        l2 = l2.next;
    }
    return l1 == null && l2 == null;
}

private ListNode reverse(ListNode list) {
    ListNode pre = null;
    ListNode next = null;
    while (list != null) {
        next = list.next;
        list.next = pre;
        pre = list;
        list = next;
    }
    return pre;
}
```

思路二： 一半空间的栈实现

```java
public boolean isPalindrome(ListNode head) {
    Stack<Integer> s = new Stack<>();           // store half node
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null) {
        s.push(slow.val);
        slow = slow.next;
        fast = fast.next.next;
    }
    if (fast != null)
        slow = slow.next;     // odd count, [slow, tail] N/2+1, now slow point to middle node
    while (slow != null) {    // also can !s.isEmpty();   slow from left to right, stack from right to left;
        if (s.pop() != slow.val)
            return false;
        slow = slow.next;
    }
    return true;
}
```

**在 O(1) 删除节点**

[237. Delete Node in a Linked List](https://leetcode.com/problems/delete-node-in-a-linked-list/description/)

思路一： 将可能的情况考虑全面即可

P1: head
P2: middle node
P3: tail node

```java
public ListNode deleteNode(ListNode head, ListNode tobeDelete) {
    if (head == null || tobeDelete == null)
        return null;
    if (tobeDelete.next != null) {   // not tail
        ListNode next = tobeDelete.next;
        tobeDelete.val = next.val;
        tobeDelete.next = next.next;
        next.next = null;
    } else {
        if (head == tobeDelete) {  // only one element
            head = null;
        } else {
            ListNode cur = head;
            while (cur.next != tobeDelete)      // find 2th from end
                cur = cur.next;
            cur.next = null;   // as tail
        }
    }
    return head;
}
```




## 复杂操作

**链表存放数字求相加后的链表**

[2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    return addTwoNumbers(l1, l2, 0);
}

private ListNode addTwoNumbers(ListNode l1, ListNode l2, int carry) {
    if (l1 == null && l2 == null && carry == 0)
        return null;
    int val = (l1 == null ? 0 : l1.val) + (l2 == null ? 0 : l2.val) + carry;
    ListNode head = new ListNode(val % 10);        // insure one element
    head.next = addTwoNumbers(l1 == null ? null : l1.next, l2 == null ? null : l2.next, val / 10);
    return head;
}
```

**合并 2 个已经排序的链表**

[21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

思路一： 递归实现

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```

思路二： 迭代控制



**合并 k 个已经排序的链表**

[23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

```
Input:
[
  1->4->5,
  1->3->4,
  2->6
]
Output: 1->1->2->3->4->4->5->6
```

思路一： 类似归并排序试下

```java
public ListNode mergeKLists(ListNode[] lists) {
    return mergeSortList(lists, 0, lists.length - 1);
}

private ListNode mergeSortList(ListNode[] lists, int lo, int hi) {
    if (lo > hi) return null;
    if (lo == hi) return lists[lo];    // only one list original condition or terminal sorted list

    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeSortList(lists, lo, mid);
    ListNode right = mergeSortList(lists, mid + 1, hi);
    return merge(left, right);
}

private ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val < l2.val) {
        l1.next = merge(l1.next, l2);
        return l1;
    } else {
        l2.next = merge(l1, l2.next);
        return l2;
    }
}
```

思路二： 类似堆排序实现

```java
public ListNode mergeKLists(ListNode[] lists) {
    return mergeSortList(lists, 0, lists.length - 1);
}

private ListNode mergeSortList(ListNode[] lists, int lo, int hi) {
    if (lo > hi) return null;
    if (lo == hi) return lists[lo];    // only one list original condition or terminal sorted list

    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeSortList(lists, lo, mid);
    ListNode right = mergeSortList(lists, mid + 1, hi);
    return merge(left, right);
}

private ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val < l2.val) {
        l1.next = merge(l1.next, l2);
        return l1;
    } else {
        l2.next = merge(l1, l2.next);
        return l2;
    }
}
```



 **分隔链表,左部分小于 x, 有部分大于等于 x**

[86. Partition List](https://leetcode.com/problems/partition-list/)

```
Input: head = 1->4->3->2->5->2, x = 3
Output: 1->2->2->4->3->5
```

题目描述： 需要调整后的链表保持原来的顺序

```java
public ListNode partition(ListNode head, int x) {
    ListNode first1 = new ListNode(-1), first2 = new ListNode(-1);
    ListNode tail1 = first1, tail2 = first2;
    while (head != null) {
        if (head.val < x) {
            tail1.next = head;
            tail1 = head;
        } else {
            tail2.next = head;
            tail2 = head;
        }
        head = head.next;
    }
    tail2.next = null;                  // as all list tail ??
    tail1.next = first2.next;
    return first1.next;
}
```



**根据有序链表构造平衡的二叉查找树**

[109. Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)

```
Given the sorted linked list: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5
```

```java
public TreeNode sortedListToBST(ListNode head) {
    if (head == null) return null;
    if (head.next == null)
        return new TreeNode(head.val);

    ListNode preMid = preMid(head);           // not need precise odd OR even count
    ListNode mid = preMid.next;
    preMid.next = null;                       // cut ⇒ two list

    TreeNode root = new TreeNode(mid.val);   // head... preMid
    root.left = sortedListToBST(head);
    root.right = sortedListToBST(mid.next);   // Note: mid.next ... tail
    return root;
}

private ListNode preMid(ListNode head) {
    ListNode slow = head, fast = head;
    ListNode pre = null;
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    return pre;
}
```





**复杂链表的深度拷贝**

[138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)



思路一： 分为三个步骤进行执行

step1. insert to cur and cur.next
step2. random connect
step3. split by one point

```
public RandomListNode copyRandomList(RandomListNode head) {
    if (head ==  null) return null;
    RandomListNode cur = head;
    while (cur != null) {
        RandomListNode copy = new RandomListNode(cur.label);
        copy.next = cur.next;
        cur.next = copy;
        cur = copy.next;
    }
    cur = head;    // traverse original list
    while (cur != null) {
        RandomListNode clone = cur.next;
        if (cur.random != null) clone.random = cur.random.next;     // random not every node have
        cur = clone.next;
    }
    cur = head;     // traverse combined list and split it
    RandomListNode copyHead = cur.next;
    while (cur.next != null) {
        RandomListNode next = cur.next;
        cur.next = next.next;
        cur = next;
    }
    return copyHead;
}
```

思路二： 通过 Map 记录拷贝

```java
public RandomListNode copyRandomList(RandomListNode head) {
    Map<RandomListNode, RandomListNode> map = new HashMap<>();   // record origin node -> copy node
    RandomListNode cur1 = head;               // use for iteration
    RandomListNode tail = new RandomListNode(-1);  // use for link copy node
    while (cur1 != null) {
        RandomListNode copy = new RandomListNode(cur1.label);
        map.put(cur1, copy);
        tail.next = copy;             // NOTE: need to link list
        tail = copy;
        cur1 = cur1.next;
    }
    for (Map.Entry<RandomListNode, RandomListNode> entry : map.entrySet())
        entry.getValue().random = map.get(entry.getKey().random);
    return map.get(head);
}
```



**按照指定规则排序原始链表**

[143. Reorder List](https://leetcode.com/problems/reorder-list/)

```
Given a singly linked list L: L0→L1→…→Ln-1→Ln,
reorder it to: L0→Ln→L1→Ln-1→L2→Ln-2→…

Given 1->2->3->4, reorder it to 1->4->2->3.

Given 1->2->3->4->5, reorder it to 1->5->2->4->3.
```

思路： 截取两条链表，将后面的链表进行反转，链条链表进行交叉连接

```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;
    ListNode preMid = preMid(head);
    ListNode l1 = head;
    ListNode l2 = reverse(preMid.next);  // NOTE: may more than l1 one element
    preMid.next = null;           // cut into two list
    mergeCross(l1, l2);
}

public ListNode mergeCross(ListNode l1, ListNode l2) {
    ListNode newHead = l1;
    while (l1 != null && l2 != null) {
        ListNode n1 = l1.next, n2 = l2.next;
        l1.next = l2;
        if (n1 == null) break;
        l2.next = n1;
        if (n2 == null) break;
        l1 = n1;
        l2 = n2;
    }
    return newHead;
}

private ListNode preMid(ListNode head) {
    ListNode fast = head, slow = head, pre = null;
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    return pre;
}

private ListNode reverse(ListNode head) {
    ListNode pre = null;
    ListNode next = null;
    while (head != null) {
        next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre;
}
```



**链表排序**

[148. Sort List](https://leetcode.com/problems/sort-list/)

```
Sort a linked list in O(n log n) time using constant space complexity.

Example 1:

Input: 4->2->1->3
Output: 1->2->3->4
Example 2:

Input: -1->5->3->4->0
Output: -1->0->3->4->5
```

思路： 分割成两条链表，分别对其进行排序，之后进行两条链表的合并

```java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode preMid = preMid(head);
    ListNode mid = preMid.next;
    preMid.next = null;

    ListNode l1 = sortList(head);
    ListNode l2 = sortList(mid);
    return merge(l1, l2);
}

private ListNode preMid(ListNode head) {
    ListNode pre = null, fast = head, slow = head;
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    return pre;
}

private ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val < l2.val) {
        l1.next = merge(l1.next, l2);
        return l1;
    } else {
        l2.next = merge(l1, l2.next);
        return l2;
    }
}
```



**链表元素按奇偶分割并拼接**

[328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/)

```
Input: 1->2->3->4->5->NULL
Output: 1->3->5->2->4->NULL

Input: 2->1->3->5->6->4->7->NULL
Output: 2->3->6->7->1->5->4->NULL
```

思路： 

```java
public ListNode oddEvenList(ListNode head) {
    if (head == null) return null;
    ListNode odd = head, even = head.next, evenHead = even;    // 保存用于最后直接链接
    while (odd.next != null && even.next != null) {          // even is fast than odd one step ⇔ odd.next != null && even.next != null
        odd.next = odd.next.next;
        odd = odd.next;
        even.next = even.next.next;
        even = even.next;
    }
    odd.next = evenHead;
    return head;
}
```





**反向表示数字的链表求和**

```
Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 8 -> 0 -> 7

Follow up:
What if you cannot modify the input lists? In other words, reversing the lists is not allowed.
```

思路一： 借助栈实现

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Stack<ListNode> s1 = buildStack(l1);
    Stack<ListNode> s2 = buildStack(l2);
    int carry = 0;
    ListNode first = new ListNode(-1);             // NOTE: make result to reverse
    while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {        // can enter the loop
        int val = (s1.isEmpty() ? 0 : s1.pop().val) + (s2.isEmpty() ? 0 : s2.pop().val) + carry;
        carry = val / 10;
        ListNode node = new ListNode(val % 10);
        node.next = first.next;                           // 头插法逆序 7087 ⇒ 7807
        first.next = node;
    }
    return first.next;
}

private Stack<ListNode> buildStack(ListNode list) {
    Stack<ListNode> s = new Stack<>();
    while (list != null) {
        s.push(list);
        list = list.next;
    }
    return s;
}
```



思路二： 将两条链表反转，对反转链表进行求和，之后将求和的链表进行反转返回

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    l1 = reverse(l1);
    l2 = reverse(l2);
    ListNode resReverse = addTwoNumbers(l1, l2, 0);
    return reverse(resReverse);
}

private ListNode addTwoNumbers(ListNode l1, ListNode l2, int carry) {
    if (l1 == null && l2 == null && carry == 0) return null;

    int val = (l1 == null ? 0 :l1.val) + (l2 == null ? 0 : l2.val) + carry;
    carry = val / 10;
    ListNode res = new ListNode(val % 10);
    res.next = addTwoNumbers(l1 == null ? null : l1.next, l2 == null ? null : l2.next, carry);  // prevent NullPointerException
    return res;
}

private ListNode reverse(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;
    ListNode next = null;
    while (cur != null) {
        next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```





**将一条链表分割成多条链表，有余时前面的个数大于后面的一个**

[725. Split Linked List in Parts](https://leetcode.com/problems/split-linked-list-in-parts/)

```
Input:
root = [1, 2, 3], k = 5
Output: [[1],[2],[3],[],[]]

Input:
root = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], k = 3
Output: [[1, 2, 3, 4], [5, 6, 7], [8, 9, 10]]
```

思路： 获取平均链表长度，计算需要多少额外的链表长度可以多一个节点

```java
public ListNode[] splitListToParts(ListNode root, int k) {
    int len = length(root);
    ListNode[] parts = new ListNode[k];
    int size = len / k, remain = len % k;
    ListNode pre = null, cur = root;
    for (int i = 0; i < k && cur != null; i ++) {         // two iteration pointer
        parts[i] = cur;
        int curSize = size + (remain -- > 0 ? 1 : 0);
        for (int j = 0; j < curSize; j ++) {
            pre = cur;
            cur = cur.next;
        }
        pre.next = null;            // split
    }
    return parts;
}

private int length(ListNode head) {
    int len = 0;
    ListNode cur = head;
    while (cur != null) {
        len ++;
        cur = cur.next;
    }
    return len;
}
```


# 二叉树问题  

BST: 二分查找树  
BT:  二叉树  
CBT: 完全二叉树  

LCA(Lowest Common Ancestor): 最近的公共祖先  

序列化和反序列化  
clone    

二叉树的左叶子、右叶子、最后一层的叶子、所有叶子节点的性质    
二叉树中左下角的节点   
二叉树的路径问题，必须经过跟节点/可不经过根节点，路径距离，路径上节点的和  


可以算出来的属性:  
高度  
整棵树的节点数  
左边的节点数  
右边的节点数  
树的parent节点(Map迭代过程中记录收集)  


二分查找数问题  

树的基本性质问题:  
遍历问题    
深度问题    
平衡性问题  


判断性质类问题:  
判断给定树是否是 BST
判断一颗树是否对称


思路:  
分析可能性；
包含当前节点、不包含；
定义消息体；
可能性中需要什么数据来进行确定具体的；


问题的分类:  
性质类, 可作为结构的工具方法类:  
求解深度相关  
二分搜索树相关  
平衡二叉树相关  
树的迭代相关  
节点的祖先问题  


操作类，特殊操作，多个树的操作:  
与其他数据结构结合: 根据有序数组构造 BST，链表构造出平衡二叉树, 二叉树转变成链表    
与数字结合: 将路径表示为
与其他算法结合  
两个树  
子树  
子序列  
转化  
路径问题  


实现:  
模拟为每个节点增加高度、最长路径属性...


**反转二叉树**

[226. Invert Binary Tree(easy)](https://leetcode.com/problems/invert-binary-tree/description/)

```
Input:
     4
   /   \
  2     7
 / \   / \
1   3 6   9
Output:
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

思路: 遍历更换左右子树

```java
public TreeNode invertTree(TreeNode root) {
    if (root == null)
        return null;
    TreeNode tmp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(tmp);
    return root;
}
```



**判断一颗树是否是另一颗树的子树**
[572. Subtree of Another Tree(Easy)](https://leetcode.com/problems/subtree-of-another-tree/)

```
     3
    / \
   4   5
  / \
 1   2
    /
   0
Given tree t:
   4
  / \
 1   2
Return false.
```

思路: 遍历大树，从其任意一个节点出发的子树与将要匹配的子树进行匹配  

```java
public boolean isSubtree(TreeNode s, TreeNode t) {
    if (s == null || t == null)
        return false;
    if (s.val == t.val)
        if (isSame(s, t))
            return true;
    return isSubtree(s.left, t) || isSubtree(s.right, t);  
}

private boolean isSame(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null)
        return true;
    if (t1 == null || t2 == null)     
        return false;
    if (t1.val != t2.val)
        return false;
    return isSame(t1.left, t2.left) && isSame(t1.right, t2.right);
}
```



**归并两颗树**
[617. Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/)

```
Input:
	Tree 1                     Tree 2
          1                         2
         / \                       / \
        3   2                     1   3
       /                           \   \
      5                             4   7
Output:
Merged tree:
	     3
	    / \
	   4   5
	  / \   \
	 5   4   7
```

```java
public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
    if (t1 == null)
        return t2;
    if (t2 == null)
        return t1;
    TreeNode root = new TreeNode(t1.val + t2.val);    
    root.left = mergeTrees(t1.left, t2.left);
    root.right = mergeTrees(t1.right, t2.right);
    return root;
}
```


## 遍历相关

**判断两棵树是否相同**
[100. Same Tree(Easy)](https://leetcode.com/problems/same-tree/description/)

```
Input:     1         1
          / \       / \
         2   1     1   2

        [1,2,1],   [1,1,2]

Output: false
```

思路：遍历查找比较

```java
public boolean isSameTree(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null)
        return true;
    if (root1 == null || root2 == null)
        return false;
    if (root1.val != root2.val)
        return false;
    return isSameTree(root1.left, root2.left) && isSameTree(root1.right, root2.right);
}
```



**先序遍历**

[144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/description/)

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;

    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        res.add(node.val);
        if (node.right != null)  // R -> L
            stack.push(node.right);
        if (node.left != null)
            stack.push(node.left);
    }
    return res;
}
```



**后续遍历**

[145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/description/)

```java
public static final boolean GO = false;
public static final boolean PRINT = true;
private class Command {
    boolean op;
    TreeNode node;
    Command(boolean s, TreeNode node) {
        this.op = s;
        this.node = node;
    }
}

public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null)
        return res;
    Stack<Command> stack = new Stack<>();
    stack.push(new Command(GO, root));
    while (!stack.isEmpty()) {
        Command cmd = stack.pop();
        if (cmd.op == PRINT)
            res.add(cmd.node.val);
        else {
            stack.push(new Command(PRINT, cmd.node));
            if (cmd.node.right != null)
                stack.push(new Command(GO, cmd.node.right));
            if (cmd.node.left != null)
                stack.push(new Command(GO, cmd.node.left));
            // reverse : L --> R --> D
        }
    }
    return res;
}
```



==层序遍历问题



**之字形打印二叉树**

[103. Binary Tree Zigzag Level Order Traversal(medium)](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;

    Queue<TreeNode> q  = new LinkedList<>();
    q.offer(root);
    boolean isRight = true;
    while (!q.isEmpty()) {
        int cnt = q.size();
        List<Integer> level = new ArrayList<>();
        while (cnt -- > 0) {
            TreeNode cur = q.poll();
            level.add(cur.val);
            if (cur.left != null)
                q.offer(cur.left);
            if (cur.right != null)
                q.offer(cur.right);
        }
        if (isRight) {
            res.add(level);
        } else {
            Collections.reverse(level);
            res.add(level);
        }
        isRight = !isRight;
    }
    return res;
}
```



**按层反向遍历二叉树**

[107. Binary Tree Level Order Traversal II(easy)](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

```java
public List<List<Integer>> levelOrderBottom(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;

    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int cnt = q.size();
        List<Integer> level = new ArrayList<>();
        while (cnt -- > 0) {
            TreeNode front = q.poll();
            level.add(front.val);
            if (front.left != null)
                q.offer(front.left);
            if (front.right != null)
                q.offer(front.right);
        }
        res.add(level);
    }
    Collections.reverse(res);
    return res;
}
```



**二叉树的向右视图**

[199. Binary Tree Right Side View(medium)](https://leetcode.com/problems/binary-tree-right-side-view/)

```
Input: [1,2,3,null,5,null,4]
Output: [1, 3, 4]
   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
```

思路： 二叉树层序遍历中，在迭代访问该层最后一个节点的时候就是最右节点，直接将其放入。

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int cnt = queue.size();
        while (cnt -- > 0) {
            TreeNode front = queue.poll();
            if (cnt == 0)
                res.add(front.val);          // right edge to collect result
            if (front.left != null)
                queue.offer(front.left);
            if (front.right != null)
                queue.offer(front.right);
        }
    }
    return res;
}
```



**得到左下角的节点**
[513. Find Bottom Left Tree Value](https://leetcode.com/problems/find-bottom-left-tree-value/)

```
Input:
        1
       / \
      2   3
     /   / \
    4   5   6
       /
      7
Output:
7
```
思路: 层序遍历, 从右到左遍历, 则最后保存的便为左下角的节点

```java
public int findBottomLeftValue(TreeNode root) {
    TreeNode cur = null;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        cur = queue.poll();
        if (cur.right != null)  // R -> L
            queue.offer(cur.right);
        if (cur.left != null)
            queue.offer(cur.left);
    }
    return cur.val;
}
```



**二叉树每层的平均值**
[637. Average of Levels in Binary Tree(Easy)](https://leetcode.com/problems/average-of-levels-in-binary-tree/description/)

```
Input:
    3
   / \
  9  20
    /  \
   15   7
Output: [3, 14.5, 11]
```

思路： 层序遍历

```java
public List<Double> averageOfLevels(TreeNode root) {
    List<Double> res = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int cnt = queue.size();
        double sum = 0.0;
        for (int i = 0; i < cnt; i ++) {         
            TreeNode node = queue.poll();
            sum += node.val;
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        res.add(sum / cnt);
    }
    return res;
}
```





==BST 与 中序遍历

**判断给定树是否是 BST**
[98. Validate Binary Search Tree(Medium)](https://leetcode.com/problems/validate-binary-search-tree/)

```
    5
   / \
  1   4
     / \
    3   6
Output: false
```

思路一: 中序遍历看是否有序

```java
public boolean isValidBST(TreeNode root) {
    if (root == null) return true;
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    TreeNode pre = null;      
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        if (pre != null && cur.val < pre.val) return false;   // compare
        pre = cur;
        cur = cur.right;
    }
    return true;
}
```

思路二： todo





**寻找二叉查找树的第 k 个元素**
[230. Kth Smallest Element in a BST(Medium)](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

```
Input: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
Output: 3
Follow up: 经常更改，如何处理
```

思路一： 简单的中序遍历实现

```java
public int kthSmallest(TreeNode root, int k) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        if (-- k == 0) return cur.val;
        cur = cur.right;
    }
    throw new IllegalArgumentException();
}
```

思路二： 通过遍历模拟为树中每个节点添加其左子树的秩来实现

在左子树数量正好为 k - 1 时，此时恰好为该节点；

在左子树数量大于 k - 1时，此时需要查找的元素在左子树上，转化成在左子树上寻找第 k 个元素；

在左子树数量小于 k - 1 时，此时需要查找的元素在右子树上，转化成在右子树上寻找第 k - leftSize + 1 个元素；

```java
public int kthSmallest(TreeNode root, int k) {
    int leftSize = count(root.left);
    if (leftSize == k-1)
        return root.val;
    else if (leftSize > k-1)
        return kthSmallest(root.left, k);   
    else
        return kthSmallest(root.right, k - leftSize - 1);  
}

private int count(TreeNode node) {
    if (node == null) return 0;
    return 1 + count(node.left) + count(node.right);
}
```



**二叉树左叶子的总和**
[404. Sum of Left Leaves(Easy)](https://leetcode.com/problems/sum-of-left-leaves/)

```
    3
   / \
  9  20
    /  \
   15   7
values 9 and 15 respectively. Return 24.
```

思路： 通过遍历寻找到左叶子，之后加上左叶子的值，同时加上对应右子树上所有的左叶子值

```java
public int sumOfLeftLeaves(TreeNode root) {
    if (root == null)
        return 0;
    if (isLeaf(root.left))  // meet
        return root.left.val + sumOfLeftLeaves(root.right);
    return sumOfLeftLeaves(root.left) + sumOfLeftLeaves(root.right);  
}
private boolean isLeaf(TreeNode node) {
    if (node == null)
        return false;
    return node.left == null && node.right == null;
}
```



**在二叉查找树中查找两个节点之差的最小绝对值**
[530. Minimum Absolute Difference in BST(easy)](https://leetcode.com/problems/minimum-absolute-difference-in-bst/)

```
Input:

   1
    \
     3
    /
   2
Output:
1
```

思路: 通过中序遍历，在有序时求解两者的差值即可。

```java
private int minDiff = Integer.MAX_VALUE;
private TreeNode pre = null;

public int getMinimumDifference(TreeNode root) {
    inOrder(root);
    return minDiff;
}
private void inOrder(TreeNode node) {
    if (node == null) return;
    inOrder(node.left);
    if (pre != null)
        minDiff = Math.min(minDiff, node.val - pre.val);   
    pre = node;
    inOrder(node.right);
}
```



**二叉搜索树中是否有两个节点值等于给定的数**
[653. Two Sum IV - Input is a BST(easy)](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/)

```
Input:
    5
   / \
  3   6
 / \   \
2   4   7

Target = 28

Output: False
```

思路一: 转换成数组求解，双指针处理

```java
public boolean findTarget(TreeNode root, int k) {
    List<Integer> list = new ArrayList<>();
    inOrder(root, list);
    int i = 0, j = list.size() - 1;
    while (i < j) {
        int sum = list.get(i) + list.get(j);
        if (sum == k)
            return true;
        else if (sum < k)
            i ++;
        else
            j --;
    }
    return false;
}
private void inOrder(TreeNode root, List<Integer> list) {
    if (root == null) return;
    inOrder(root.left, list);
    list.add(root.val);
    inOrder(root.right, list);
}
```



==间隔遍历

**小偷偷房子,不可偷连续的两个房子**
[337. House Robber III(Medium)](https://leetcode.com/problems/house-robber-iii/)

```
Input: [3,2,3,null,3,null,1]

     3
    / \
   2   3
    \   \
     3   1

Output: 7
Explanation: Maximum amount of money the thief can rob = 3 + 3 + 1 = 7.
```

思路一： 两者可能的情况

放弃抢当前层，从而抢其下面的两个房子；

抢当前层，并抢其下两层的房子 LL, LR, RL, RR ；

徐泽两种情况的最大值作为结果

```java
public int rob(TreeNode root) {
    if (root == null)
        return 0;
    int val1 = root.val 
        + (root.left != null ? rob(root.left.left) + rob(root.left.right) : 0)
            + (root.right != null ? rob(root.right.left) + rob(root.right.right) : 0);  
    int val2 = rob(root.left) + rob(root.right);
    return Math.max(val1, val2);
}
```

改进一: 通过记忆化搜索实现

借助 Map 来记录键为 TreeNode 的情况

```java
private Map<TreeNode, Integer> memo;

public int rob(TreeNode root) {
    memo = new HashMap<>();
    return tryRob(root);
}

private int tryRob(TreeNode root) {
    if (root == null)
        return 0;
    if (root.left == null && root.right == null)
        return root.val;
    if (memo.containsKey(root))
        return memo.get(root);
    int val1 = root.val + (root.left != null ? tryRob(root.left.left) + tryRob(root.left.right) : 0)
            + (root.right != null ? tryRob(root.right.left) + tryRob(root.right.right) : 0);
    int val2 = tryRob(root.left) + tryRob(root.right);
    int val =  Math.max(val1, val2);
    memo.put(root, val);
    return val;
}
```

改进二：通过 DP 记录实现


## 高度路径问题

**二叉树的最大深度**
[104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)

```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);
    return 1 + Math.max(leftDepth, rightDepth);
}
```



**判断一个树是否是平衡树**
[110. Balanced Binary Tree(easy)](https://leetcode.com/problems/balanced-binary-tree/)

```
Given the following tree [1,2,2,3,3,null,null,4,4]:
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
Return false.
```

思路： 每次在计算到节点高度时，只需要知道其左右孩子的高度即可，而函数的语义便是求解高度的；

```java
private boolean isBalanced = true;

public boolean isBalanced(TreeNode root) {
    height(root);
    return isBalanced;
}

private int height(TreeNode root) {
    if (root == null) return 0;
    int left = height(root.left);
    int right = height(root.right);
    if (Math.abs(left - right) > 1)  // find by node attribute
        isBalanced = false;
    return 1 + Math.max(left, right);
}
```



**计算完全二叉树的节点个数**
[222. Count Complete Tree Nodes(Medium)](https://leetcode.com/problems/count-complete-tree-nodes/)

```
Input:
    1
   / \
  2   3
 / \  /
4  5 6

Output: 6
```

思路一： 完全二叉树的情况下若其右边界的长度与左边界的长度正好相等，此时便能够确定其为一颗满二叉树，直接根据边界长度返回最终的结果；若不是满二叉树则不断递归向下知道找到其为一颗满二叉树返回结果；空树为一颗满二叉树；

```java
public int countNodes(TreeNode root) {
    int leftDepth = leftDepth(root);
    int rightDepth = rightDepth(root);
    if (leftDepth == rightDepth)     // meet condition
        return (1 << leftDepth) - 1;
    return 1 + countNodes(root.left) + countNodes(root.right);
}

private int leftDepth(TreeNode node) {
    int depth = 0;
    while (node != null) {
        depth ++;
        node = node.left;
    }
    return depth;
}

private int rightDepth(TreeNode node) {
    int depth = 0;
    while (node != null) {
        depth ++;
        node = node.right;
    }
    return depth;
}
```

思路二： 每次求解最左路径

todo



**二叉树的维度_两个节点间最长的路径可不经过根节点**
[543. Diameter of Binary Tree(Easy)](https://leetcode.com/problems/diameter-of-binary-tree/)
```
找出二叉树最大的维度
          1
         / \
        2   3
       / \     
      4   5    
Return 3, which is the length of the path [4,2,1,3] or [5,2,1,3].
```

思路一: 等价于为每个节点添加 "diameter" 属性: 子树中相距最远的两个节点之间边的个数
相距最远等价于高度，此是高度与边数的一定等价特性

```java
int max = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return max;
}

private int height(TreeNode node) {
    if (node == null)
        return 0;
    int left = height(node.left);
    int right = height(node.right);
    max = Math.max(max, left + right);  // collect
    return 1 + Math.max(left, right);
}
```



**二叉树的最大路径和,路径为任意节点开始，任意节点结束**
[124. Binary Tree Maximum Path Sum(hard)](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

```
Input: [-10,9,20,null,null,15,7]

   -10
   / \
  9  20
    /  \
   15   7

Output: 42
```

思路: 该路径路径即为左右子树节点最长的路径中加上当前节点形成的;

语义： 当前节点作为根节点**向下**的最大路径和；

```java
private int max = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    helper(root);
    return max;
}

private int helper(TreeNode node) {
    if (node == null)
        return 0;
    int left = Math.max(0, helper(node.left));
    int right = Math.max(0, helper(node.right));

    max = Math.max(max, node.val + left + right);       
    return node.val + Math.max(left, right);            
}
```



**可不经过根节点的路径中最长的都是相同数的长度**
[687. Longest Univalue Path(Easy)](https://leetcode.com/problems/longest-univalue-path/)

```
Input:
              1
             / \
            4   5
           / \   \
          4   4   5
Output: 2
```
语义： 以当前节点为根节点向下相同节点的最大个数
```java
int maxPathLen = 0;

public int longestUnivaluePath(TreeNode root) {
    if (root == null) return 0;
    getMaxDuplicationPathLength(root);
    return maxPathLen;

}

private int getMaxDuplicationPathLength(TreeNode node) {
    if (node == null)
        return 0;

    int leftLen = getMaxDuplicationPathLength(node.left);
    int rightLen = getMaxDuplicationPathLength(node.right);
    int left = (node.left != null && node.left.val == node.val ? leftLen + 1 : 0);
    int right = (node.right != null && node.right.val == node.val ? rightLen + 1: 0);
    maxPathLen = Math.max(maxPathLen, left + right);
    return Math.max(left, right);
}
```


## 结构

**根据二叉树的前序和后续遍历构造出二叉树**

[105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```html
preorder = [3,9,20,15,7]
inorder = [9,3,15,20,7]
Return the following binary tree:
    3
   / \
  9  20
    /  \
   15   7
```

```java
Map<Integer, Integer> inorderNumIndex = new HashMap<>();

public TreeNode buildTree(int[] preorder, int[] inorder) {
    if (preorder == null || preorder.length == 0 || inorder == null || inorder.length == 0) return null;

    for (int i = 0; i < inorder.length; i ++)
        inorderNumIndex.put(inorder[i], i);
    return buildTree(preorder, 0, preorder.length-1, 0);
}

private TreeNode buildTree(int[] preorder, int preL, int preR, int inL) {
    if (preL > preR) return null;

    TreeNode root = new TreeNode(preorder[preL]);
    int index = inorderNumIndex.get(root.val);
    int leftSize = index - inL;
    root.left = buildTree(preorder, preL + 1, preL + leftSize, inL);
    root.right = buildTree(preorder, preL+leftSize+1, preR, index + 1);
    return root;
}
```



**根据中序和后续遍历构造二叉树**

[106. Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

```html
inorder = [9,3,15,20,7]
postorder = [9,15,7,20,3]
Return the following binary tree:
    3
   / \
  9  20
    /  \
   15   7
```

```java
Map<Integer, Integer> inorderNumIndex = new HashMap<>();
public TreeNode buildTree(int[] inorder, int[] postorder) {
    for (int i = 0; i < inorder.length; i ++)
        inorderNumIndex.put(inorder[i], i);
    return buildTree(0, postorder, 0, postorder.length-1);
}

private TreeNode buildTree(int inL, int[] postorder, int postL, int postR) {
    if ( postL > postR) return null;

    TreeNode root = new TreeNode(postorder[postR]);
    int index = inorderNumIndex.get(root.val);
    int leftSize = index - inL;
    root.left = buildTree(inL, postorder, postL, postL+leftSize-1);
    root.right = buildTree(index+1, postorder, postL+leftSize, postR - 1);
    return root;
}
```


## 两颗树|子树

**判断一颗树是否对称**
[101. Symmetric Tree(Easy)](https://leetcode.com/problems/symmetric-tree/description/)

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
true
```

思路： {LL, RR}, {LR, RL}

```java
public boolean isSymmetric(TreeNode root) {
    if (root == null) return true;
    return isSymmetric(root.left, root.right);
}

private boolean isSymmetric(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null)
        return true;
    if (t1 == null || t2 == null)
        return false;
    if (t1.val != t2.val)
        return false;
    return isSymmetric(t1.left, t2.right) && isSymmetric(t1.right, t2.left);   
}
```



## LCA

**二叉查找树的最近公共祖先**
[235. Lowest Common Ancestor of a Binary Search Tree(easy)](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/description/)

```
        _______6______
      /                \
  ___2__             ___8__
 /      \           /      \
0        4         7        9
        /  \
       3   5

For example, the lowest common ancestor (LCA) of nodes 2 and 8 is 6.
Another example is LCA of nodes 2 and 4 is 2.
```

思路： BST 节点有序，可以很容易的判断两个节点是否都在某一颗树上；

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null) return null;
    if (p.val < root.val && q.val < root.val)
        return lowestCommonAncestor(root.left, p, q);
    if (p.val > root.val && q.val > root.val)
        return lowestCommonAncestor(root.right, p, q);
    return root;
}
```



**二叉树的最近公共祖先**
[236. Lowest Common Ancestor of a Binary Tree(Medium)](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

```
       _______3______
      /              \
  ___5__           ___1__
 /      \         /      \
6        2       0        8
        /  \
       7    4
Note:
All of the nodes' values will be unique.
p and q are different and both values will exist in the binary tree.
```

思路一: 记录每个节点的父亲节点

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    Map<TreeNode, TreeNode> parent = new HashMap<>();
    parent.put(root, null);

    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    while (!stack.isEmpty()) {  
        root = stack.pop();
        if (root.right != null) {
            parent.put(root.right, root);                
            stack.push(root.right);
        }
        if (root.left != null) {
            parent.put(root.left, root);
            stack.push(root.left);
        }
    }

    Set<TreeNode> pAncestors = new HashSet<>();
    while (p != null) {
        pAncestors.add(p);
        p = parent.get(p);
    }
    while (!pAncestors.contains(q))
        q = parent.get(q);
    return q;
}
```

思路二: 递归性质??



思路三: 找到两条到达指定 p, q 节点的路径
转换成在 两条链表中求公共交点问题





**序列化和反序列化二叉树**
[297. Serialize and Deserialize Binary Tree(Hard)](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

```
    1
   / \
  2   3
     / \
    4   5

as "[1,2,3,null,null,4,5]"
```

思路： 对于 NULL 也作为一个节点进行遍历添加进入到返回的序列化字符串中；反序列化执行相同的逻辑；

```java
private static final String SEPARATOR = " ";
private static final String NULL = "#";
public String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        TreeNode cur = q.poll();
        if (cur == null) {
            sb.append(NULL + SEPARATOR);
            continue;
        }
        sb.append(cur.val + SEPARATOR);
        q.offer(cur.left);
        q.offer(cur.right);
    }
    return sb.toString();
}

public TreeNode deserialize(String data) {
    String[] vals = data.split(SEPARATOR);
    int index = 0;
    TreeNode root = geneTreeNode(vals[index ++]);
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        TreeNode cur = q.poll();      
        if (cur == null) continue;
        cur.left = geneTreeNode(vals[index ++]);
        cur.right = geneTreeNode(vals[index ++]); // link
        q.offer(cur.left);
        q.offer(cur.right);
    }
    return root;
}

private TreeNode geneTreeNode(String val) {
    if (val.equals(NULL))                    
        return null;
    return new TreeNode(Integer.valueOf(val));
}
```

思路二： 通过 前序遍历来实现，效率更高??





**非负节点值只含有0或2个孩子节点,2个时其比孩子节点小的树中找出第二小的数**
[671. Second Minimum Node In a Binary Tree(Easy)](https://leetcode.com/problems/second-minimum-node-in-a-binary-tree/description/)

```
Input:
    2
   / \
  2   5
     / \
    5   7

Output: 5
Input:
    2
   / \
  2   2

Output: -1
```

思路： 

```java
public int findSecondMinimumValue(TreeNode root) {
    if (root == null)
        return -1;
    if (root.left == null && root.right == null)
        return -1;

    int leftVal = root.left.val;
    int rightVal = root.right.val;
    if (leftVal == root.val)
        leftVal = findSecondMinimumValue(root.left);     // skip duplication or leaf
    if (rightVal == root.val)
        rightVal = findSecondMinimumValue(root.right);

    if (leftVal != -1 && rightVal != -1) 
        return Math.min(leftVal, rightVal);    
    return leftVal != -1 ? leftVal : rightVal;   // one or all duplication with current node
}
```



**二叉树转变成特定线性的单链表**
[114. Flatten Binary Tree to Linked List(Medium)](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)

```
    1
   / \
  2   5
 / \   \
3   4   6
The flattened tree should look like:
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

★☆☆

思路一:  这道题看了hint，说每个node的右节点都是相应先序遍历中它的下一个节点。
 所以我的思路是先把先序遍历的node顺序搞出来，然后对于这里面的每一个节点，只需要做两个操作：

  1. node.left = None
  2. node.right = 相应先序遍历中node的下一个节点

```java
public void flatten(TreeNode root) {
    if (root == null)
        return;
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    TreeNode pre = new TreeNode(-1);
    while (!stack.isEmpty()) {
        TreeNode cur = stack.pop();
        pre.right = cur;
        pre.left = null;      // single list
        pre = cur;
        if (cur.right != null)
            stack.push(cur.right);
        if (cur.left != null)
            stack.push(cur.left);
    }
}
```



**把二叉查找树每个节点的值都加上比它大的节点的值**
[538. Convert BST to Greater Tree(Easy)](https://leetcode.com/problems/convert-bst-to-greater-tree/description/)

```
Input: 
              5
            /   \
           2     13
Output: 
             18
            /   \
          20     13
```

思路: 控制实现 BST 的逆序遍历，保存逆序遍历中之前的节点和，当前节点即为该路径和

LDR 正序 ⇒ RDL 逆序；

```java
int sum = 0;

public TreeNode convertBST(TreeNode root) {
    if (root == null) return null;
    traverseRDL(root);
    return root;
}

private void traverseRDL(TreeNode node) {
    if (node == null)
        return;
    traverseRDL(node.right);
    sum += node.val;
    node.val = sum;
    traverseRDL(node.left);
}
```

## 其他

### BST 相关问题

**从有序数组中构造二叉查找树**
[108. Convert Sorted Array to Binary Search Tree(Easy)](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)


```
Given the sorted array: [-10,-3,0,5,9],
One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
      0
     / \
   -3   9
   /   /
 -10  5
```

思路： 不断选择中间值作为根节点, 之后将数组对半划分不断重复即可

```java
public TreeNode sortedArrayToBST(int[] nums) {
    if (nums == null || nums.length == 0) return null;
    return buildBSTFromArray(nums, 0, nums.length - 1);
}

private TreeNode buildBSTFromArray(int[] nums, int l, int r) {
    if (l > r) return null;
    int mid = l + (r - l) / 2;
    TreeNode root = new TreeNode(nums[mid]);
    root.left = buildBSTFromArray(nums, l, mid-1);
    root.right = buildBSTFromArray(nums, mid + 1, r);
    return root;
}
```



**从二叉搜索树中删除一个节点**
[450. Delete Node in a BST(Medium)](https://leetcode.com/problems/delete-node-in-a-bst/description/)

```
root = [5,3,6,2,4,null,7]
key = 3

    5
   / \
  3   6
 / \   \
2   4   7

One valid answer is [5,4,6,2,null,null,7], shown in the following BST.

    5
   / \
  4   6
 /     \
2       7

Another valid answer is [5,2,6,null,4,null,7].

    5
   / \
  2   6
   \   \
    4   7
```

思路一: 找到删除节点的后继节点，使用其来代替当前节点，之后借助函数的语义来删除右子树上对应的后继节点  

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null)
        return null;
    if (key < root.val)
        root.left = deleteNode(root.left, key);
    else if (key > root.val)
        root.right = deleteNode(root.right, key);
    else {                   // truncate
        if (root.left == null)
            return root.right;
        if (root.right == null)
            return root.left;
        TreeNode succ = minimum(root.right);
        root.val = succ.val;
        root.right = deleteNode(root.right, root.val);   
    }
    return root;
}

private TreeNode minimum(TreeNode node) {
    if (node.left == null)
        return node;
    return minimum(node.left);
}
```


思路二: 找到后继节点，使用后继节点来代替删除节点

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null)
        return null;
    if (key < root.val) {
        root.left = deleteNode(root.left, key);
        return root;
    }
    else if (key > root.val) {
        root.right = deleteNode(root.right, key);
        return root;
    }
    else {  
        if (root.left == null) {
            TreeNode rightNode = root.right;
            root.right = null;   
            return rightNode;
        }
        if (root.right == null) {
            TreeNode leftNode = root.left;
            root.left = null;
            return leftNode;
        }
         TreeNode succ = minimum(root.right);
         succ.right = deleteMin(root.right);
         succ.left = root.left;
         root.left = root.right = null; 
         return succ;
    }
}

private TreeNode deleteMin(TreeNode node) {
    if (node.left == null) { 
        TreeNode rightNode = node.right;
        node.right = null;
        return rightNode;
    }
    node.left = deleteMin(node.left);    
    return node;
}
```



**寻找二叉查找树中出现次数最多的值**
[501. Find Mode in Binary Search Tree(Easy)](https://leetcode.com/problems/find-mode-in-binary-search-tree/)

注: 答案可能不止一个，也就是有多个值出现的次数一样多。
```
Given BST [1,null,2,2],

   1
    \
     2
    /
   2

return [2].
```
思路: 通过一个 list 记录当前频次最大的节点中保存的值, 可能随着遍历的过程中不断的清空, 之后为其添加元素的操作。类似于转换成在有序数组中寻找词频最高的所有元素

```java
private int curCnt=1;
private int maxCnt=1;
private TreeNode pre;   
public int[] findMode(TreeNode root) {
    List<Integer> list = new ArrayList<>();   // record maxCnt val
    inOrder(root, list);
    return list.stream().mapToInt(p -> p.intValue()).toArray();
}

// inorder is ordered 1,1,1,2,2,3,4,5,5,5
private void inOrder(TreeNode node, List<Integer> list) {
    if (node == null) return;
    inOrder(node.left, list);
    if (pre != null)
        curCnt = (pre.val == node.val ? curCnt + 1 : 1);
    pre = node;
    if (maxCnt < curCnt) {
        list.clear();
        list.add(node.val);
        maxCnt = curCnt;
    } else if (curCnt == maxCnt) {   
        list.add(node.val);
    }
    inOrder(node.right, list);
}
```



**修剪二叉查找树，只保留值在 L ~ R 之间的节点**
[669. Trim a Binary Search Tree(Easy)](https://leetcode.com/problems/trim-a-binary-search-tree/)

```
Input:
    3
   / \
  0   4
   \
    2
   /
  1

  L = 1
  R = 3

Output:
      3
     /
   2
  /
 1
```

思路： 类似删除的逻辑，进行对应的拼接

```java
public TreeNode trimBST(TreeNode root, int L, int R) {
    if (root == null)
        return null;
    if (root.val < L)
        return trimBST(root.right, L, R); 
    if (root.val > R)
        return trimBST(root.left, L, R);
    root.left = trimBST(root.left, L, R);
    root.right = trimBST(root.right, L, R);
    return root;
}
```







**给定 BST 的中序遍历可以构成多少不同的二叉树**
[96. Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)


思路一: 
 [衔接](https://leetcode.com/problems/unique-binary-search-trees/discuss/31666/DP-Solution-in-6-lines-with-explanation.-F(i-n)-G(i-1)-*-G(n-i))

 首先明确n个不等的数它们能构成的二叉搜索树的种类都是相等的。

 而且1到n都可以作为二叉搜索树的根节点，当k是根节点时，它的左边有k-1个不等的数，它的右边有n-k个不等的数。

 以k为根节点的二叉搜索树的种类就是左右可能的种类的乘积。

 用递推式表示就是 h(n) = h(0)*h(n-1) + h(1)*h(n-2) + ... + h(n-1)h(0) (其中n>=2) ，其中h(0)=h(1)=1，因为0个或者1个数能组成的形状都只有一个。从1到n依次算出h(x)的值即可。


 G(n): 长度为n的序列的唯一BST数量
 F(i, n), 1 <= i <= n: 唯一BST的数量，其中数字i是BST的根，序列范围从1到n。

 G(n) = F(1, n) + F(2, n) + ... + F(n, n).

 F(i, n) = G(i-1) * G(n-i)	1 <= i <= n.
 表示以i作为根节点, 其左边在[1...i-1之间继续选择, 右边在 i,...n 之间选择, 左边从i-1个、右边从n-i个元素中选择作为根节点

 G(n) = G(0) * G(n-1) + G(1) * G(n-2) + … + G(n-1) * G(0)

```java
int numTrees(int n) {
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = 1;
    for (int i = 2; i <=n; i ++) {
        for (int j = 1; j <= i; j ++) {
            dp[i] = dp[i] + dp[j-1] * dp[i - j]; 
        }
    }
    return dp[n];
}
```

思路二:  此外这其实就是一个卡特兰数，可以直接用数学公式计算，不过上面的方法更加直观一些。

```java
int numTrees(int n) {
    long ans = 1;
    for (int i = n + 1; i <= 2 * n; i++) {
        ans = ans * i / (i - n);
    }
    return (int) (ans / (n + 1));
}
```



### 路径问题(dfs)

**从根节点到叶子节点的最小路径长度**
[111. Minimum Depth of Binary Tree(easy)](https://leetcode.com/problems/minimum-depth-of-binary-tree/)

思路一: 通过递归实现,每进入一层便将层数加一,返回值即为当前的
得到左右子树的最小路径长度,在都存在的情况下选取最小的即可,存在一个的情况下选择一个

```java
public int minDepth(TreeNode root) {
    if (root == null)
        return 0;
    int left = minDepth(root.left);
    int right = minDepth(root.right);
    if (left == 0 || right == 0)       
        return left + right + 1;
    return 1+ Math.min(left, right);     
}
```


思路二: 通过层序遍历实现

```java
public int minDepth(TreeNode root) {
    if (root == null)
        return 0;
    int depth = 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int cnt = q.size();
        depth ++;
        while (cnt -- > 0) {
            TreeNode node = q.poll();
            if (node.left == null && node.right == null)
                return depth;
            if (node.left != null)
                q.offer(node.left);
            if (node.right != null)
                q.offer(node.right);
        }
    }
    throw new RuntimeException();
}
```



**是否存在一条路径和是否等于一个数**
[112. Path Sum(Easy)](https://leetcode.com/problems/path-sum/description/)


```
 Given the below binary tree and sum = 22,
     5
    / \
   4   8
  /   / \
  11  13  4
 /  \      \
7    2      1
return true, as there exist a root-to-leaf path 5->4->11->2 which sum is 22.
```

思路一: 前序遍历方式,每遍历到一个节点减小树的规模同时减少路径和的总值

```java
public boolean hasPathSum(TreeNode root, int sum) {
    if (root == null)
        return false;
    if (root.left == null && root.right == null)
        return root.val == sum;
    return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
}
```



**找出所有路径和为给定数的路径**
[113. Path Sum II(Medium)](https://leetcode.com/problems/path-sum-ii/)

```
Given the below binary tree and sum = 22,
      5
     / \
    4   8
   /   / \
  11  13  4
 /  \    / \
7    2  5   1
Return:
[
   [5,4,11,2],
   [5,8,4,5]
]
```
思路: 在遍历的过程中收集结果,不断缩小树的规模以及求解的目标和, 附带回溯功能;
在遍历到叶子节点时需要特殊处理控制,处理回退和结果加入判断;

```java
public List<List<Integer>> pathSum(TreeNode root, int sum) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    backtracking(root, sum, new ArrayList<>(), res);
    return res;
}

private void backtracking(TreeNode root, int sum, List<Integer> path, List<List<Integer>> res) {
    if (root.left == null && root.right == null) {    
        if (sum - root.val == 0) {
            path.add(root.val);
            res.add(new ArrayList<>(path));
            path.remove(path.size() - 1);
        }
        return ;
    }
    path.add(root.val);
    if (root.left != null)
        backtracking(root.left, sum - root.val, path, res);
    if (root.right != null)
        backtracking(root.right, sum - root.val, path, res);
    path.remove(path.size() - 1);
}
```



**二叉树路径表示数字，求解总和**
[129. Sum Root to Leaf Numbers(Medium)](https://leetcode.com/problems/sum-root-to-leaf-numbers/)


```
Input: [1,2,3]
    1
   / \
  2   3
Output: 25
Therefore, sum = 12 + 13 = 25.
```

思路: DFS, 收集所有拼接的字符结果,之后进行累加

```java
public int sumNumbers(TreeNode root) {
    if (root == null) return 0;

    List<String> paths = new ArrayList<>();
    dfs(root, "", paths);
    return paths.stream().mapToInt(p -> Integer.parseInt(p)).sum();
}

private void dfs(TreeNode root, String path, List<String> paths) {
    if (root.left == null && root.right == null) {
        paths.add(path + root.val);
        return;
    }
    if (root.left != null)
        dfs(root.left, path + root.val, paths);
    if (root.right != null)
        dfs(root.right, path + root.val, paths);
}
```



**求出二叉树的所有路径**
[257. Binary Tree Paths(easy)](https://leetcode.com/problems/binary-tree-paths/description/)

```
Input:

   1
 /   \
2     3
 \
  5

Output: ["1->2->5", "1->3"]
```

```java
public List<String> binaryTreePaths(TreeNode root) {
    List<String> paths = new ArrayList<>();
    if (root == null) return paths;

    dfs(root, "", paths);
    return paths;
}

void dfs (TreeNode root, String path, List<String> paths) {
    if  (root.left == null && root.right == null) {
        paths.add(path + root.val);    
        return;
    }
    if (root.left != null)
        dfs(root.left, path + root.val + "->", paths);
    if (root.right != null)
        dfs(root.right, path + root.val + "->", paths);
}
```



**任意节点开始向下节点结束路径和为给定数的可能数**
[437. Path Sum III(easy)](https://leetcode.com/problems/path-sum-iii/description/)

```
      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

Return 3. The paths that sum to 8 are:

1.  5 -> 3
2.  5 -> 2 -> 1
3. -3 -> 11
```

思路: 最后结果为从根节点作为开始节点向下寻找+不使用根节点而加上左右子树相同逻辑的总可能数量

注: 在求解某个节点向下时因为负数问题可能在达到要求后仍然有符合要求的

```java
public int pathSum(TreeNode root, int sum) {
    if (root == null) return 0;

    int res = 0;
    res += getPathCountFromNode(root, sum);
    res += pathSum(root.left, sum);
    res += pathSum(root.right, sum);
    return res;
}

private int getPathCountFromNode(TreeNode node, int target) {
    if (node == null)
        return 0;
    int res = 0;
    if (node.val == target)  // can continue
        res += 1;
    res += getPathCountFromNode(node.left, target - node.val);
    res += getPathCountFromNode(node.right, target - node.val);
    return res;
}
```


# 数组与矩阵

数组输入条件

- 有序无重复元素
- 有序有重复元素
- 全部都为数字

- 旋转数组
  - 含有重复元素
  - 不含有重复元素

- 二维矩阵
  - 从上到下，从左到右有序
  - 下一行元素大于上一行，行内元素有序

- 直方图



子数组问题与滑动窗口的解题思路。



**无重复排序数组中元素的范围段**

[228.Summary Ranges]((https://leetcode.com/problems/summary-ranges/))

```
Input:  [0,2,3,4,6,8,9]
Output: ["0","2->4","6","8->9"]
```

```java
public List<String> summaryRanges(int[] nums) {
    List<String> res = new ArrayList<>();
    if (nums.length == 0)  {
        return res;
    }
    int begin = nums[0], end = nums[0];
    Arrays.sort(nums);
    for (int i = 1; i < nums.length; i ++) {       
        if (nums[i] == nums[i - 1] + 1) {    // continue
            end = nums[i];
        } else {
            res.add(geneRange(nums, begin, end));
            begin = nums[i];
            end = nums[i];
        }
    }
    res.add(geneRange(nums, begin, end));    // handle tail
    return res;
}

private String geneRange(int[] nums, int begin, int end) {
    if (begin == end) {
        return "" + begin;
    }
    return begin + "->" + end;
}
```



**数组元素为符合某种特征的数**

**错误的数字**

645. Set Mismatch

```
Input: nums = [1,2,2,4]
Output: [2,3]
```

```java
public int[] findErrorNums(int[] nums) {
    for (int i = 0; i < nums.length; i ++) {
        while (nums[i] != i + 1 && nums[i] != nums[nums[i] - 1]) {   
            swap(nums, i, nums[i] - 1);
        }
    }
    for (int i = 0; i < nums.length; i ++) {
        if (nums[i] != i + 1) {
            return new int[]{nums[i], i + 1};
        }
    }
    return null;
}
private void swap(int[] a, int i, int j) {
  int t = a[i];
  a[i] = a[j];
  a[j] = t;
}
```

**排序数组中删除重复的值**

[26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)

```html
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.
```

```java
public int removeDuplicates(int[] nums) {
    if (nums == null || nums.length == 0) return 0;
    if (nums.length == 1) return 1;

    int k = 0;
    for (int i = 0; i < nums.length; i ++) {
        if (i == 0 || nums[i] != nums[i - 1])  
            nums[k ++] = nums[i];
    }
    return k;
}
```

**数组中删除给定的值**

[27. Remove Element](https://leetcode.com/problems/remove-element/description/)

```java
public int removeElement(int[] nums, int val) {
    int k = 0;
    for (int i = 0; i < nums.length; i ++)
        if (nums[i] != val)                     
            nums[k ++] = nums[i];
    return k;
}
```

**排序数组中删除相同的元素，允许相同元素最多出现两次**

```html
Given nums = [1,1,1,2,2,3],

Your function should return length = 5, with the first five elements of nums being 1, 1, 2, 2 and 3 respectively.

Given nums = [0,0,1,1,1,1,2,3,3],
```

```java
public int removeDuplicates(int[] nums) {
    if (nums.length < 2) return nums.length;

    int k = 1;                 
    int count = 1;
    for (int i = 1; i < nums.length; i ++) {
        if (nums[i] != nums[i-1]) {
            nums[k ++] = nums[i];
            count = 1;
        } else {                    // excepted
            if (count < 2)
                nums[k ++] = nums[i];
            count ++;
        }
    }
    return k;
}
```



**数组中的主元素**

[169. Majority Element](https://leetcode.com/problems/majority-element/)

```java
public int majorityElement(int[] nums) {
    int candidate = -1;
    int count = 0;
    for (int num : nums) {
        if (count == 0) {    
            count = 1;
            candidate = num;
            continue;
        }
        count = num == candidate ? ++ count : -- count;
    }
    return candidate;
}
```



**找出 n/3 的主元素**

```html
Input: [3,2,3]
Output: [3]
Example 2:

Input: [1,1,1,3,3,2,2,2]
Output: [1,2]
```

思路： 每次遍历的元素只能够更新 candidate1 | candidate2 的 counter,   

或者对 candidate 进行重新选取  

或者对两者的 counter 进行 -- 操作  

判断两个 candidate 在数组中出现的频次是否符合要求 

```java
public List<Integer> majorityElement(int[] nums) {
    List<Integer> res = new ArrayList<>();
    if (nums == null || nums.length == 0) return res;

    int candidate1 = nums[0], candidate2= nums[0];
    int count1 = 0, count2 = 0;    // iterate from 0 to len, need initialize as 0
    for (int num : nums) {     // must if
        if (num == candidate1)
            count1 ++;
        else if (num == candidate2)
            count2 ++;
        else if (count1 == 0) {           // need to reset
            candidate1 = num;
            count1 = 1;
        } else if (count2 == 0) {
            candidate2 = num;
            count2 = 1;
        } else {
            count1 --;
            count2 --;
        }
    }

    count1 = count2 = 0;           // find candidate frequency
    for (int num : nums) {
        if (num == candidate1)
            count1 ++;
        else if (num == candidate2)
            count2 ++;
    }
    if (count1 > nums.length/3) res.add(candidate1);
    if (count2 > nums.length/3) res.add(candidate2);
    return res;
}
```

**构建乘积数组**

[238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self)

不使用除法，时间复杂度 o(n)

```html
Input:  [1,2,3,4]
Output: [24,12,8,6]
Note: Please solve it without division and in O(n).
```

```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] product = new int[n];
    Arrays.fill(product, 1);
    int left = 1;
    for (int i = 1; i < n; i ++) {
        left *= nums[i-1];
        product[i] = left;
    }
    int right = 1;
    for (int i = n-2; i >= 0; i --) {    // From [n-1] begin *
        right *= nums[i+1];
        product[i] *= right;            // left*right
    }
    return product;
}
```

**把数组中的 0 移到末尾**

[283. Move Zeroes](https://leetcode.com/problems/move-zeroes/description/)

```java
public void moveZeroes(int[] nums) {
    int k = 0;
    for (int i = 0; i < nums.length; i ++)
        if (nums[i] != 0)
            nums[k ++] = nums[i];
    Arrays.fill(nums, k, nums.length, 0);
}
```



## 双指针

**两数之和为给定数的位置**

[1. Two Sum](https://leetcode.com/problems/two-sum/description/)

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> valIdxMap = new HashMap<>();
    for (int i = 0; i < nums.length; i ++) {
        if (valIdxMap.containsKey(target - nums[i]))
            return new int[]{valIdxMap.get(target-nums[i]), i}; 
        valIdxMap.put(nums[i], i);
    }
    throw new IllegalStateException();
}
```

**容器中放入水的最大容量**

[11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/description/)

![img](http://img.janhen.com/20210324215723question_11.jpg)

```
Example:
Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```

```java
public int maxArea(int[] height) {
    int i = 0, j = height.length - 1;
    int maxArea = Integer.MIN_VALUE;
    while (i < j) {
        int curArea = Math.min(height[i], height[j])  * (j - i);
        maxArea = Math.max(maxArea, curArea);
        if (height[i] < height[j])     // skip left part
            i ++;
        else             // skip down part
            j --;
    }
    return maxArea;
}
```



**有序数组两数和为给定值**

[167. Two Sum II - Input array is sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

```java
public int[] twoSum(int[] nums, int target) {
    int i = 0, j = nums.length - 1;
    while (i < j) {                   
        int sum = nums[i] + nums[j];
        if (sum == target) 
            return new int[]{i + 1, j + 1};  
        else if (sum < target)
            i ++;
        else
            j --;
    }
    throw new IllegalArgumentException();
}
```



## 元组问题

**3元组和为 0 的所有可能**

[15. 3Sum](https://leetcode.com/problems/3sum/)

```html
Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```
思路： 处理重复问题, 注意循环边界
```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    if (nums.length < 3) return res;
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 2; i ++) {     
        if (i > 0 && nums[i] == nums[i -1]) continue;    
        int l = i + 1, r = nums.length - 1;         
        while (l < r) {
            if (l != i + 1 && nums[l] == nums[l-1]) {            
                l ++;
                continue;
            }
            if (r != nums.length - 1 && nums[r] == nums[r+1]) {    
                r --;
                continue;
            }
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0)
                res.add(Arrays.asList(nums[i], nums[l ++], nums[r --]));    
            else if (sum > 0)
                r --;
            else
                l ++;
        }
    }
    return res;
}
```



## 排序

**排序颜色**

```java
public void sortColors(int[] nums) {
    int lt = -1, gt = nums.length;
    int i = 0;
    int pivot = 1;
    while (i < gt) {
        if (nums[i] == pivot) 
            i ++;
        else if (nums[i] < pivot) 
            swap(nums, i ++, ++ lt);
        else 
            swap(nums, i, -- gt);
    }
}
```



**归并两个排序数组**

[88.Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/description/)

```html
Input:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

Output: [1,2,2,3,5,6]
```

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int i = m - 1, j = n - 1;
    for (int merge = m + n - 1; merge >= 0; merge--) {
        if (i < 0)
            nums1[merge] = nums2[j--];
        else if (j < 0)
            nums1[merge] = nums1[i--];
        else if (nums1[i] > nums2[j])
            nums1[merge] = nums1[i--];    // select big one
        else
            nums1[merge] = nums2[j--];
    }
}
```



## **子数组与子序列**

**最短的未排序的连续子数组**

581.Shortest Unsorted Continuous Subarray

```java
public int findUnsortedSubarray(int[] nums) {
    int[] aux = nums.clone();
    Arrays.sort(aux);

    int start = 0;            // find first not correct position
    while (start < nums.length && nums[start] == aux[start]) {
        start ++;
    }

    int end = nums.length - 1;    
    while (end > start && nums[end] == aux[end]) {  
        end --;
    }
    return end - start + 1;
}
```



**生成杨辉三角**

[118. Pascal's Triangle](https://leetcode.com/problems/pascals-triangle)

```
// TODO
```



**获得帕斯卡三角形的指定行**

[119. Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/)

```html
Input: 3
Output: [1,3,3,1]
```

```java
public List<Integer> getRow(int rowIndex) {
    List<Integer> res = new ArrayList<>();
    for (int i = 0; i < rowIndex + 1; i ++) {
        res.add(1);   
        for (int j = i - 1; j > 0; j --) {    // no need to handle 0,i position
            res.set(j, res.get(j - 1) + res.get(j));
        }
    }
    return res;
}
```



**最大的乘积子数组**

[152. Maximum Product Subarray(medium)](<https://leetcode.com/problems/maximum-product-subarray/>)

```
Input: [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```

思路： 需要处理当前遍历的值为负数的情况，维护当前最大和最小相乘结果，随着结果而不断改变的, curMax, curMin 所对应的子数组元素数量可能不一致。

DP 问题的精简实现；

```java
public int maxProduct(int[] nums) {
    int N = nums.length;
    int maxProduct = nums[0];
    int curMax = nums[0], curMin = nums[0];   
    for (int i = 1; i < nums.length; i ++) {
        if (nums[i] < 0) {
            int t = curMax;
            curMax = curMin;
            curMin = t;                      
        }
        curMax = Math.max(curMax * nums[i], nums[i]);  // is or not continue
        curMin = Math.min(curMin * nums[i], nums[i]);
        maxProduct = Math.max(curMax, maxProduct);
    }
    return maxProduct;
}
```



**最短的正数子数组和为给定数**

[209. Minimum Size Subarray Sum(medium)](https://leetcode.com/problems/minimum-size-subarray-sum/)

```java
public int minSubArrayLen(int s, int[] nums) {
    int minLen = Integer.MAX_VALUE;     // record result
    int L = 0, R = -1;
    int winSum = 0;
    while (L < nums.length) {
        if (R + 1 < nums.length && winSum < s) {
            winSum += nums[++ R];
        } else {
            winSum -= nums[L ++];
        }
        if (winSum >= s) {
            minLen = Math.min(minLen, R - L + 1);
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;           // need handle init value
}
```



**找出数组中最长的连续 1**

[485. Max Consecutive Ones(easy)](https://leetcode.com/problems/max-consecutive-ones/)

```java
public int findMaxConsecutiveOnes(int[] nums) {
    int maxLen = 0, curLen = 0;
    for (int num : nums) {
        curLen = num == 1 ? curLen + 1 : 0;  // judge
        maxLen = Math.max(maxLen, curLen);
    }
    return maxLen;
}
```



**连续子数组和等于给定数的数量**

[560. Subarray Sum Equals K(medium)](https://leetcode.com/problems/subarray-sum-equals-k/)

```java
public int subarraySum(int[] nums, int k) {     //
    Map<Integer, Integer> curSumCntMap = new HashMap<>();
    curSumCntMap.put(0, 1);

    int curSum = 0;
    int count = 0;
    for (int i = 0; i < nums.length; i ++) {
        curSum += nums[i];
        int key = curSum - k;
        if (curSumCntMap.containsKey(key)) {
            count += curSumCntMap.get(key);
        }
        curSumCntMap.put(curSum, curSumCntMap.getOrDefault(curSum, 0) + 1);
    }
    return count;
}
```



**找出子数组和最大的子数组**

[53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

```html
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

```java
public int maxSubArray(int[] nums) {
    int preSum = nums[0];
    int maxSum = preSum;
    for (int i = 1; i < nums.length; i ++) {
        preSum = (preSum > 0) ? preSum + nums[i] : nums[i];   
        maxSum = Math.max(preSum, maxSum);
    }
    return maxSum;
}
```





---

**子序列**

**最长的连续序列**

128. Longest Consecutive Sequence

```java
public int longestConsecutive(int[] nums) {
    int maxCount = 0;
    Set<Integer> set = Arrays.stream(nums).boxed().collect(Collectors.toSet());

    for (int num : nums) {
        int curCount = 1;
        int curNum = num;
        while (set.contains(++ curNum)) {
            curCount ++;
            set.remove(curNum);
        }
        curNum = num;
        while (set.contains(-- curNum)) {
            curCount ++;
            set.remove(curNum);                         // remove to prevent duplication calculation
        }
        maxCount = Math.max(maxCount, curCount);
    }
    return maxCount;
}
```





**买股票问题**



**最合适时机购买和销售股票**

[121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0)
        return 0;
    int curMin = prices[0];
    int maxProfit = 0;
    for (int i = 1; i < prices.length; i ++) {
        curMin = Math.min(curMin, prices[i]);
        maxProfit = Math.max(maxProfit, prices[i] - curMin);
    }
    return maxProfit;
}
```





**杨辉三角形**

**生成帕斯卡三角形**

118. Pascal's Triangle

```java
public List<List<Integer>> generate(int numRows) {
    List<List<Integer>> res = new ArrayList<>();
    if (numRows == 0) return res;

    res.add(Arrays.asList(1));
    if (numRows == 1)
        return res;
    res.add(Arrays.asList(1,1));
    if (numRows == 2)
        return res;
    for (int i = 2; i < numRows; i ++) {
        List<Integer> preList = res.get(i-1);
        List<Integer> curList = new ArrayList<>();
        curList.add(1);
        for (int j = 1; j < i; j ++)
            curList.add(j, preList.get(j-1) + preList.get(j));  // add not set
        curList.add(i, 1);
        res.add(new ArrayList<>(curList));
    }
    return res;
}
```



**获得帕斯卡三角形的指定行**

[119. Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/)

```java
public List<Integer> getRow(int rowIndex) {
    List<Integer> res = new ArrayList<>();
    for (int i = 0; i < rowIndex + 1; i ++) {
        res.add(1);   // every time add one element
        for (int j = i - 1; j > 0; j --) {    // no need to handle 0,i position
            res.set(j, res.get(j - 1) + res.get(j));
        }
    }
    return res;
}
```



## 矩阵问题

**重新构造矩阵**

[566. Reshape the Matrix](https://leetcode.com/problems/reshape-the-matrix/description/)

```html
Input:
nums =
[[1,2],
 [3,4]]
r = 1, c = 4
Output:
[[1,2,3,4]]

Input:
nums =
[[1,2],
 [3,4]]
r = 2, c = 4
Output:
[[1,2],
 [3,4]]
```

```java
public int[][] matrixReshape(int[][] nums, int r, int c) {
    int m = nums.length, n = nums[0].length;
    if (m*n != r*c) return nums;

    int[][] matrix = new int[r][c];
    int index = 0;
    for (int i = 0; i < r; i ++) {
        for (int j = 0; j < c; j ++) {
            matrix[i][j] = nums[index/n][index%n];   // one dimension use col to
            index ++;
        }
    }
    return matrix;
}
```



**合并时间间隔**

[56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

```html
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

```java
public List<Interval> merge(List<Interval> intervals) {
    if (intervals.size() < 2) return intervals;
    List<Interval> res = new ArrayList<>();
    intervals.sort((o1, o2) -> Integer.compare(o1.start, o2.start));          // sort to greedy
    int start = intervals.get(0).start;    // record in global
    int end = intervals.get(0).end;
    for (Interval item : intervals) {
        if (item.start <= end) {        
            end = Math.max(end, item.end);
        } else {    
            res.add(new Interval(start, end));
            start = item.start;
            end = item.end;
        }
    }
    // NOTE: handle end
    res.add(new Interval(start, end));
    return res;
}
```



**将正方形矩阵顺时针旋转90°**

[48. Rotate Image](https://leetcode.com/problems/rotate-image)

```html
Given input matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
],
rotate the input matrix in-place such that it becomes:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    int up = 0, left = 0, bottom = n -1, right = n - 1;
    while (left < right)     // only one element not need rotate
        rotateEdge(matrix, up ++, left ++, bottom --, right --);
}

// four point to rotate
private void rotateEdge(int[][] matrix, int up, int left, int bottom, int right) {
    int time = right - left;                 // rotate times
    for (int i = 0; i < time; i ++) {
        int t = matrix[up][left+i];
        matrix[up][left+i] = matrix[bottom-i][left];
        matrix[bottom-i][left] = matrix[bottom][right - i];
        matrix[bottom][right - i] = matrix[up+i][right];
        matrix[up+i][right] = t;
    }
}
```



**螺旋打印矩阵**

[54. Spiral Matrix](https://leetcode.com/problems/spiral-matrix)

```html
Input:
[
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9,10,11,12]
]
Output: [1,2,3,4,8,12,11,10,9,5,6,7]
```

```java
public  ArrayList<Integer> printMatrix(int[][] m) {
    ArrayList<Integer> res = new ArrayList<>();
    if (m == null || m.length == 0) return res;

    int up = 0, left = 0, bottom = m.length - 1, right = m[0].length - 1;
    int layers = (Math.min(m.length, m[0].length) - 1) / 2 + 1;          // math ceil
    while (layers -- > 0)           // left <= right && up <= bottom
        visitEdge(m, up ++, left ++, bottom --, right --, res);
    return res;
}

private void visitEdge(int[][] m, int up, int left, int bottom, int right,  ArrayList<Integer> res) {
    if (left == right) {
        for (int i = up; i <= bottom; i ++)
            res.add(m[i][left]);
        return;
    }
    if (up == bottom) {
        for (int i = left; i <= right; i ++)
            res.add(m[up][i]);
        return;
    }
    // →
    for (int i = left; i < right; i ++)
        res.add(m[up][i]);
    // ↓
    for (int i = up; i < bottom; i ++)
        res.add(m[i][right]);
    // ←
    for (int i = right; i > left; i --)
        res.add(m[bottom][i]);
    // ↑
    for (int i = bottom; i > up; i --)
        res.add(m[i][left]);
}
```



**螺旋填充矩阵**

[59. Spiral Matrix II](https://leetcode.com/problems/spiral-matrix-ii/)

```html
Output:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]
```

```java
int sequencer;
public int[][] generateMatrix(int n) {
    int[][] matrix = new int[n][n];
    sequencer = 1;
    int left = 0, up = 0, right = n -1, down = n - 1;
    while (left <= right)
        fillEdge(matrix, left ++, up ++, right --, down --);
    return matrix;
}

// note: direction  {--, ++}
//       extreme condition
private void fillEdge(int[][] matrix, int left, int up, int right, int down) {
    if (left == right) {
        matrix[left][up] = sequencer ++;
        return;
    }
    for (int i = left; i < right; i ++)
        matrix[up][i] = sequencer ++;
    for (int i = up; i < down; i ++)
        matrix[i][right] = sequencer ++;
    for (int i = right; i > left; i --)
        matrix[down][i] = sequencer ++;
    for (int i = down; i > up; i --)
        matrix[i][left] = sequencer ++;
}
```



**数组表示字符串对其进行加一**

[66. Plus One](https://leetcode.com/problems/plus-one/)

```html
Input: [4,3,2,1]
Output: [4,3,2,2]
```

```java
public int[] plusOne(int[] digits) {
    for (int i = digits.length - 1; i >= 0; i --) {   // use array to express num, [n-1] is bit
        if (digits[i] < 9) {
            digits[i] ++;
            return digits;
        }
        digits[i] = 0;
    }
    int[] nums =  new int[digits.length+1];      // need carry
    nums[0] = 1;
    return nums;
}
```



**将矩阵中为 0 的行和列都设置成 0**

[73. Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)

```html
Input:
[
  [1,1,1],
  [1,0,1],
  [1,1,1]
]
Output:
[
  [1,0,1],
  [0,0,0],
  [1,0,1]
]
```

```java
public void setZeroes(int[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    boolean[] rows = new boolean[m];             // whatever row or col need to modify 0
    boolean[] cols = new boolean[n];
    for (int i = 0; i < m; i ++) {
        for (int j = 0; j < n; j ++) {
            if (matrix[i][j] == 0) {
                rows[i] = true;
                cols[j] = true;
            } 
        }
    }
    for (int i = 0; i < m; i ++) {
        for (int j = 0; j < n; j ++) {
            if (rows[i] || cols[j])       // narrow space
                matrix[i][j] = 0;
        }
    }
}
```



**矩阵从左上到右下共有多少可能的路径**

[62. Unique Paths](https://leetcode.com/problems/unique-paths/)

```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i ++) {       // init [0][0] no need handle specially
        dp[i][0] = 1;
    }
    for (int j = 0; j < n; j ++) {
        dp[0][j] = 1;
    }
    for (int i = 1; i < m; i ++) {
        for (int j = 1; j < n; j ++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
        }
    }
    return dp[m - 1][n - 1];
}
```



**矩阵从左上到右下路径最小路径和**

[64. Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

```java
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;

    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    for (int i = 1; i < m; i ++) {
        dp[i][0] = dp[i - 1][0] + grid[i][0];
    }
    for (int j = 1; j < n; j ++) {
        dp[0][j] = dp[0][j - 1] + grid[0][j];
    }
    for (int i = 1; i < m; i ++) {
        for (int j = 1; j < n; j ++) {
            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
    }
    return dp[m - 1][n - 1];
}
```



**矩阵中是否存在指定的单词**

[79. Word Search](https://leetcode.com/problems/word-search/description/)

```html
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

```java
public boolean exist(char[][] board, String word) {
    int rows = board.length;
    int cols = board[0].length;
    boolean[][] visited = new boolean[rows][cols];
    for (int i = 0; i < rows; i ++) {
        for (int j = 0; j < cols; j ++) {
            if (backtracking(board, i, j, rows, cols, word, 0, visited)) {
                return true;
            }
        }
    }
    return false;
}

private boolean backtracking(char[][] board, int r, int c, int rows, int cols, String word, int index, boolean[][] visited) {
    if (index == word.length())
        return true;
    if (r >= rows || r < 0 || c >= cols || c < 0 || visited[r][c] || board[r][c] != word.charAt(index))
        return false;
    visited[r][c] = true;
    boolean hasFound = backtracking(board, r - 1, c, rows, cols, word, index + 1, visited) ||
            backtracking(board, r, c + 1, rows, cols, word, index + 1, visited) ||
            backtracking(board, r + 1, c, rows, cols, word, index + 1, visited) ||
            backtracking(board, r, c - 1, rows, cols, word, index + 1, visited);
    if (!hasFound)
        visited[r][c] = false;
    return hasFound;
}
```



## 旋转数组

**数组旋转 k 位**

[189. Rotate Array](https://leetcode.com/problems/rotate-array/)

```html
Input: [1,2,3,4,5,6,7] and k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]
```

```java
public void rotate(int[] nums, int k) {
    if (nums.length == 1) return ;

    int n = nums.length;
    k = k % n;                     // prevent unnecessary rotate
    reverse(nums, 0, n-k-1);
    reverse(nums, n-k, n-1);
    reverse(nums, 0, n-1);
}
```



## 数组元素约束

**元素范围 0_n 数组中丢失的数字**

[268. Missing Number](https://leetcode.com/problems/missing-number)

```html
Input: [3,0,1]
Output: 2
Example 2:

Input: [9,6,4,2,3,5,7,0,1]
Output: 8
```

题目描述：数组元素在 0-n 之间，但是有一个数是缺失的，要求找到这个缺失的数。

A ^ A = 0
0 1 2 3 4 5 6    7 6 3 4 5 2    ...
手动构造重复的对 ^0-n
            ^[0]-[n-1]

```java
public int missingNumber(int[] nums) {
    int res = 0;
    for (int i = 0; i < nums.length; i ++) {
        res ^= i;                  // [0,n-1]
        res ^= nums[i];            // [0,n]
    }
    res ^= nums.length;
    return res;
}
```



**元素范围 1-n 找出所有消失的数**

[448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)

```java

```





## 查找

**排序矩阵中找出 kth 小的元素**

[378. Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/)

```html
matrix = [
   [ 1,  5,  9],
   [10, 11, 13],
   [12, 13, 15]
],
k = 8,
```

题目描述： 第 k 小的元素，非第 k 小的不同元素，每行与每列是递增的

思路一： 通过 堆来进行对应的顺序添加放入，保证插入的顺序即为对应的顺序

```java
public int kthSmallest(int[][] matrix, int k) {
    PriorityQueue<Tuple> pq = new PriorityQueue<>();
    int m = matrix.length, n = matrix[0].length;
    for (int i = 0; i < n; i ++)
        pq.offer(new Tuple(0, i, matrix[0][i]));

    for (int i = 0; i < k - 1; i ++) {
        Tuple t = pq.poll();
        if (t.x == m-1) continue;
        pq.offer(new Tuple(t.x + 1, t.y, matrix[t.x+1][t.y]));
    }
    return pq.peek().val;
}


class Tuple implements Comparable<Tuple> {
    int x, y;
    int val;

    Tuple(int x, int y, int val) {
        this.x = x;
        this.y = y;
        this.val = val;
    }

    public int compareTo(Tuple that) {
        return this.val - that.val;
    }
}
```

思路二： 二分查找实现

二分查找用于范围查询，相当于为每个数添加一个属性，小于等于该数的总个数

```java
public int kthSmallest(int[][] matrix, int k) {
    int m = matrix.length, n = matrix[0].length;
    int lo = matrix[0][0], hi = matrix[m-1][n-1];   // [lo,hi]
    while (lo <= hi) {
        int mid = lo + (hi-lo)/2;
        int cnt = countOfLessEqual(matrix, mid);
        if (cnt < k) 
            lo = mid + 1;
        else 
            hi = mid - 1;    // cnt=k kth in [lo,mid], and -1 to skip loop
    }
    return lo;
}

// <=mid
private int countOfLessEqual(int[][] matrix, int key) {
    int cnt = 0;
    for (int i = 0; i < matrix.length; i ++) {
        for (int j = 0; j < matrix[0].length; j ++) {
            if (matrix[i][j] <= key) cnt ++;
            else break;      // NOTE: use sorted property
        }
    }
    return cnt;
}
```



**<u>==&& 旋转数组上的查找==</u>**





**旋转有序无重复数组中查找指定元素索引**

[33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array)

```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```

```java
public int search(int[] nums, int target) {
    int smallestIndex = -1;
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < nums[hi])     // [mid] in R half,  smallest in R half
            hi = mid;
        else         // [mid] in L half, smallest in R half
            lo = mid + 1;
    }
    smallestIndex = lo;

    lo = 0;
    hi = nums.length - 1;
    while (lo <= hi) {         // binary search with offset
        int mid = lo + (hi - lo) / 2;
        int realMid = (mid + smallestIndex) % nums.length;   // add offset
        if (nums[realMid] == target)
            return realMid;
        else if (nums[realMid] < target)
            lo = mid + 1;
        else
            hi = mid - 1;
    }
    return -1;
}
```



**旋转有序含重复元素数组中判断是否存在某个值**

[81. Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)

```java
Input: nums = [2,5,6,0,0,1,2], target = 0
Output: true
Example 2:

Input: nums = [2,5,6,0,0,1,2], target = 3
Output: false
```

思路一： 找到含重复元素旋转数组中的旋转点，之后通过带偏移的二分查找进行查找。

```java
public boolean search(int[] nums, int target) {
    if (nums == null || nums.length == 0) return false;

    int firstIndex = getFirstIndex(nums);
    if (nums[nums.length-1] >= target) {
        int index = Arrays.binarySearch(nums, firstIndex, nums.length, target);
        return index >= 0;
    } else {
        int index = Arrays.binarySearch(nums, 0, firstIndex, target);
        return index >= 0;
    }
}

private int getFirstIndex(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == nums[lo] && nums[mid] == nums[hi])
            return getMinIndex(nums, lo, hi);
        if (nums[mid] <= nums[hi])
            hi = mid;   // [mid] in sub2, first int left half, first can be [mid]
        else
            lo = mid + 1;
    }
    return lo;
}

// case1: [1,1,3,1]
// case2: [1,1,0,1]
// case3: [1,1,1,1]
// NOTE: not find first element, is find first element index
private int getMinIndex(int[] nums, int lo, int hi) {
    for (int i = lo + 1; i < hi; i ++) {
        if (nums[i] == nums[lo])
            continue;
        else if (nums[i] < nums[lo]) {    // nums[lo] as excepted
            return i;
        } else if (nums[i] > nums[lo]) {
            for (int j = i + 1; j <= hi; j ++)    // find first equal [lo]
                if (nums[j] == nums[lo])
                    return j;
        }
    }
    return lo;
}
```

思路2： 不断在有序的某个段上进行判断之后查找，分析各种可能情况

```java
public boolean search(int[] nums, int target) {
    int start = 0, end = nums.length - 1, mid = -1;
    while(start <= end) {
        mid = (start + end) / 2;
        if (nums[mid] == target) {
            return true;
        }
        //If we know for sure right side is sorted or left side is unsorted
        if (nums[mid] < nums[end] || nums[mid] < nums[start]) {
            if (target > nums[mid] && target <= nums[end]) {
                start = mid + 1;
            } else {
                end = mid - 1;
            }
            //If we know for sure left side is sorted or right side is unsorted
        } else if (nums[mid] > nums[start] || nums[mid] > nums[end]) {
            if (target < nums[mid] && target >= nums[start]) {
                end = mid - 1;
            } else {
                start = mid + 1;
            }
            //If we get here, that means nums[start] == nums[mid] == nums[end], then shifting out
            //any of the two sides won't change the result but can help remove duplicate from
            //consideration, here we just use end-- but left++ works too
        } else {
            end--;
        }
    }
    return false;
}
```



**在无序数组中找出重复的数，数组中的元素值在 1~n**

[287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

```
Input: [1,3,4,2,2]
Output: 2
Example 2:

Input: [3,1,3,4,2]
Output: 3
```
思路1： 鸽子洞原理
```java
public int findDuplicate(int[] nums) {
    int lo = 1, hi = nums.length - 1;                 // lo, hi mean value, then find count that in range in all array
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int cnt = countOfLessEqual(nums, mid);
        if (cnt > mid)
            hi = mid - 1; // duplication in left half
        else
            lo = mid + 1;   // cnt=mid, mean in [lo,mid] have no duplication, duplication(key) in right [mid+1,lo]
    }
    return lo;
}

private int countOfLessEqual(int[] nums, int key) {    // find in all array
    int cnt = 0;
    for (int num : nums)
        if (num <= key)
            cnt ++;
    return cnt;
}
```

思路2： 1~n 的特性

```java
public int findDuplicate(int[] nums) {
    int[] aux = nums.clone();
    for (int i = 0; i < nums.length; i ++) {
        while (aux[i] != i+1) {
            if (aux[i] == aux[aux[i] - 1]) {
                return aux[i];
            }
            swap(aux, i, aux[i] - 1);
        }
    }
    return -1;
}

private void swap(int[] a, int i, int j) {
    int t = a[i];
    a[i] = a[j];
    a[j] = t;
}
```



**排序数组中第一次和最后一次出现的位置**

[34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array)

```
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
Example 2:

Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
```

思路一： 构建两个查找第一次出现和最后一次出现的二分查找算法

```java
// first occur index to reduce last search range
public int[] searchRange(int[] nums, int aim) {
    int first = binarySearchFirst(nums, 0, nums.length - 1, aim);
    if (first == -1) {
        return new int[]{-1, -1};
    }
    int last = binarySearchLast(nums, first, nums.length - 1, aim);   // reduce search range
    return new int[]{first, last};
}

private int binarySearchFirst(int[] nums, int L, int R, int key) {
    while (L <= R) {
        int mid = L + (R - L) / 2;
        if (nums[mid] == key) {
            if (mid == 0 || (mid > 0 && nums[mid-1] != nums[mid]))
                return mid;
            else
                R = mid - 1;
        } else if (nums[mid] < key)
            L = mid + 1;
        else
            R = mid - 1;
    }
    return -1;
}

private int binarySearchLast(int[] nums, int L, int R, int key) {
    while (L <= R) {
        int mid = L + (R - L) / 2;
        if (nums[mid] == key) {
            if (mid == nums.length-1 || nums[mid] != nums[mid + 1])
                return mid;
            else
                L = mid + 1;
        } else if (nums[mid] < key)
            L = mid + 1;
        else
            R = mid - 1;
    }
    return - 1;
}
```

思路二： 复用同一个方法逻辑，进行处理

```java
public int[] searchRange(int[] nums, int target) {
    int first = binarySearch(nums, target);
    int last = binarySearch(nums, target + 1) - 1;
    if (first == nums.length || nums[first] != target) {
        return new int[]{-1, -1};
    } else {
        return new int[]{first, Math.max(first, last)};
    }
}

private int binarySearch(int[] nums, int target) {
    int l = 0, h = nums.length; // 注意 h 的初始值
    while (l < h) {
        int m = l + (h - l) / 2;
        if (nums[m] >= target) {
            h = m;
        } else {
            l = m + 1;
        }
    }
    return l;
}
```



**查找给定值应该插入的位置**

[35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)

思路一： [a,b] 区间方式查找

```
public int searchInsert(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (target == nums[mid])
            return lo;
        else if (target > nums[mid])
            lo = mid + 1;
        else
            hi = mid - 1;
    }
    return lo;
}
```

思路2： [a, b) 区间方式查找

```java
public int searchInsert(int[] nums, int target) {
    int lo = 0, hi = nums.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (target > nums[mid])
            lo = mid + 1;
        else                    // target in left half section,   when key=[mid] ⇔ key is left half
            hi = mid;
    }
    return lo;
}
```



**在二维矩阵中判断是否存在给定数**

[74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)

```
Input:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 13
Output: false
```

题目描述： i 行元素大于 i+1 行元素，i 行元素从左到右递增，非精确性排序的矩阵。

```java
public boolean searchMatrix(int[][] matrix, int target) {
    if (matrix == null || matrix.length == 0) return false;

    int m = matrix.length, n = matrix[0].length;
    int r = 0, c = n - 1;
    while (r < m && c >= 0) {
        if (matrix[r][c] == target)
            return true;
        else if (matrix[r][c] < target)
            r ++;
        else
            c --;
    }
    return false;
}
```



[442. Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/)

```
Input:
[4,3,2,7,8,2,3,1]

Output:
[2,3]
```

思路： 1~n 的特性

```java
public List<Integer> findDuplicates(int[] nums) {
    List<Integer> res = new ArrayList<>();
    for (int i = 0; i < nums.length; i ++) {
        while (nums[i] != i + 1 && nums[i] != nums[nums[i] - 1])
            swap(nums, i, nums[i] - 1);
    }
    for (int i = 0; i < nums.length; i ++)
        if (nums[i] != i + 1)
            res.add(nums[i]);
    return res;
}
```





## 其他

**两个已排序数组的中位数**

[4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

```
nums1 = [1, 3]
nums2 = [2]
The median is 2.0

nums1 = [1, 2]
nums2 = [3, 4]
The median is (2 + 3)/2 = 2.5
```

思路一： todo

```java
public double findMedianSortedArrays(int[] A, int[] B) {
    int m = A.length, n = B.length;
    int l = (m + n + 1) / 2;
    int r = (m + n + 2) / 2;
    return (getkth(A, 0, B, 0, l) + getkth(A, 0, B, 0, r)) / 2.0;
}

// [aL, len1)             [bL, len2)
public double getkth(int[] A, int aL, int[] B, int bL, int k) {
    if (aL > A.length - 1) return B[bL + k - 1];
    if (bL > B.length - 1) return A[aL + k - 1];
    if (k == 1) return Math.min(A[aL], B[bL]);

    int aMid = Integer.MAX_VALUE, bMid = Integer.MAX_VALUE;
    if (aL + k/2 - 1 < A.length) aMid = A[aL + k/2 - 1];
    if (bL + k/2 - 1 < B.length) bMid = B[bL + k/2 - 1];

    if (aMid < bMid)
        return getkth(A, aL + k/2, B, bL,       k - k/2);// Check: aRight + bLeft
    else
        return getkth(A, aL,       B, bL + k/2, k - k/2);// Check: bRight + aLeft
}
```


# 栈和队列

## 栈

动态性： 弹出几个，加入根据弹出逻辑形成的新的值；

单调栈，用来处理一些需要顺序性的问题，值存放的为对应的索引；

通过容量来控制最终的动态调整结果；、结构设计：

带有最小值的栈

使用两个栈实现队列

使用两个队列实现栈

设计带有最大值的队列









括号问题：

验证括号









栈特性：

用于递归

判断一个弹出序列是否为某一放入序列的弹出序列  


## 单调栈
可循环查找下一个更大的元素  

**计算逆波兰表达式**

[150. Evaluate Reverse Polish Notation(medium)](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

```
Input: ["4", "13", "5", "/", "+"]
Output: 6
Explanation: (4 + (13 / 5)) = 6
```

题目描述：有效的运算符包括 `+`, `-`, `*`, `/` 。每个运算对象可以是整数，也可以是另一个逆波兰表达式。

```java
public int evalRPN(String[] tokens) {
    Stack<Integer> stack = new Stack<>();
    int num1 = 0, num2 = 0, val = 0;
    for (String str : tokens) {
        switch (str) {              // string as switch judge condition
            case "+":
                num1 = stack.pop();
                num2 = stack.pop();
                val = num2 + num1;
                stack.push(val);
                break;
            case "-":
                num1 = stack.pop();
                num2 = stack.pop();
                val = num2 - num1;
                stack.push(val);
                break;
            case "*":
                num1 = stack.pop();
                num2 = stack.pop();
                val = num2 * num1;
                stack.push(val);
                break;
            case "/":
                num1 = stack.pop();
                num2 = stack.pop();
                val = num2 / num1;   // div not meet commutative law   
                stack.push(val);
                break;
            default:
                stack.push(Integer.parseInt(str));
                break;
        }
    }
    if (stack.isEmpty())
        return val;
    else if (stack.size() == 1)  // only one element
        return stack.pop();
    else
        throw new IllegalArgumentException();
}
```



**丑数II**

[264. Ugly Number II](https://leetcode.com/problems/ugly-number-ii/)

```
Input: n = 10
Output: 12
Explanation: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 is the sequence of the first 10 ugly numbers.
Note:
```

思路： 

```java
public int nthUglyNumber(int n) {

    int[] dp = new int[n];
    dp[0] = 1;
    int index2=0, index3=0, index5=0;     // index as index, not value
    for (int i = 1; i < n; i++) {
        // cur
        dp[i] = Math.min(dp[index2]*2, Math.min(dp[index3]*3, dp[index5]*5));
        while (dp[index2]*2 <=dp[i])
            index2 ++;
        while (dp[index3]*3 <= dp[i])
            index3 ++;
        while (dp[index5]*5 <= dp[i])
            index5 ++;
    }
    return dp[n-1];
}
```





**验证括号**

[20. Valid Parentheses(easy)](https://leetcode.com/problems/valid-parentheses/description/)

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '[') stack.push(']');
        else if (c == '{') stack.push('}');
        else {
            if (stack.isEmpty()) 
                return false;
            char top = stack.pop();
            if (top != c) 
                return false;
        }
    }
    return stack.isEmpty();  // capacity judge
}
```





**处理内嵌的逻辑**

**窄化路径**

[71. Simplify Path(medium)](https://leetcode.com/problems/simplify-path/)

```
Input: "/../"
Output: "/"
Input: "/a/../../b/../c//.//"
Output: "/c"
```

```java
public String simplifyPath(String path) {
    String[] dirs = path.split("/+");
    Stack<String> s = new Stack<>();
    for (String dir : dirs) {
        if (dir.equals(".") || dir.equals("")){  // handle split ""
            continue;
        } else if (dir.equals("..")) {          // upper layer
            if (!s.isEmpty())
                s.pop();
        } else {
            s.push(dir);
        }
    }
    if (s.isEmpty())
        return "/";

    StringBuilder sb = new StringBuilder();
    while (!s.isEmpty())
        sb.insert(0, "/"+s.pop());
    return sb.toString();
}
```



**单调栈相关问题**

**循环数组中比当前元素大的下一个元素** 

[503. Next Greater Element II (Medium)](https://leetcode.com/problems/next-greater-element-ii/description/)

```text
Input: [1,2,1]
Output: [2,-1,2]
可以循环
```

思路： 栈中从栈底到栈顶中存放元素索引的值为顺序递减的，即每次保证新插入的元素 [i] 为栈中元素最小的便保证了该结构语义。

★★☆

```java
public int[] nextGreaterElements(int[] nums) {
    int N = nums.length;
    nums = geneNewNums(nums);
    Stack<Integer> idxStack = new Stack<>();    // index
    int[] nexts = new int[N * 2];    // next
    Arrays.fill(nexts, -1);
    for (int i = 0; i < nums.length; i++) {    // 2N use for mocking loop
        while (!idxStack.isEmpty() && nums[i] > nums[idxStack.peek()]) {
            nexts[idxStack.pop()] = nums[i];
        }
        idxStack.push(i);
    }
    return Arrays.copyOfRange(nexts, 0, N);
}
private int[] geneNewNums(int[] nums) {
    int[] newNums = new int[nums.length * 2];
    int k = 0;
    for (int i = 0; i < nums.length; i ++) 
        newNums[k ++] = nums[i];
    for (int i = 0; i < nums.length; i ++) 
        newNums[k ++] = nums[i];
    return newNums;
}
```



**数组中元素与下一个比它大的元素之间的距离**

[739. Daily Temperatures (Medium)](https://leetcode.com/problems/daily-temperatures/description/)

```html
Input: [73, 74, 75, 71, 69, 72, 76, 73]
Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

```java
public int[] dailyTemperatures(int[] T) {
    Stack<Integer> idxStack = new Stack<>();  // store index
    int[] dist = new int[T.length];           // distance
    for (int i = 0; i < T.length; i ++) {
        while (!idxStack.isEmpty() && T[i] > T[idxStack.peek()]) {
            int preIndex = idxStack.pop();     // 比当前值小的向右最近的元素索引
            dist[preIndex] = i - preIndex;
        }
        idxStack.push(i);
    }
    return dist;
}
```





**处理嵌套问题**

**扁平嵌套列表迭代器**

[341. Flatten Nested List Iterator(Medium)](https://leetcode.com/problems/flatten-nested-list-iterator/)

```
Input: [[1,1],2,[1,1]]
Output: [1,1,2,1,1]

Input: [1,[4,[6]]]
Output: [1,4,6]
```

思路： 栈中保存各个嵌套的结果，逆序的放入，弹出的时候即为正序弹出；

```java
Stack<NestedInteger> stack = new Stack<>();

public NestedIterator(List<NestedInteger> nestedList) {
    for (int i = nestedList.size() - 1; i >= 0; i --) {
        stack.push(nestedList.get(i));
    }
}

@Override
public Integer next() {
    return stack.pop().getInteger();
}

@Override
public boolean hasNext() {
    while (!stack.isEmpty()) {  // iterate to find one integer
        NestedInteger cur = stack.peek();
        if (cur.isInteger())   // is or not nested
            return true;
        stack.pop();
        for (int i = cur.getList().size() - 1; i >= 0; i --) 
            stack.push(cur.getList().get(i));
    }
    return false;
}
```



**解码字符串**

[394. Decode String](https://leetcode.com/problems/decode-string/)

```
s = "3[a]2[bc]", return "aaabcbc".
s = "3[a2[c]]", return "accaccacc".
s = "2[abc]3[cd]ef", return "abcabccdcdcdef".
```

思路： 通过 stack 来处理多层嵌套的问题，不断的添加与对弹出的内容进行处理之后再放入进行处理；区分数字和字母，两者表示不同的行为；

★★★

```java
public String decodeString(String s) {
    Stack<Integer> count = new Stack<>();
    Stack<String> result = new Stack<>();
    int i = 0;
    result.push("");
    while (i < s.length()) {
        char ch = s.charAt(i);
        if (ch >= '0' && ch <= '9') {     // count
            int start = i;
            while (s.charAt(i + 1) >= '0' && s.charAt(i + 1) <= '9') 
                i++;   // make i last digit 
            count.push(Integer.parseInt(s.substring(start, i + 1)));
        } else if (ch == '[') {
            result.push("");
        } else if (ch == ']') {
            // generate string
            String str = result.pop();
            StringBuilder sb = new StringBuilder();
            int times = count.pop();
            for (int j = 0; j < times; j ++) {
                sb.append(str);
            }
            result.push(result.pop() + sb.toString());
        } else {  // character
            result.push(result.pop() + ch);
        }
        i += 1;
    }
    return result.pop();
}
```

  





## 队列

**图的 BFS 遍历寻找**

**最短单词路径** 

[127. Word Ladder (Medium)](https://leetcode.com/problems/word-ladder/description/)

```
Input:
beginWord = "hit",
endWord = "cog",
wordList = ["hot","dot","dog","lot","log","cog"]

Output: 5

one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
```

题目描述：找出一条从 beginWord 到 endWord 的最短路径，每次移动规定为改变一个字符，并且改变之后的字符串必须在 wordList 中。

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    wordList.add(beginWord);
    int N = wordList.size();
    int start = N - 1;
    int end = 0;
    while (end < N && !wordList.get(end).equals(endWord))
        end++;

    if (end == N)
        return 0;

    List<Integer>[] graphic = buildGraphic(wordList);
    return getShortestPath(graphic, start, end);
}

private List<Integer>[] buildGraphic(List<String> wordList) {
    int N = wordList.size();
    List<Integer>[] graphic = new List[N];
    for (int i = 0; i < N; i++) {
        graphic[i] = new ArrayList<>();
        for (int j = 0; j < N; j++) {
            if (isConnect(wordList.get(i), wordList.get(j))) {
                graphic[i].add(j);
            }
        }
    }
    return graphic;
}

private boolean isConnect(String s1, String s2) {
    int diffCnt = 0;
    for (int i = 0; i < s1.length(); i++) {    //  && diffCnt <= 1 
        if (s1.charAt(i) != s2.charAt(i)) {
            diffCnt++;
        }
    }
    return diffCnt == 1;
}

private int getShortestPath(List<Integer>[] graphic, int start, int end) {
    Queue<Integer> queue = new LinkedList<>();
    boolean[] visited = new boolean[graphic.length];
    queue.offer(start);
    visited[start] = true;
    int path = 1;
    while (!queue.isEmpty()) {
        int size = queue.size();
        path++;
        while (size-- > 0) {
            int cur = queue.poll();
            for (int next : graphic[cur]) {
                if (next == end)   // found
                    return path;
                if (visited[next])
                    continue;
                visited[next] = true;
                queue.offer(next);
            }
        }
    }
    return 0;
}
```



## 优先队列

第 k 个元素

找出频次出现最高的 k 个数



滑动窗口的最大值

字符串解码

树的遍历

用于内部多次迭代：

扁平嵌套列表迭代器



遍历、设计、heap、parentheses

heap、nested、解码、

ugly



kth 问题：

出现频次最大的 k 个数；

两个排序数组中找出 k 个最小的数；

**数组中第 k 大的元素**

[215. Kth Largest Element in an Array(medium)](https://leetcode.com/problems/kth-largest-element-in-an-array/description/)

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

思路一： 堆实现

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> pq = new PriorityQueue<>();
    for (int num : nums) {
        pq.offer(num);
        if (pq.size() == k + 1)
            pq.poll();
    }
    return pq.peek();
}
```

思路二： 转换成 第 N-k 小问题，使用快速选择算法

```java
public int findKthLargest(int[] nums, int k) {
    int N = nums.length;
    int lo = 0, hi = N - 1;
    k = N - k;            // ==>   kthLowest
    while (lo < hi) {
        int j = partition(nums, lo, hi);
        if (j == k)
            return nums[k];
        else if (j < k)
            lo = j + 1;
        else
            hi = j - 1;
    }
    return nums[k];
}

public int partition(int[] nums, int lo, int hi) {
    int pivot = nums[lo];
    int i = lo, j = hi + 1;
    while (true) {
        while (nums[++ i] < pivot) if (i == hi) break;       
        while (nums[-- j] > pivot) if (j == lo) break;
        if (i >= j)
            break;
        swap(nums, i, j);
    }
    swap(nums, j, lo);
    return j;
}

private static void swap(int[] a, int i, int j) {
    int t = a[i];
    a[i] = a[j];
    a[j] = t;
}
```



**滑动窗口的最大值**

[239. Sliding Window Maximum(hard)](https://leetcode.com/problems/sliding-window-maximum/)

```
Input: nums = [1,3,-1,-3,5,3,6,7], and k = 3
Output: [3,3,5,5,6,7] 
Explanation: 

Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

思路一： 通过 Deque 构建最大队列，队首的元素一直都是最大的，队尾的元素一定是整体最小的，且队列整体单调，从队首到队尾为递减的； 通过每次添加一个元素时，让其插入后为原来符合规则的结构中最小的即可；

之后是 queue 与 deque 处理有序中的删除情况，通过 sequence 来实现；

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    if (nums == null || nums.length == 0 || nums.length < k || k < 0) return new int[]{};

    MaxQueue maxQueue = new MaxQueue();
    int[] res = new int[nums.length - k];
    int index = 0;
    for (int i = 0; i < nums.length; i ++) {
        maxQueue.offer(nums[i]);
        if (maxQueue.size() == k) {
            res[index ++] = maxQueue.max();
            maxQueue.poll();
        }
    }
    return res;
}

class MaxQueue {
    Deque<Tuple> qmax      = new LinkedList<>();   
    Queue<Tuple> queue         = new LinkedList<>();
    int          sequencer = 0;

    public void offer(int val) {
        while (!qmax.isEmpty() && val >= qmax.peekLast().val)  
            qmax.pollLast();
        Tuple tuple = new Tuple(val, sequencer ++);
        queue.offer(tuple);
        qmax.offerLast(tuple);
    }

    public int poll() {
        if (queue.isEmpty())
            throw new NoSuchElementException();
        Tuple oldFront = queue.poll();
        if (qmax.peekFirst().index == oldFront.index)  
            qmax.pollFirst();
        return oldFront.val;
    }

    public int max() {
        if (queue.isEmpty())
            throw new NoSuchElementException();
        return qmax.peekFirst().val;
    }

    public int size() {
        return queue.size();
    }
}

class Tuple {
    int val;
    int index;

    Tuple(int val, int index) {
        this.val = val;
        this.index = index;
    }
}
```



**找出频次最高的 k 个数**

[347. Top K Frequent Elements(medium)](https://leetcode.com/problems/top-k-frequent-elements/)

思路一： 堆+Map 实现

todo pq 如何快速转换成 List 且每次都为最大数添加

★☆☆

```java
public List<Integer> topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freqs = new HashMap<>();
    for (int num : nums)
        freqs.put(num, freqs.getOrDefault(num, 0) + 1);

    PriorityQueue<Map.Entry<Integer, Integer>> pq = new PriorityQueue<>((o1,o2) -> o1.getValue() != o2.getValue() ? Integer.compare(o1.getValue(), o2.getValue()) : Integer.compare(o1.getKey(), o2.getKey()));
    for (Map.Entry<Integer, Integer> entry : freqs.entrySet()) {
        pq.offer(entry);
        if (pq.size() == k + 1)
            pq.poll();
    }
    List<Integer> res = new ArrayList<>();
    while (!pq.isEmpty())
        res.add(pq.poll().getKey());
    return res;
}
```

思路二： 排序 HashMap 成 LinkedHashMap 实现；之后只需要截取 LinkedHashMap 中前 k 个根据 Val 排序的 Key 即可。

```java
public List<Integer> topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freqs = new HashMap<>();
    for (int num : nums)
        freqs.put(num, freqs.getOrDefault(num, 0) + 1);

    freqs = sortByValue(freqs);
    List<Integer> res = new ArrayList<>();
    for (Map.Entry<Integer, Integer> entry : freqs.entrySet()) {
        res.add(entry.getKey());
        if (res.size() == k)
            return res;
    }
    throw new IllegalArgumentException();
}

private Map<Integer, Integer> sortByValue(Map<Integer, Integer> map) {
    List<Map.Entry<Integer, Integer>> list = new ArrayList<>(map.entrySet());
    Collections.sort(list, (o1, o2) -> o1.getValue() != o2.getValue() ? Integer.compare(o2.getValue(), o1.getValue()) : Integer.compare(o1.getKey(), o2.getKey()));
    Map<Integer, Integer> newMap = new LinkedHashMap<>();
    for (Map.Entry<Integer, Integer> entry : list) {
        newMap.put(entry.getKey(), entry.getValue());
    }
    return newMap;
}
```

思路三： 桶排序

需要控制住正好为 k 个元素

```java
public List<Integer> topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freqs = new HashMap<>();
    for (int val : nums)
        freqs.put(val, freqs.getOrDefault(val, 0) + 1);

    List<Integer>[] buckets = new ArrayList[nums.length + 1];
    for (Map.Entry<Integer, Integer> entry : freqs.entrySet()) {
        int freq = entry.getValue();
        if (buckets[freq] == null)
            buckets[freq] = new ArrayList<>();
        buckets[freq].add(entry.getKey());
    }

    List<Integer> topK = new ArrayList<>();
    for (int i = buckets.length - 1; i >= 0; i --) {
        if (buckets[i] == null) continue;
        if (topK.size() >= k) break;
        if (buckets[i].size() <= (k - topK.size()))
            topK.addAll(buckets[i]);
        else
            topK.addAll(buckets[i].subList(0, k - topK.size()));
    }
    return topK;
}
```



**一定规则有序下**

**从两个排序链表中找出前 k 个最小的対**

[373. Find K Pairs with Smallest Sums(medium)](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/)

```
Input: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
Output: [1,1],[1,1]
Explanation: The first 2 pairs are returned from the sequence:
             [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```

思路一： 通过<u>弹出最小，设法获得次小的方式实现</u>

处理 k 过大的情况， min{k, len1*len2}

```java
public List<int[]> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    List<int[]> res = new ArrayList<>();
    if(nums1 == null || nums1.length == 0 || nums2 == null || nums2.length == 0 || k <= 0) 
        return res;

    PriorityQueue<Tuple> pq = new PriorityQueue<>((o1,o2) -> Integer.compare(o1.val, o2.val));
    int len1 = nums1.length, len2 = nums2.length;
    for (int i = 0; i < len2; i ++)                         
        pq.offer(new Tuple(0, i, nums1[0] + nums2[i]));

    for (int i = 0; i < Math.min(k, len1 * len2); i ++) {// k may too big
        Tuple t = pq.poll();
        res.add(new int[]{nums1[t.x], nums2[t.y]});
        if (t.x == len1 - 1) continue;
        pq.offer(new Tuple(t.x+1, t.y, nums1[t.x + 1] + nums2[t.y]));
    }
    return res;
}

public class Tuple {
    int x, y;
    int val;

    public Tuple(int x, int y, int val) {
        this.x = x;
        this.y = y;
        this.val = val;
    }
}
```

思路二： 穷举所有可能，之后对其进行排序实现；

```java
public List<int[]> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    List<int[]> res = new ArrayList<>();
    PriorityQueue<Tuple> pq = new PriorityQueue<>();
    for (int i = 0; i < nums1.length; i ++) {
        for (int j = 0; j < nums2.length; j ++)
            pq.offer(new Tuple(nums1[i], nums2[j]));
    }

    while (k -- > 0)
        if (!pq.isEmpty())
            res.add(new int[]{pq.peek().val1, pq.poll().val2});
    return res;
}

public class Tuple implements Comparable<Tuple> {
    int val1, val2;
    int sum;

    public Tuple(int x, int y) {
        this.val1 = x;
        this.val2 = y;
        this.sum = x + y;
    }

    @Override
    public int compareTo(Tuple that) {
        return this.sum - that.sum;
    }
}
```


# 其他
## Hash 表

Map 的排序:

- `LinkedHashMap `排序

- `TreeMap` 排序

- `PriorityQueue` 放入 `Map.Entry` 进行排序



配合数组，进行统计词频、第一次、最后一次出现的位置、当前和的个数；

记录数组中 val -> index

配合 LinkedHashMap 实现根究 Value 排序并作为一个 Map 返回方便后续调用；

配合 PriorityQueue 实现不断地获取最大值；

配合 List 实现值对应多个 KEY 的应用场景；

配合 List ，在 Value 为 integer 时进行处理，如词频统计；



一对一的映射： 一个原来的节点对应一个拷贝的节点

构建一对一的映射表： 用于实现对称的一种结构

Hash 表配合滑动窗口实现，包含固定的窗口实现；

Hash 表用来去重，构建类似图形问题时，用于避免成环，形成无线循环；

存放连续的数据，通常用于求解最长的问题，最长的和谐序列、最长的连续数字等；

%%特殊的使用：

作为 DP 的动态规划表使用:

处理索引不为数字的类型，而直接为一个对象

337 问题记录抢房子的最大值

```
Map<Integer, Integer> valIdxMap;
Map<TreeNode, Integer> memo;
```







**统计词频类问题**

**CPU 任务调度**

[621. Task Scheduler(Medium)](https://leetcode.com/problems/task-scheduler/)

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B.
```

统计词频，获取最大词频，获取为最大词频的总数

使用 PriorityQueue 来实现有序性

```java
 public int leastInterval(char[] tasks, int n) {
     Map<Character, Integer> freqs = new HashMap<>();
     for (int i = 0; i < tasks.length; i++) {
         freqs.put(tasks[i], freqs.getOrDefault(tasks[i], 0) + 1);
     }

     PriorityQueue<Map.Entry<Character, Integer>> maxHeap = new PriorityQueue<>((o1, o2) -> o1.getValue() != o2.getValue() ? Integer.compare(o2.getValue(), o1.getValue()) : Integer.compare(o1.getKey(), o2.getKey()));    // NOTE: maxHeap
     maxHeap.addAll(freqs.entrySet());           // use for Entry to control

     int maxFreq = maxHeap.peek().getValue();
     int count = 0;
     while (!maxHeap.isEmpty() && maxHeap.peek().getValue() == maxFreq) {
         count ++;
         maxHeap.poll();
     }

     return Math.max(tasks.length, (maxFreq - 1) * (n + 1) + count);
 }
```

使用 LinkedList 来实现有序性

```java
public int leastInterval(char[] tasks, int n) {
    Map<Character, Integer> freqs = new HashMap<>();
    for (int i = 0; i < tasks.length; i++) {
        freqs.put(tasks[i], freqs.getOrDefault(tasks[i], 0) + 1);
    }

    freqs = sortByValue(freqs);
    int maxFreq = freqs.entrySet().iterator().next().getValue();
    int count = 0;
    for (Map.Entry<Character, Integer> entry : freqs.entrySet()) {
        if (entry.getValue() == maxFreq) {
            count ++;
        } else {
            break;
        }
    }
    return Math.max(tasks.length, (maxFreq - 1) * (n + 1) + count);
}

private Map<Character, Integer> sortByValue(Map<Character, Integer> map) {
    List<Map.Entry<Character, Integer>> list = new ArrayList<>(map.entrySet());
    Collections.sort(list, (o1, o2) -> o1.getValue() != o2.getValue() ? Integer.compare(o2.getValue(), o1.getValue()) : Character.compare(o1.getKey(), o2.getKey()));
    Map<Character, Integer> newMap = new LinkedHashMap<>();
    for (Map.Entry<Character, Integer> entry : list) {
        newMap.put(entry.getKey(), entry.getValue());
    }
    return newMap;
}
```



**数组中在 k 范围差内是否有相同的数**

[219. Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/description/)

```
Example 1:
Input: nums = [1,2,3,1], k = 3
Output: true
Example 2:
Input: nums = [1,0,1,1], k = 1
Output: true
Example 3:
Input: nums = [1,2,3,1,2,3], k = 2
Output: false
```

描述： 

思路： 维护动态的一个记录表，此动态记录表中存放的是 [i-k-1, i]  之间的唯一数字，注意约束；

```java
public boolean containsNearbyDuplicate(int[] nums, int k) {
    if (nums == null || nums.length <= 1)
        return false;
    if (k <= 0)
        return false;

    Set<Integer> record = new HashSet<>();  // fixed capacity k
    for (int i = 0; i < nums.length; i ++) {
        if (record.contains(nums[i]))  // NOTE: ∃x, x==nums[i]
            return true;
        record.add(nums[i]);  // init and strategy
        if (record.size() == k + 1) // fixed capacity
            record.remove(nums[i - k]);
    }
    return false;
}
```



**在 k 索引范围内两数是否有小于等于 t**

[220. Contains Duplicate III](https://leetcode.com/problems/contains-duplicate-iii/description/)

```java
public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
    TreeSet<Long> record = new TreeSet<>();
    for (int i = 0; i < nums.length; i ++) {
        if (record.ceiling( (long) nums[i] - (long) t) != null &&      // NOTE: [i]-t=<x<=[i]+t, x∈record
                record.ceiling((long) nums[i] - (long) t) <= (long) nums[i] + (long) t) 
            return true;
        record.add((long) nums[i]);
        if (record.size() == k + 1)
            record.remove((long) nums[i - k]);
    }
    return false;
}
```



**找出两个数组中重合的数**

[349. Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/description/)

```java
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2]
Example 2:

Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [9,4]
```

思路一： 动态维护 Set，在使用完一次后便删除

```java
public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> set = new HashSet<>();
    for (int num : nums1)      // init map set
        set.add(num);
    ArrayList<Integer> list = new ArrayList<>();
    for (int num : nums2) {
        if (set.contains(num)) {   // ∃, remove
            list.add(num);
            set.remove(num);
        }
    }
    return list.stream().mapToInt(Integer::valueOf).toArray();
}
```



思路二： 排序 + 双指针；

Set 只有在两个数组有相同数据的情况下进行记录，同时排除掉对应的重复数据；

```java
public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> set = new HashSet<>();
    Arrays.sort(nums1);
    Arrays.sort(nums2);
    int i = 0, j = 0;
    while (i < nums1.length && j < nums2.length) {
        if (nums1[i] == nums2[j]) {
            set.add(nums1[i]);                 // handle duplicated element
            i ++;
            j ++;
        }
        else if (nums1[i] < nums2[j])
            i ++;
        else
            j ++;
    }
    int[] res = new int[set.size()];
    int k = 0;
    for (Integer val : set)
        res[k ++] = val;
    return res;
}
```



**两个数组相同的数，可重复**

[350. Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/description/)

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2,2]
Example 2:

Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [4,9]
```

思路一： 统计词频，使用一次减一次词频，使用完毕进行删除的一个 hash 表；

```java
public int[] intersect(int[] nums1, int[] nums2) {

    Map<Integer, Integer> freqs = new HashMap<>();
    for (int num : nums1)
        freqs.put(num, freqs.getOrDefault(num, 0) + 1);

    List<Integer> res = new ArrayList<>();
    for (int num : nums2) {               
        if (freqs.containsKey(num)) {    // have token can use
            res.add(num);
            freqs.put(num, freqs.get(num) - 1);
            if (freqs.get(num) == 0)
                freqs.remove(num);   // this element not have in all
        }
    }
    return res.stream().mapToInt(Integer::intValue).toArray();
}
```

思路二： 通过 List 来作为查找表，查找为 O(n)，效率低

```java
public int[] intersect(int[] nums1, int[] nums2) {
    List<Integer> record = new ArrayList<>();    // every element record
    List<Integer> res = new ArrayList<>();
    for (int num : nums1)  // init record list
        record.add(num);

    for (int num : nums2) {             // ∃, remove
        if (record.contains(num)) {
            res.add(num);
            record.remove(Integer.valueOf(num));  // convert to value to remove
        }
    }
    return res.stream().mapToInt(Integer::valueOf).toArray();
}
```



**三点之间距离相等的个数**

[447. Number of Boomerangs](https://leetcode.com/problems/number-of-boomerangs/description/)

```java
 Given n points in the plane that are all pairwise distinct, a "boomerang" is a tuple of points (i, j, k)
 such that the distance between i and j equals the distance between i and k (the order of the tuple matters).

Find the number of boomerangs.
 You may assume that n will be at most 500 and coordinates of points are all in the range [-10000, 10000] (inclusive).

Example:
Input:
[[0,0],[1,0],[2,0]]

Output:
2

Explanation:
The two boomerangs are [[1,0],[0,0],[2,0]] and [[1,0],[2,0],[0,0]]
```

思路一： 对于当前遍历的点，为其构建一张 hash 表，作为一个属性，记录其可以到达的距离，以及到达该距离可能点的个数；

```java
public int numberOfBoomerangs(int[][] points) {
    int res = 0;
    for (int i = 0; i < points.length; i ++) {
        Map<Integer, Integer> record = new HashMap<>();   
        for (int j = 0; j < points.length; j ++) {
            if (j != i) {
                int distance = distance(points[i], points[j]);
                record.put(distance, record.getOrDefault(distance, 0) + 1);   
            }
        }
   
        for (int key : record.keySet())  
            if (record.get(key) >= 2)
                res += record.get(key) * (record.get(key) - 1);    
    }
    return res;
}

private int distance(int[] A, int[] B) {
    return (A[0] - B[0]) * (A[0] - B[0]) + (A[1] - B[1]) * (A[1] - B[1]);
}
```





**四个数相加为0 的个数**

[454. 4Sum II](https://leetcode.com/problems/4sum-ii/description/)

```java
Input:
A = [ 1, 2]
B = [-2,-1]
C = [-1, 2]
D = [ 0, 2]

Output:
2

Explanation:
The two tuples are:
1. (0, 0, 0, 1) -> A[0] + B[0] + C[0] + D[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) -> A[1] + B[1] + C[0] + D[0] = 2 + (-1) + (-1) + 0 = 0
```

思路： 对于 {C, D} 成为一个结构，使其具备两者和为某个数的频率属性，之后统计 {A, B} 结构时借助 {C, D} 的属性；

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    Map<Integer, Integer> recordCD = new HashMap<>();
    for (int i = 0; i < C.length; i ++) {
        for (int j = 0; j < D.length; j ++) {
            int key = C[i] + D[j];
            recordCD.put(key, recordCD.getOrDefault(key, 0) + 1);
        }
    }
    int res = 0;
    for (int i = 0; i < A.length; i ++) {
        for (int j = 0; j < B.length; j ++) {
            int key = 0 - A[i] - B[j];
            if (recordCD.containsKey(key))    
                res += recordCD.get(key);
        }
    }
    return res;
}
```



**三数和最接近目标数的和**

[16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/)

```
Given array nums = [-1, 2, 1, -4], and target = 1.

The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

思路： 跳过重复元素进行加速搜索，在找到完全等于自身即最近时直接返回；

```java
public int threeSumClosest(int[] nums, int target) {
    int res = nums[0] + nums[1] + nums[nums.length - 1];    // initialization
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 2; i ++) {
        if (i > 0 && nums[i] == nums[i - 1])
            continue;
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum > target) {
                while (l < r && nums[r] == nums[r - 1]) r --;     // find last duplicated element
                r --;               // find first element not duplicated
            }
            else  {
                while (l < r && nums[l] == nums[l+1]) l ++;
                l ++;
            }
            if (Math.abs(sum - target) < Math.abs(res - target)) {
                res = sum;
                if (res - target == 0) return res;     // skip loop in advance
            }
        }
    }
    return res;
}
```



**数组中四数和为给定数的所有不同的组**

```
Given array nums = [1, 0, -1, 0, -2, 2], and target = 0.

A solution15 set is:
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

思路一： 排序+去重+双指针实现；

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res = new ArrayList<>();
    int N = nums.length;
    Arrays.sort(nums);
    for (int i = 0; i < N - 3; i ++) {
        if (i > 0 && nums[i] == nums[i - 1])      // prevent duplication
            continue;
        for (int j = i + 1; j < N - 2; j ++) {
            if (j > i + 1 && nums[j] == nums[j - 1])   // prevent duplication
                continue;
            int p = j + 1, q = N - 1;
            while (p < q) {
                int sum = nums[i] + nums[j] + nums[p] + nums[q];
                if (sum > target)
                    q --;
                else if (sum < target)
                    p ++;
                else {
                    res.add(Arrays.asList(nums[i], nums[j], nums[p], nums[q]));
                    while (p < q && nums[p] == nums[p + 1]) p ++;    // skip duplication
                    while (p < q && nums[q] == nums[q - 1]) q --;
                    p ++;
                    q --;
                }
            }
        }
    }
    return res;
}
```

思路二： hash 表实现

todo



**判断一个数字是否是快乐数字**

[202. Happy Number](https://leetcode.com/problems/happy-number/)

```
Input: 19
Output: true
Explanation:
1^2 + 9^2 = 82
8^2 + 2^2 = 68
6^2 + 8^2 = 100
1^2 + 0^2 + 0^2 = 1
```

描述： 每一位数字的平方和相加为一个数，之后对求出的数执行同样的逻辑，知道得到的数为 1；

思路： 通过 record 记录每次产生的结果，当出现重复的结果时，直接退出，未重复则继续；

```java
public boolean isHappy(int n) {
    int cur = n;
    Set<Integer> record = new HashSet<>();     // record to prevent loop
    while (cur != 1) {
        cur = getDigitSquare(cur);
        if (!record.contains(cur))
            record.add(cur);
        else           // in this have loop
            return false;
    }
    return true;
}

private int getDigitSquare(int n) {
    int sum = 0;
    while (n > 0) {
        int digit = n % 10;
        sum += digit * digit;
        n = n / 10;
    }
    return sum;
}
```









**基本应用**



**判断字符串同构**

[205. Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)

```
Input: s = "egg", t = "add"
Output: true

Input: s = "foo", t = "bar"
Output: false

Input: s = "paper", t = "title"
Output: true
```

整体为一对一映射问题，通过 hash 表实现该数据结构即可

思路一： 记录每个字符出现的位置，比较两个对应的字符出现位置，只有对应字符出现位置才能进一步向下寻找；

```java
public boolean isIsomorphic(String s, String t) {
    if (s.length() != t.length())
        return false;
    int[] hash1 = new int[256];      // record position as unique identifier
    int[] hash2 = new int[256];
    for (int i = 0; i < s.length(); i ++) {
        if (hash1[s.charAt(i)] != hash2[t.charAt(i)])
            return false;
        hash1[s.charAt(i)] = i + 1;
        hash2[t.charAt(i)] = i + 1;
    }
    return true;
}
```

思路二： 通过对 value List 的查找来实现，复杂度 O(N)

```
public boolean isIsomorphic(String s, String t) {
    if (s.length() != t.length())
        return false;
    Map<Character, Character> record = new HashMap<>();
    for (int i = 0; i < s.length(); i ++) {
        if (record.containsKey(s.charAt(i))) {
            if (record.get(s.charAt(i)) != t.charAt(i))
                return false;
        } else {
            if (record.containsValue(t.charAt(i)))
                return false;
            record.put(s.charAt(i), t.charAt(i));
        }
    }
    return true;
}
```



**判断两个字符是否是变位词**

[242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)

```
Input: s = "anagram", t = "nagaram"
Output: true

Input: s = "rat", t = "car"
Output: false

Note:
You may assume the string contains only lowercase alphabets.
```

思路一： 统计原始词频，与带匹配的字符进行对应的减词频，类似某个桶中可用的 token 数量；

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length())
        return false;
    int[] freqs = new int[26];
    for (char c : s.toCharArray())
        freqs[c-'a'] ++;
    for (char c : t.toCharArray())
        if (-- freqs[c-'a'] < 0)
            return false;
    return true;
}
```

思路二： 排序比较

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length())
        return false;
    return sort(s).equals(sort(t));
}

private String sort(String s) {
    char[] chs = s.toCharArray();
    Arrays.sort(chs);
    return new String(chs);
}
```



**判断单词是否与指定模式匹配**

[290. Word Pattern](https://leetcode.com/problems/word-pattern/)

```
Input: pattern = "abba", str = "dog cat cat dog"
Output: true
Example 2:

Input:pattern = "abba", str = "dog cat cat fish"
Output: false
```

思路： 与。。。

```java
public boolean wordPattern(String pattern, String str) {
    String[] words = str.split(" ");
    if (pattern.length() != words.length)
        return false;
    Map<Character, Integer> map1 = new HashMap<>();
    Map<String, Integer> map2 = new HashMap<>();
    for (Integer i = 0; i < pattern.length(); i ++) {            // why `Integer`,  `int` can not
        if (map1.put(pattern.charAt(i), i) != map2.put(words[i], i))
            return false;
    }
    return true;
}
```



**根据词频排序字符**

[451. Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/)

```
Input:
"Aabb"

Output:
"bbAa"
```

思路一： 统计词频，根据传入字符串构建对应的桶容量，将对应词频的桶中放入对应词频的数字；

通过 chs 字符数组进行最终结果的返回，赋值开销更小，效率比拼接更高；

```java
public String frequencySort(String s) {
    Map<Character, Integer> freqs = new HashMap<>();
    for (char c : s.toCharArray())
        freqs.put(c, freqs.getOrDefault(c, 0) + 1);

    List<Character>[] buckets = new ArrayList[s.length() + 1];
    for (char key : freqs.keySet()) {
        int freq = freqs.get(key);
        if (buckets[freq] == null)
            buckets[freq] = new ArrayList<>();
        buckets[freq].add(key);
    }

    char[] chs = s.toCharArray();
    int k = 0;
    for (int freq = buckets.length - 1; freq > 0; freq --) {
        if (buckets[freq] == null) continue;
        for (char val : buckets[freq]) {
            for (int i = 0; i < freq; i ++)
                chs[k ++] = val;
        }
    }
    return new String(chs);
}
```



**最长和谐序列**

描述： 和谐序列中最大数和最小数之差正好为 1，应该注意的是序列的元素不一定是数组的连续元素

[594. Longest Harmonious Subsequence](https://leetcode.com/problems/longest-harmonious-subsequence/)

```java
public int findLHS(int[] nums) {
    Map<Integer, Integer> freqs = new HashMap<>();
    for (int num : nums)
        freqs.put(num, freqs.getOrDefault(num, 0) + 1);

    int longest = 0;
    for (Map.Entry<Integer, Integer> entry : freqs.entrySet()) {
        if (freqs.containsKey(entry.getKey() + 1)) {
            longest = Math.max(longest, entry.getValue() + freqs.get(entry.getKey() + 1));
        }
    }
    return longest;
}
```


## 字符串

KMP-字符串匹配问题  

Manacher-回文串问题    

Trie 前缀树问题  

序列化问题, 保存值，树的序列化，用于构造，根据指定的字符串构造出指定的数据结构  

正则校验问题    

前缀问题  

回文串问题   

对称性问题  

滑动窗口  

API ：
- isLetter

- isLetterOrDigit

- toLowerCase

  包装类型进行比较的时候自动转换成基本类型比较 ???


StringBuilder
reverse
setCharAt



填充法统一处理 ：  # 增加串长度，统一 奇偶 的处理


## 滑动窗口
如何向窗口中添加新元素，如何缩小窗口，在窗口滑动的哪个阶段更新结果



**滑动窗口算法的思路是这样**：

1、我们在字符串 `S` 中使用双指针中的左右指针技巧，初始化 `left = right = 0`，把索引**左闭右开**区间 `[left, right)` 称为一个「窗口」。

2、我们先不断地增加 `right` 指针扩大窗口 `[left, right)`，直到窗口中的字符串符合要求（包含了 `T` 中的所有字符）。

3、此时，我们停止增加 `right`，转而不断增加 `left` 指针缩小窗口 `[left, right)`，直到窗口中的字符串不再符合要求（不包含 `T` 中的所有字符了）。同时，每次增加 `left`，我们都要更新一轮结果。

4、重复第 2 和第 3 步，直到 `right` 到达字符串 `S` 的尽头。



无法处理数组中的值为负值的情况



求解包含另一个子串所有元素的，包含子串排列的 


窗口的大小比较的是需要匹配的字符串，而不是大的字符串


**最下给定和为 s 的子数组**

209.Minimum Size Subarray Sum

```java
public int minSubArrayLen(int s, int[] nums) {
    int minLen = Integer.MAX_VALUE;     // record result
    int L = 0, R = -1;
    int winSum = 0;
    while (L < nums.length) {
        if (R + 1 < nums.length && winSum < s) {
            winSum += nums[R + 1];
            R ++;   // [R] in window
        } else {
            winSum -= nums[L];
            L ++;                  // sliding window only can move right
        }
        if (winSum >= s) {
            minLen = Math.min(minLen, R - L + 1);
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;           // need handle init value
}
```
