### 描述

用栈来实现队列。

### 思路

这题应该是最为经典的题目了。

我们其实思考一下，栈的特性是什么？

- 先进后出

队列的特性呢？

- 先进先出

好了，那我们可以这么做，拿出两个栈，A，B。

假设我们有这么一堆数，`[1,2,3,4,5]`，他们需要入队，那么出队的顺序应该也是`[1,2,3,4,5]`。

OK，我们正常把数字丢进A栈里面。那我们如何实现出队呢？

**把A里面的元素全部倒入B，把栈B的栈顶元素返回，再把B栈的东西倒回去。**

OK，就这么简单。

那么我们来看代码：

```java
class MyQueue {
    
    private Stack<Integer> stack1;
    private Stack<Integer> stack2;

    /** Initialize your data structure here. */
    public MyQueue() {
        this.stack1 = new Stack();
        this.stack2 = new Stack();
    }
    
    /** Push element x to the back of queue. */
    public void push(int x) {
        if (stack1.empty()) stack1.push(x);
        else {
        	while (!stack1.empty()) stack2.push(stack1.pop());
        	stack1.push(x);
        	while(!stack2.empty()) stack1.push(stack2.pop());
        }
        
    }
    
    /** Removes the element from in front of queue and returns that element. */
    public int pop() {
        if (!stack1.empty()) 
            return stack1.pop();
        return Integer.MIN_VALUE;
    }
    
    /** Get the front element. */
    public int peek() {
        if (!stack1.empty()) return stack1.peek();
        return Integer.MIN_VALUE;
    }
    
    /** Returns whether the queue is empty. */
    public boolean empty() {
        return stack1.empty();
    }
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = new MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * boolean param_4 = obj.empty();
 */
```

代码风格有点差，但是这么短几句不影响阅读，就这样子。

重点要多理解。