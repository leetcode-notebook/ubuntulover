### BFS专题学习

#### 介绍

BFS 叫广度优先搜索（bread first search), 是一种常见的搜索算法。广度优先搜索的例子和解释有很多， 这里不再详细赘述。

这里想介绍一下bfs的相关模版，然后会有相关的题解练习。

BFS要用到队列的特性来完成。队列的常见特性是先进先出，利用这个特性，我们可以很好的完成搜索。

```java
Queue<T> q = new LinkedList();
q.add(start_position);
while (!q.isEmpty()) {
  T t = q.poll();
  if (something is connected to t) {
    q.add(something);
  }
}
```

这就是bfs的模版了。我们搜索的重点， 是入队条件的判定，我们到底要把什么样的东西入队呢？我们可以好好思考。



#### 做题记录

话不多说， 上code：

- [leetcode 130]()