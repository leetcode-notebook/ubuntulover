### 描述

根据逆波兰表示法，求表达式的值。

### 思路

这题基本没啥好说的。。。。

就是简单的碰到运算符，弹出两个元素进行操作压回去，最后栈里只有一个元素，就是我们要的结果。

```java
class Solution {
    public int evalRPN(String[] tokens) {
        Stack<Integer> stack = new Stack();
        int ans = 0;
        for (int i = 0; i < tokens.length; i++) {
            // if they are numbers.
            // lets deal with numbers first.
            try {
                int num = Integer.parseInt(tokens[i]);
                stack.push(num);
            }catch (Exception e) {
                // so if you are operators.
                String op = tokens[i];
                int x = stack.pop();
                int y = stack.pop();
                if (op.equals("+")) {
                    stack.push(x + y);
                }else if (op.equals("-")) {
                    stack.push(y - x);
                } else if (op.equals("*")) {
                    stack.push(x * y);
                }else if (op.equals("/")) {
                    stack.push(y / x);
                }
            }
            
        }
        if (stack.size() == 1) {
            ans = stack.pop();
        }
        return ans;
    }
}

```

