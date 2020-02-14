### 描述

设计一个支持 push，pop，top 操作，并能在常数时间内检索到最小元素的栈。

push(x) -- 将元素 x 推入栈中。
pop() -- 删除栈顶的元素。
top() -- 获取栈顶元素。
getMin() -- 检索栈中的最小元素。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/min-stack

### 思路

其实这题的话，如果单纯考虑这样的需求，我们可能第一时间想到的是堆。

这题我们还是思考一下，其实也不难。毕竟他的难度评判认定为简单。

我们可以可以这么做：

我们用两个标准栈，我们用一个栈来正常压栈，弹栈，另外一个栈，我们则需要维护一下大小关系了。假设A，B两个栈，A栈我们用来维护正常压栈弹栈，B栈则是最大的元素在栈底部，最小的元素在栈顶，那么我们就可以常数时间拿到最小元素了。我们这里用伪代码的形式写一下：

```
元素 E 压栈
A 压栈
如果 B 栈是空 或者 B 栈栈顶元素比 E 大
B 压栈 
否则
B 压栈一个原来的栈顶元素
```

这样，我们就可以维护一个最小栈了。

### 代码

代码不会很复杂，见下面：

```java
class MinStack {
    private Stack<Integer> stack; // A 栈
    private Stack<Integer> helper; // B 栈
    
    /** initialize your data structure here. */
    public MinStack() {
        stack = new Stack();
        helper = new Stack();
    }
    
    public void push(int x) {
        stack.add(x);
        if (helper.isEmpty() || helper.peek() >= x) {
            helper.add(x);
        }else {
            helper.add(helper.peek());
        }
    }
    
    public void pop() {
        if (!stack.isEmpty()) {
            stack.pop();
            helper.pop();
        }
    }
    
    public int top() {
        if (!stack.isEmpty()) {
            return stack.peek();
        }
        throw new RuntimeException("");
    }
    
    public int getMin() {
        if (!helper.isEmpty()) {
            return helper.peek();
        }
        throw new RuntimeException("");
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```

多思考，多学习。