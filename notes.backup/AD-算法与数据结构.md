[TOC] 



# 基础算法

[mindmap](../mindmap/算法.xmind)

**编码**

**数学归纳法**

用于证明断言对所有自然数成立

步骤：

证明对于 N = 1 成立；

证明 N>1 时，如果对于 N-1 成立，那么对于 N 成立；



**递归控制**

严格定义递归函数作用，包括参数，返回值，Side-effect
先<u>一般</u>，后<u>特殊</u>
每次调用 <u>必须</u> 缩小问题规模
每次问题规模缩小程度必须为 <u>1</u>





## 排序
> 基本的排序算法，以及变种  

|       算法       | 稳定性 |          时间复杂度          | 空间复杂度 |            备注             |
| :--------------: | :----: | :--------------------------: | :--------: | :-------------------------: |
|     选择排序     |   ×    |        N<sup>2</sup>         |     1      |                             |
|     冒泡排序     |   √    |        N<sup>2</sup>         |     1      |                             |
|     插入排序     |   √    |      N \~ N<sup>2</sup>      |     1      |     与初始的逆序度相关      |
|     希尔排序     |   ×    | N 的若干倍乘于递增序列的长度 |     1      | 每次交换逆序度数量减少大于1 |
|     快速排序     |   ×    |            NlogN             |    logN    |                             |
| 三向切分快速排序 |   ×    |          N \~ NlogN          |    logN    |     适用于有大量重复值      |
|     归并排序     |   √    |            NlogN             |     N      |                             |
|      堆排序      |   ×    |            NlogN             |     1      |     无法利用局部性原理      |

### 基础排序

#### 选择排序

（1） 算法:

选择数组中最小的元素, 将它与数组的第一个元素交换, 之后开始次小元元素...
（2） 复杂度

比较: N²/2,  交换: N
最坏: O(n²)

最好: O(n²)，

平均: O(n²)

（3） 性质:

- 运行时间与输入无关；
- 不稳定；
- 原地排序；

```java
void selectSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    for (int i = 0; i < arr.length; i ++) {
        int minIndex = i;
        for (int j = i + 1; j < arr.length; j ++) {
            if (arr[j] < arr[minIndex])
                minIndex = i;
        }
        swap(arr,i, minIndex);
    }
}
```



#### 冒泡排序

（1） 算法: 从左到右不断交换相邻逆序的元素, 经过一次循环确定最后一个元素到达最右侧
存在传入数组已经有序的情况

（2） 复杂度分析:
  最坏: O(n²)
  最好: O(n), 集合有序, 需要进行一次冒泡
  平均: O(n²)

（3） 性质：

- 元素交换的次数为固定值, 原始数据的逆序度   需要三次赋值操作；

- 稳定；

- 原地排序；
  

**1、基础冒泡**

```java
void bubbleSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    for (int i = arr.length - 1; i > 0; i --)  // insure N-1~1 position, 0 must in correct position
        for (int j = 0; j < i; j ++)
            if (arr[j] > arr[j + 1])
                swap(arr, j, j + 1);
}
```

**2、 有序性优化**

对于已经有序的数据，不进行元素交换。

```java
void bubbleSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    boolean hasSorted = false;
    for (int i = arr.length - 1; i > 0 && !hasSorted; i --) {  // except bad condtion
        hasSorted = true;
        for (int j = 0; j < i; j ++) {
            if (arr[j] > arr[j + 1]) {    // when equal not modify original order
                hasSorted = false;
                swap(arr, j, j + 1);
            }
        }
    }
}
```



#### 插入排序

（1）算法: 将数组分为两部分，将后部分元素逐一与前部分元素比较，如果前部分元素比array[i]小，就将前部分元素往后移动。当没有比array[i]小的元素，即是合理位置，在此位置插入array[i]。

（2） 复杂度分析
最坏: O(n²), 数组逆序, 需要 N²/2 比较  N²/2 交换
最好: O(n), 正序, 需要     N-1  比较    0  次交换
平均:   O(n^2)  N²/4  比较  N²/4 交换

（3） 性质:

- 复杂度取决于数组的初始顺序， 移动次数为逆序对的数量；

- 稳定；

- 原地排序；


**1、基础插入排序**

```java
public static void insertSortB(int[] arr) {
    if (arr == null || arr.length < 2)
        return;
    for (int i = 1; i < arr.length; i++)
        for (int j = i; j > 0 && arr[j] < arr[j - 1]; j --)
            swap(arr, j, j - 1);
}
```

**2、 赋值优化**

```java
void insertSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    for (int i = 1; i < arr.length; i ++) {
        int e = arr[i], j;   // e current element, j should put position
        for (j = i; j > 0; j --) {
            if (e < arr[j-1])
            	arr[j] = arr[j-1];
        }
        arr[j] = e;
    }
}
```

**3、链表实现插入排序**

[leetcode](https://leetcode.com/problems/insertion-sort-list/)

```java
public ListNode insertionSortList(ListNode head) {
  if (head == null) {
    return head;
  }

  ListNode first = new ListNode(0);
  ListNode cur = head;  //the node will be inserted
  ListNode pre = first; //insert node between pre and pre.next
  ListNode next = null; //the next node will be inserted
  while (cur != null) {
    next = cur.next;
    //find the right place to insert
    while (pre.next != null && pre.next.val < cur.val) {
      pre = pre.next;
    }
    //insert between pre and pre.next
    cur.next = pre.next;
    pre.next = cur;
    pre = first;
    cur = next;
  }

  return first.next;
}
```



#### 希尔排序

<img src="http://img.janhen.com/20210130164857image-20201122224138741.png" alt="image-20201122224138741" style="zoom: 33%;" />

<p align="center">希尔排序可视轨迹</p>

（1） 算法：使用插入排序对间隔 h 的序列进行排序。通过不断减小 h，最后令 h=1，就可以使得整个数组是有序的。 

（2） 复杂度分析： 希尔排序的运行时间达不到平方级别，使用递增序列 1, 4, 13, 40, ... 的希尔排序所需要的比较次数不会超过 N 的若干倍乘于递增序列的长度。

（3） 性质：

- 交换不相邻元素，将逆序数量减少大于1；

- 基于原来的插入排序；

- 不稳定；

- 原地排序；

```java
void shellSort(int[] arr) {
    int N = arr.length;
    int h = 1;
    while (h < N/3) h = 3 * h + 1;
    while (h > 0) {
        for (int i = h; i < N; i += h) {
            for (int j = i; j >= h; j -= h) {
                if (arr[j] < arr[j-h]) swap(arr, j, j - h);
                else break;
            }
        }
        h /= 3;
    }
}
```



### 快速排序

（1） 思想: 分治, 分区

（2） 复杂度:
由每次选取的分割点控制
最好: 每次分割点都为中间的元素， O(logN)
最坏: 每次分割点都为最后元素   O(n²)

（3） 性质:

- 每趟排序就有一个元素排在了最终的位置上，第n趟结束，**至少**有n个元素已经排在了最终的位置上；

- 非稳定

- 原地排序

（4） 归并 VS 快排:
归并由下到上, 先处理子问题之后合并，快排由上到下, 先进行分区然后处理子问题；

归并非原地排序，需要辅助空间，快排通过原地分区函数实现原地排序；

归并排序为稳定的排序，保留原来相同值的顺序；

（5） 优化:
 三数取中法
 随机选取法

**1、随机取枢纽元**

小数据集使用插入排序；

随机选择枢纽元比较；

```java
void quickSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    quickSort(arr, 0, arr.length);
}
void quickSort(int[] arr, int lo, int hi) {
    if (hi - lo < INSERTION_SORT_THRESHOLD) {
        insertSort(arr, lo, hi);
        return;
    }
    int j = partition(arr, lo, hi);
    quickSort(arr, lo, j - 1);
    quickSort(arr, j +1, hi);
}
```

快速选择；

```java
int partition(int[] arr, int lo, int hi) {
    swap(arr,lo,lo +(int) Math.random() * (hi-lo+1);
    int pivot = arr[lo];
    int i = lo, j = hi + 1;
    while (true) {
		while (arr[++ i] < pivot) if (i == hi) break;
        while (arr[-- j] > pivot) if (j == lo) break;
        if (i >= j) break;
        swap(arr, i, j);
    }
    swap(arr, j, lo);
}
```

**2、三路快排优化**

<img src="http://img.janhen.com/20210130164902image-20201122224436315.png" alt="image-20201122224436315" style="zoom: 33%;" />

<p align="center">三向切分快排可视轨迹</p>

对重复元素较多的情形优化；

函数返回重复元素第一次和最后一次出现位置；

类似荷兰国旗问题的处理；

相关： [75. Sort Colors](https://leetcode.com/problems/sort-colors/description/)  

```java
int[] partition(int[] arr, int lo,int hi) {
    int pivot = arr[lo];
    int lt = lo - 1, gt = hi + 1;
    int i = lo;
    while (i < gt) {              // 各个区间的语义
        if (arr[i] == pivot) 
            i ++;
        else if (arr[i] < pivot)
            swap(arr, i ++, ++ lt);
        else 
            swap(arr, i, -- gt);
    }
    return new int[]{lt+1, gt-1};
}
```

**3、 三数取中值确定枢纽元优化**

枢纽元的选取上进行优化；

选取边界和中间数将三处进行排序，选择中间元素作为枢纽元，并放入 [hi-1] 位置；

之后 [lo], [hi] 可以作为快排内循环的哨兵；

```java
// sort three element AND put hi-1 position
int medianOf3(int[] arr, int lo, int hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[lo] > arr[mid]) swap(arr, lo, mid);
    if (arr[lo] > arr[hi]) swap(arr, lo, hi);
    if (arr[mid] > arr[hi]) swap(arr, mid, hi);
    swap(arr, mid, hi - 1);
    return arr[hi - 1];        // pivot is mid value, and position is hi-1
}
```

```java
int partition(int[] arr, int lo, int hi, int pivot) {  // pivot original position hi - 1
    int i = lo, j = hi - 1;
    while (true) {
        while (arr[++ i] < pivot);   // NOTE: [hi-1] as sentinel
        while (arr[-- j] > pivot);   // [lo] as sentinel
        if (i >= j) break;
        swap(arr, i, j);
    }
    swap(arr, i, hi - 1);  // put pivot as correct position
    return i;
}
```

```java
void quickSort(int[] arr, int lo, int hi) {
    if (hi - lo <= INSERTITION_SORT_THRESHOLD) {
        insertSort(arr, lo, hi);
        return;
    }
    int median = medianOf3(arr, lo, hi);
    int i = partition(arr, lo, hi, median);
    quickSort(arr, lo, i - 1);
    quickSort(arr,i +1, hi);
}

```



### 归并排序

（1） 算法

（2） 复杂度

大部分为 O(NlogN)

> T(n) = 2*T(n/2) + n
>     = 2*(2*T(n/4) + n/2) + n = 4*T(n/4) + 2*n
>     = 4*(2*T(n/8) + n/4) + 2*n = 8*T(n/8) + 3*n
>     = 8*(2*T(n/16) + n/8) + 3*n = 16*T(n/16) + 4*n
>     ......
>     = 2^k * T(n/2^k) + k * n

（3） 性质

- 大数据量情况下出现无法分配空间情况；

- 稳定的排序；

- 非原地排序；

**1、基础归并排序**

<img src="http://img.janhen.com/20210130164907image-20201122224009106.png" alt="image-20201122224009106" style="zoom: 33%;" />

<p align="center">自顶向下归并排序可视轨迹</p>

① 对排序的两个子数组 [lo,mid], [mid+1, hi]，在 [mid] >= [mid+1] 数组整体有序情况下跳过合并；

② 分配当前两个数组对应的数组空间作为辅助；

```java
void mergeSort(int[] arr, int lo, int hi) {
    if (hi - lo < INSERTITION_SORT_THRESHOLD) {
        insertSort(arr, lo, hi);
        return;
    }
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, lo, mid);
    mergeSort(arr, mid + 1, hi);
    if (arr[mid] > arr[mid + 1])    loopArrQueue
    	merge(arr, lo, mid, hi);
}
```

```java
void merge(int[] arr, int lo, int mid, int hi) {
    int[] aux = new int[hi - lo + 1];
    int i = lo, j = mid + 1;
    for (int k = 0; k < aux.length; k ++) {
        if (i > mid) aux[k] = arr[j ++];
        else if (j > hi) aux[k] = arr[i ++];
        else if (arr[i] < arr[j]) aux[k] = arr[i ++];
        else aux[k] = arr[j ++];
    }
    for (int k = 0; k < aux.length; k ++) 
       arr[k + lo] = aux[k];
}
```

**2、 自底向上的归并排序**

<img src="http://img.janhen.com/20210130164913image-20201122223815424.png" alt="image-20201122223815424" style="zoom:33%;" />

<p align="center">自底向上的归并排序可视轨迹</p>

考虑处理两种情况：

```html
P1  __ __ | __ __ | __ __ | _       i + sz < arr.length to control
P2  __ __ | __ __ | __ __ | __ _    min{i + sz + sz - 1, arr.length - 1} to control
```

sz 为两个子数组的区间大小

```java
void mergeSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int N = arr.length;
    for (int sz = 1; sz < N;  sz += sz) {
        for (int i = 0; i + sz < N; i += sz + sz) {  loopArrQueue
            merge(arr, i, i + sz - 1, Math.min(i + sz + sz - 1, N-1));
        }
    }
}
```

**3、使用链表进行归并排序**

执行过程：

① 找出中间节点，分割链表；

② 对分割的链表分别进行归并排序；

③ 将链表合并；

相关： [leetcode](https://leetcode.com/problems/sort-list/) | [leetcode-cn](https://leetcode-cn.com/problems/sort-list/)

```java
public class ListNode implements Cloneable {
  public int val;
  public ListNode next;

  public ListNode(int val) {
    this.val = val;
  }

  public ListNode(int val, ListNode next) {
    this.val = val;
    this.next = next;
  }
}
```

```java
public ListNode sortList(ListNode head) {
  if (head == null || head.next == null) {
    return head;
  }

  // 1. find mid node and cut two list
  ListNode preMid = preMidNode(head);
  ListNode mid = preMid.next;
  preMid.next = null;
  // 2. handle two sub problem
  ListNode l1 = sortList(head);
  ListNode l2 = sortList(mid);
  // 3. merge result
  return merge(l1, l2);
}

private ListNode preMidNode(ListNode head) {
  ListNode pre = null, fast = head, slow = head;
  while (fast != null && fast.next != null) {
    pre = slow;
    slow = slow.next;
    fast = fast.next.next;
  }
  return pre;
}

private ListNode merge(ListNode l1, ListNode l2) {
  if (l1 == null)
    return l2;
  if (l2 == null)
    return l1;
  if (l1.val < l2.val) {
    l1.next = merge(l1.next, l2);
    return l1;
  } else {
    l2.next = merge(l1, l2.next);
    return l2;
  }
}
```

**4、合并 k 个已经排序的链表**

[leetcode-cn](https://leetcode-cn.com/problems/merge-k-sorted-lists/) | [leetcode](https://leetcode.com/problems/merge-k-sorted-lists/)

```java
  public ListNode mergeKLists(ListNode[] lists) {
    return mergeSortList(lists, 0, lists.length - 1);
  }

  private ListNode mergeSortList(ListNode[] lists, int lo, int hi) {
    if (lo > hi) {
      return null;
    }
    if (lo == hi) {
      return lists[lo];  
    }

    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeSortList(lists, lo, mid);
    ListNode right = mergeSortList(lists, mid + 1, hi);
    return merge(left, right);
  }

  // 合并两个排序的链表
  private ListNode merge(ListNode l1, ListNode l2) {
    if (l1 == null) {
      return l2;
    }
    if (l2 == null) {
      return l1;
    }
    if (l1.val < l2.val) {
      l1.next = merge(l1.next, l2);
      return l1;
    } else {
      l2.next = merge(l1, l2.next);
      return l2;
    }
  }
```





### 堆排序

<img src="http://img.janhen.com/20210130164926image-20201122225020506.png" alt="image-20201122225020506" style="zoom:33%;" />

<p align="center">堆排序可视轨迹</p>

（1） 算法

（2） 复杂度

O(logN)

（3） 性质

- 无法利用到现代处理器的缓存局部性原理，一般不使用；

- 不稳定；

- 原地排序，适用于嵌入式系统中内存小的情况；

**1、 基础堆排序**

先通过向堆中不断插入元素，向上调整形成堆结构；

之后不断删除堆顶元素实现排序；

```java
void heapSort(int[] arr) {
    if (arr == null || arr.length < 2)
        return;
    for (int i = 0; i < arr.length; i ++)
        heapify(arr, i);                            // heapInsert
    int N = arr.length;
    while (N > 0) {                              // heapify: delete and adjust heap structure
        swap(arr, -- N, 0);
        sink(arr, N, 0);
    }
}
```

```java
void heapify(int[] arr, int k) {
    while (arr[k] > arr[(k - 1) / 2]) {
        swap(arr, k, (k - 1) / 2);
        k = (k - 1) / 2;
    }
}
```

```java
void sink(int[] arr, int N, int k) {
    while (2 * k + 1 < N) {
        int j = 2 * k + 1;
        if (j + 1 < N && arr[j] < arr[j + 1])
            j ++;
        if (arr[k] >= arr[j])
            break;
        swap(arr, k, j);
        k = j;
    }
}
```

**2、算法优化**

与 java.util.ProrityQueue 中实现逻辑相同

① 通过 sink 向下调整进行优化；

② 下沉操作中使用赋值替代交换，常数级优化；

```java
void heapSort(int[] arr) {
    if (arr == null || arr.length < 2)
        return;
    int N = arr.length;
    for (int i = (N - 2)  / 2; i >= 0; i --)      // build heap
        sink(arr, i, N);
    while (N > 0) {                   // delete max ⇔ put into last position
        swap(arr, 0, -- N);
        sink(arr, 0, N);
    }
}
```

```java
void sink(int[] arr, int k, int N) {
    int val = arr[k];
    while (k * 2 + 1 < N) {
    	int j = k * 2 + 1;
   		if (j + 1 < N && arr[j] < arr[j + 1])
   			 j = j + 1;
        if (val >= arr[j])  break;
        arr[k] = arr[j];
        k = j;
    }
    arr[k] = val;
}
```

**3、合并 k 个已经排序的链表**

```java
public ListNode mergeKLists(ListNode[] lists) {
  if (lists == null || lists.length == 0) {
    return null;
  }
  // 初始化加载所有链表的头节点
  PriorityQueue<ListNode> pq = new PriorityQueue<>(lists.length, Comparator.comparingInt(l -> l.val));   // list like each data flow
  for (ListNode list : lists) {
    if (list != null) {
      pq.offer(list);
    }
  }

  // 比较每条链表当前的头节点
  ListNode first = new ListNode(-1);
  ListNode cur = first;
  while (!pq.isEmpty()) {
    cur.next = pq.poll();
    cur = cur.next;
    if (cur.next != null) {
      pq.offer(cur.next);
    }
  }
  return first.next;
}
```








## 查找

### 二分查找

（1） 算法

（2） 复杂度

O(logN)

（3）性质

- 适用于处理 ceil、floor 等操作；

- 配合索引相当于是实现了跳表结构；

**1、 普通二分查找**

```java
int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target)
            return mid;
        if (arr[mid] < target)
            lo = mid + 1;
        else
            hi = mid - 1;
    }
    return -1;
}
```

**2、 带有重复元素的二分查找-最先&最后**

（1） 查找含有重复元素的数组集合中元素第一次出现的位置

在相等的情况下，进行判断决定是否进行缩小范围或找到对应的值；

```java
int binarySearchFirst(int[] nums, int lo, int hi, int aim) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == aim) {
            if (mid == 0 || nums[mid-1] != nums[mid])  return mid;
            else hi = mid - 1;
        } else if (nums[mid] < aim) {
            lo = mid + 1;
        } else
            hi = mid - 1;
    }
    return -1;
}
```

（2） 查找含有重复元素的数组集合中元素最后一次出现的位置

```java
int binarySearchLast(int[] nums, int key, int lo, int hi) {
    while (lo <= hi) {
        int mid = (hi - lo) / 2 + lo;
        if (nums[mid] == key) {
            if (mid == nums.length - 1 || nums[mid] != nums[mid + 1]) return mid;
            else lo = mid + 1;
        } else if (nums[mid] < key) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return -1;
}
```

**3、 二分查找-大于&小于**

（1） 查找小于等于给定元素的最小元素在数组中的位置

```java
public int binarySearchFloor(int[] nums, int key, int lo, int hi) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] <= key) {
            if (mid == nums.length-1 || nums[mid+1] > key) return mid;
            else lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return -1;
}
```

（2） 查找大于等于给定元素的最小元素在数组中的位置

```java
public int binarySearchCeil(int[] nums, int key, int lo, int hi) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= key) {
            if (mid == 0 || nums[mid-1] < key) return mid;
            else hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return -1;
}
```

**4、 带偏移的二分查找**

用于旋转数组的查找，偏移后数据有序；

```java
int binarySearchOffset(int[] nums, int key, int offset) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int realMid = (mid + offset) % nums.length;
        if (nums[realMid] == key) {
            return mid;
        } else if (nums[mid] < key) {
            lo = mid + 1;
        } else
            hi = mid - 1;
    }
    return -1;
}
```





