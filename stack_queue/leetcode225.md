### 描述

用队列实现栈

### 思路

上次我们讲了如何用栈来模拟队列，那么这里研究如何用队列模拟栈。

其实也不难，我们思考一下本质东西：

栈的特点是什么？

- 先进后出

队列的特点是什么？

- 先进先出

这个题还是和[leetcode 232 栈模拟队列](stack_queue/leetcode232.md)类似，是最基本的加强对这两个东西理解的。

那我们怎么实现呢？

类似的，我们也用到两个队列。A，B。

入栈操作：

正常加队列A。

弹栈操作：

A出队一个元素，这是我们要返回的结果。然后，**交换A，B。**

代码：

```java
class MyStack {

    Queue<Integer> q1 = new  LinkedList();
    Queue<Integer> q2 = new LinkedList();
    private int top;
    /** Initialize your data structure here. */
    public MyStack() {
        
    }
    
    /** Push element x onto stack. */
    public void push(int x) {
        q1.add(x);
        top = x;
    }
    
    /** Removes the element on top of the stack and returns that element. */
    public int pop() {
        while (q1.size() > 1) {
            top = q1.remove();
            q2.add(top);
        }
        int ret = q1.remove();
        Queue<Integer> temp = q1;
        q1 = q2;
        q2 = temp;
        return ret;
        
    }
    
    /** Get the top element. */
    public int top() {
        return top;
        
    }
    
    /** Returns whether the stack is empty. */
    public boolean empty() {
        return q1.isEmpty();
        
    }
}

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * boolean param_4 = obj.empty();
 */
```

重点还是在于理解。