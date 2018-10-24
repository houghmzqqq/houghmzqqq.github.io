---
title: 'LeetCode:23.Merge k Sorted Lists'
date: 2017-08-21 15:23:24
tags:
    - LeetCode-频度5
    - 算法
    - Hard
---


## 思路
原题：Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

大意：合并k个已排序的链表并返回一个合并后的链表。注意时间复杂度。

<!-- more -->
## 解法一
分析：最简单的方法是遍历每一个链表的当前节点，并取出最小的一个数加入result（结果链表）中，取出数字的链表当前节点后移。
该方法的时间复杂度较高。

代码：
```
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        ListNode head = new ListNode(0);
        ListNode result = head;
        
        //链表数组转化为ArrayList,方便对链表数量进行增删
        List<ListNode> nodeList = new ArrayList(Arrays.asList(lists));
        
        //过滤空链表
        for(int i=0;i<nodeList.size();i++){
            if(nodeList.get(i)==null){
                nodeList.remove(i);
                i--;
            } 
        }
        if(nodeList.size()==1) return nodeList.get(0);
        
        while(nodeList.size()>1){
            Map<Integer,Integer> map = new HashMap();
            int min = nodeList.get(0).val;
            map.put(min,0);

            for(int i=1;i<nodeList.size();i++){
                if(nodeList.get(i).val<min){
                    min=nodeList.get(i).val;
                    map.put(min,i);
                } 
            }
            result.next = new ListNode(min);
            //result.next.val = min;
            result = result.next;

            int index = map.get(min);
            if(nodeList.get(index).next==null)
                nodeList.remove(index);
            else
                nodeList.set(index,nodeList.get(index).next);

            if(nodeList.size()==1){
                result.next = nodeList.get(0);
            }
        }
        
        return head.next;
    }
}
```


## 方法二：基于“二分”思想的归并排序
分析：进行循环合并链表，对于k个链表，总节点数为n，每一次循环使链表的数量减少为(k+1)/2。
在每次循环中：lists[0]与lists[(k+1)/2]合并，lists[1]与lists[(k+1)/2+1]合并，lists[2]与lists[(k+1)/2+2]合并，类推......

代码：
```
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        int len = lists.length;
        
        if(lists==null || len==0) return null;
        if(len==1) return lists[0];
        
        //进行归并排序
        while(len>1){
            int mid = (len+1)/2;
            
            for(int i=0;i<len/2;i++){
                lists[i] = merge(lists[i],lists[i+mid]);
            }
            len = mid;
        }
        return lists[0];
    }
    
    //对lists[i],lists[(k+1)/2+i]进行合并
    public ListNode merge(ListNode node01,ListNode node02){
        
        ListNode head = new ListNode(0);
        ListNode result = head;
        
        while(node01!=null && node02!=null){
            if(node01.val<=node02.val){
                result.next = node01;
                node01 = node01.next;
            }
            else{
                result.next = node02;
                node02 = node02.next;
            }
            result = result.next;
        }
        if(node01!=null)
            result.next = node01;
        else
            result.next = node02;
        
        return head.next;
    }
}
```


## 方法三：基于优先级队列的堆排序
分析：使用Java __api__中的优先级队列（Priority Queue），将所有链表的当前节点offer进堆中，PriorityQueue会根据实现了Comparable接口的实现类中的compareTo方法，将符合条件的元素放在堆顶，我们将堆顶的元素逐个取出放入结果链表中即可。

代码：
```
//实现Comparable接口
public class ListNode implements Comparable<ListNode>{
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
	
	//PriorityQueue根据该方法决定放在堆顶的元素
    public int compareTo(ListNode l1){
    	return val-l1.val;
    }
}

class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        
        ListNode head = new ListNode(0);
        ListNode result = head;
        
        Queue<ListNode> queue = new PriorityQueue(Arrays.asList(lists));
        
        if(lists==null || lists.length==0) return null;
        if(lists.length==1) return lists[0];
        
        while(queue.size()>0){
            ListNode l1 = queue.poll();
            result.next = l1;
            if(l1.next!=null) queue.offer(l1.next);
        }
        
        return head.next;
    }
}
```