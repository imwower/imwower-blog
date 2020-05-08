---
title: 堆排序合并N个有序链表
date: 2020-05-08 11:42:24
tags: [堆排序,算法,合并有序链表]
---

#### 合并N个有序链表：

###### 思路：
1. 将每个链表的头节点组成一个包含N个元素的数组 `arr`，`length`为`N`；
2. 用数组构造小顶堆，堆顶的最小值，就是最终`result`的第一个节点；
  2.1 如果`length`为1，说明只剩下一个链表了，将该链表剩余部分接到`result`，退出递归；
3. 取走堆顶的元素，也就是`arr[0]`；将`arr[0]`的链表指向下一个节点；
  3.1 如果下一个节点为空，说明当前的链表结束了，需要将它从开头交换到末尾，然后将`length-1`；
4. 因为数组变化了，递归执行第2步；

###### 时间复杂度：
1. 由N个链表组成数组，进行小顶堆交换，时间复杂度为：`logN`；
2. 对于每个链表，假设长度都是K，则都需要遍历，时间复杂度为：`K`；
所以总的时间复杂度为：`O(K*logN)`

###### 图解：
1. 将每个链表的头节点组成一个包含N个元素的数组 arr；
![merge_linked_list_1.jpg](/images/merge_linked_list/merge_linked_list_1.jpg)

2. 用数组构造小顶堆，堆顶的最小值，就是最终结果链表的第一个节点；
  2.1 如果`length`为1，说明只剩下一个链表了，将该链表剩余部分接到`result`，退出递归
![merge_linked_list_2.jpg](/images/merge_linked_list/merge_linked_list_2.jpg)

3. 取走堆顶的元素，也就是arr[0]；将arr[0]的链表指向下一个节点；
  3.1 如果下一个节点为空，说明当前的链表结束了，需要将它从开头交换到末尾，然后将数组的长度-1；
![merge_linked_list_3.jpg](/images/merge_linked_list/merge_linked_list_3.jpg)

4. 因为数组变化了，递归执行第2步；
![merge_linked_list_4.jpg](/images/merge_linked_list/merge_linked_list_4.jpg)

###### 示例代码：
``` java
public class Test {
    public static void main(String[] args) {
        ListNode l1 = ListNode.getInstance(new int[]{3, 4, 5, 6, 28});
        ListNode l2 = ListNode.getInstance(new int[]{1, 5, 6, 8, 20});
        ListNode l3 = ListNode.getInstance(new int[]{2, 5, 6, 20});
        ListNode l4 = ListNode.getInstance(new int[]{0, 2, 10, 12});
        ListNode[] list = {l1, l2, l3, l4};
        //最终结果：
        ListNode result = new ListNode(0);
        //指向result的指针；因为每次取新的节点时，c需要向后位移；而result为最开始的头节点；
        ListNode c = result;
        //堆排序合并：
        heap(list, c, list.length);
        //去掉头节点：
        System.out.println(result.next.toString());
        //0 -> 1 -> 2 -> 2 -> 3 -> 4 -> 5 -> 5 -> 5 -> 6 -> 6 -> 6 -> 8 -> 10 -> 12 -> 20 -> 20 -> 28s
    }

    //length不是arr的长度；因为递归的过程中，某个链表可能先结束了；
    //将先结束的链表交换到数组末尾，然后把length-1；
    public static void heap(ListNode[] arr, ListNode c, int length) {
        //只剩下一个链表：
        if (length == 1) {
            //将剩余部分接到result中：
            c.next = arr[0];
            //退出递归：
            return;
        }
        //构造小顶堆
        for (int i = length / 2 - 1; i >= 0; i--) {
            adjust(arr, i, length);
        }
        //数组第一个元素就是最小的元素，取出来，放入结果链表：
        c.next = arr[0];
        //将结果链表的指针后移：
        c = c.next;
        //将数组第一个元素的指针后移：
        arr[0] = arr[0].next;
        //如果链表为空，说明该链表提前结束了：
        if (arr[0] == null) {
            //将先结束的链表交换到数组末尾
            arr[0] = arr[length - 1];
            //然后把length-1；
            length--;
        }
        heap(arr, c, length);
    }

    //构造小顶堆，并交换元素：
    public static void adjust(ListNode[] arr, int i, int length) {
        for (int k = i * 2 + 1; k < length; k = k * 2 + 1) {
            if (k + 1 < length && arr[k].val > arr[k + 1].val) {
                k++;
            }
            if (arr[i].val > arr[k].val) {
                ListNode value = arr[i];
                arr[i] = arr[k];
                arr[k] = value;
                i = k;
            }
        }
    }
}
```

```java
public class ListNode {
    public int val;
    public ListNode next;

    public ListNode(int val) {
        this.val = val;
    }

    public static ListNode getInstance(int[] values) {
        ListNode t = new ListNode(0);
        ListNode n = t;
        for (int i = 0; i < values.length; i++) {
            n.next = new ListNode(values[i]);
            n = n.next;
        }
        return t.next;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder().append(val);
        ListNode next = this.next;
        while (next != null) {
            sb.append(" -> ").append(next.val);
            next = next.next;
        }
        return sb.toString();
    }
}
```