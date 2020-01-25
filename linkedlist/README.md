### LinkedList的概念
单链表中的每个结点不仅包含值，还包含链接到下一个结点的引用字段。通过这种方式，单链表将所有结点按顺序组织起来。

这是C语言的定义：
```c
typedef _node Linkedlist;
struct _node {
	int val; // data here
	struct _node* next;
};
```
Java的写法类似：
```java
public class Linkedlist {
	int val;
	Linkedlist next;
	// constructor.
	Linkedlist(int x) {this.val = x;}
}
```

绝大多数的时候，我们会用头结点来表示整个列表。一个成语可以很好的解释这个：**顺藤摸瓜**

### LinkedList的相关操作
我们设计出一个数据结构，势必要涉及到跟他相关的一些操作：增删改查。
我们首先来看如何往单链表里面添加元素。
#### 添加
1. 使用给定的值去初始化新的节点cur;

![before](/home/xuetao/.config/Typora/typora-user-images/1578974211336.png)

2. 将cur的next字段连接到原来的下一个节点next当中：![linking](/home/xuetao/.config/Typora/typora-user-images/1578974292368.png)
3. 完成。![finish](/home/xuetao/.config/Typora/typora-user-images/1578974316048.png)

插入操作是非常高效的，可以在O(1)的时间复杂度中把新节点插入进去，这是非常高效的。

#### 删除

如果想要删除单链表上的节点， 也非常简单，就是删除过程的逆过程。这里不再赘述。

###　拓展
网上链表相关的代码有比较多，这里可以考虑使用链表来实现队列/栈。
队列：一种先进先出的结构
栈：　一种先进后出的结构

可以看到，栈和队列非常相似。
好了，还是程序猿那句老话，**talk is cheap, show me the code.**
我们来设计一个接口，又可以当栈用，也可以当队列用。栈叫Stack， 队列叫Queue， 那么我们称之为Quack。

---
Quack.h的内容：
```c
#include<stdio.h>
#include<stdlib.h>

typedef struct _node* Quack;


Quack createQuack(void);    // create and return Quack
Quack destroyQuack(Quack);  // remove the Quack 
void  push(int, Quack);     // put the given integer onto the top of the quack
void  qush(int, Quack);     // put the given integer onto the bottom of the quack
int   pop(Quack);           // pop and return the top element on the quack
int   isEmptyQuack(Quack);  // return 1 is Quack is empty, else 0
void  makeEmptyQuack(Quack);// remove all the elements on Quack
void  showQuack(Quack);     // print the contents of Quack, from the top down

```

在头文件里面写好了函数定义，我们来考虑实现功能。

Quack.c
```c
#include "Quack.h"
#include <limits.h> // for INT_MAX macro.

#define MARKERDATA INT_MAX // dummy data

struct _node {
   int data;
   struct node *next;
};

Quack createQuack(void) {
   Quack marker;
   marker = malloc(sizeof(struct node));
   if (marker == NULL) {
      fprintf (stderr, "createQuack: no memory, aborting\n");
      exit(1);
   }
   marker->data = MARKERDATA; // should never be used
   marker->next = NULL;
   return marker;
}

void push(int data, Quack qs) {
   Quack newnode;
   if (qs == NULL) {
      fprintf(stderr, "push: quack not initialised\n");
   }
   else {
      newnode = malloc(sizeof(struct node));
      if (newnode == NULL) {
         fprintf(stderr, "push: no memory, aborting\n");
         exit(1);
      }
      newnode->data = data;
      newnode->next = qs->next;
      qs->next = newnode;
   }
   return;
}

void qush(int data, Quack qs) {

   // code not shown

   return;
}

int pop(Quack qs) {
   int retval = 0;
   if (qs == NULL) {
      fprintf(stderr, "pop: quack not initialised\n");
   }
   else {
      if (isEmptyQuack(qs)) {
         fprintf(stderr, "pop: quack underflow\n");
      }
      else {
         Quack topnode = qs->next; // top dummy node is always there
         retval = topnode->data;
         qs->next = topnode->next;
         free(topnode);
      }
   }
   return retval;
}

Quack destroyQuack(Quack qs) { // remove the Quack  and returns NULL
   if (qs == NULL) {
      fprintf(stderr, "destroyQuack: quack not initialised\n");
   }
   else {
      Quack ptr = qs->next;
      while (ptr != NULL) {
         Quack tmp = ptr->next;
         free(ptr);
         ptr = tmp;
      }
      free(qs);
   }
   return NULL;
}

void makeEmptyQuack(Quack qs) {
   if (qs == NULL)
      fprintf(stderr, "makeEmptyQuack: quack not initialised\n");
   else {
      while (!isEmptyQuack(qs)) {
         pop(qs);
      }
   }
   return;
}

int isEmptyQuack(Quack qs) {
   int empty = 0;
   if (qs == NULL) {
      fprintf(stderr, "isEmptyQuack: quack not initialised\n");
   }
   else {
      empty = qs->next == NULL;
   }
   return empty;
}

void showQuack(Quack qs) {
   if (qs == NULL) {
      fprintf(stderr, "showQuack: quack not initialised\n");
   }
   else {
      printf("Quack: ");
      if (qs->next == NULL) {
         printf("<< >>\n");
      }
      else {
         printf("<<");                 // start with <<
         qs = qs->next;                // step over the marker link
         while (qs->next != NULL) {
            printf("%d, ", qs->data);  // print each element
            qs = qs->next;
         }
         printf("%d>>\n", qs->data);    // last element ends with >>
      }
   }
   return;
}
```

那么我们来编写程序来使用，我们来反转字符串。
revarg.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "quack.h"

int main(int argc, char *argv[]) {
  Quack s = NULL;

  if (argc >= 2) {
     char *inputc = argv[1];
     s = createQuack();
     while (*inputc != '\0') {
        push(*inputc++, s);
     }
     while (!isEmptyQuack(s)) {
        printf("%c", pop(s));
     }
     putchar('\n');
  }
  return EXIT_SUCCESS;
}
```
编译代码：
```
gcc Quack.c revarg.c 
```
运行：
```
./a.out 0123456789
>> 9876543210
```

[LeetCode876 链表中间节点题解](linkedlist/Leetcode876MiddleOftheLinkedList.md)

[Leetcode19 删除倒数n个节点题解](linkedlist/leetcode19.md)

[Leetcode206 反转链表题解](linkedlist/leetcode206.md)

[Leetcode61 旋转链表题解](linkedlist/leetcode61.md)

[Leetcode160 相交链表题解](linkedlist/leetcode160.md)

[Leetcode147 链表插入排序题解](linkedlist/leetcode147.md)

[Leetcode86 分割链表题解](linkedlist/leetcode86.md)