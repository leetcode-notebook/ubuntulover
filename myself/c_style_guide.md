## C 语言风格指导

这份C语言风格指导来自于新南威尔士大学CSE学院的本科课程COMP1511的C语言风格指导。

[COMP1511 style guide](https://cgi.cse.unsw.edu.au/~cs1511/19T1/resources/style_guide.html#code-structure)



本文将会介绍一些推荐的和不推荐的C语言风格。

### 推荐的代码布局

#### 头注释

所有的程序都应该有头注释。

哪怕是非常小的功能点的代码，你的代码至少应该包含：

- 作者的名字
- 日期
- 一个简短的描述，描述你的程序的功能

代码例子：

```c
//
// COMP1511 Lab 10 Exercise - Turing's test
//
// Pretend to be a human in text-based conversation.
// https://en.wikipedia.org/wiki/Turing_test
//
// Authors:
// Grace Hopper (z1234567@unsw.edu.au)
// Ada Lovelace (z5000000@unsw.edu.au)
//
//
//
// Written: 19/03/2019
//

#include <stdio.h>
#include <string.h>
#include <math.h>

#define MEANING_OF_LIFE 42


// function prototypes would go here


int main(void) {
    printf("Hello!\n");
    return 0;
}


// function definitions would go here
```

#### 括号和缩进

函数体，循环，if-else应该使用_完整的括号_。

代码实例：

```c
int someFunction(int a, int b) { // 推荐这里有个空格
    if (a > b) {
        // do something
    } else if (b < a) {
        // do something
    } else {
        // do something
    }

    int i = 0;
    while (i < 0) {
        // do something
        i++;
    }
}
```

下面是糟糕的代码风格：

```c
// BAD - 在{}之间的代码应该被缩进

if (i % 2 == 0) {
printf("even");
} else {
printf("odd");
}


// BAD - {}应该有换行。

while (i < 0) { i++; }


// BAD - if 和while要有{}

while (i < 0)
    i++;

// BAD - 应该有缩进和对齐。

if (a > b) {
    // do something
    } else if (b < a) {
        // do something
        } else {
            // do something
            }
```

### 缩进

在任何的括号之间，应该有**4个空格**的缩进。

在以下关键词之后需要有**一个空格**：

`if `, `while`,`for`, `return`

在下列二元操作符有空格：

`=`， `+`，`-`，`*`，`<`,`>`,`/`,`%`,`<=`,`>=`,`==`,`!=`

不需要在前置单元操作符上使用空格：

`& `，`*`，`！`

同样的，下面也不需要空格：

`++`，`--`

`.`，`->`

代码实例

```c

// 下面是正确的
if (a + b > c) {
    i++;
    curr = curr->next;
}


// 下面是不正确的:
if(a+b>c) {
    i ++;
    curr = curr -> next;
}
```

**一行最多有80个字符（包括空格）**

### 常量

使用 `#define`来定义常量

所有使用`#define`的常量，需要用大写字母和下划线来表示。

代码实例：

```c
#define PI 3.14
#define DAYS_OF_WEEK 7

// ...
int array[DAYS_OF_WEEK];
int i = 0;
while (i < DAYS_OF_WEEK) {
    array[i] = i;
    i++;
}
```

### 变量

变量名应该尽可能的描述性。

应该遵从`snake_case`或者`camelCase`的方式来命名。

不推荐使用全局变量，不允许使用静态变量

全局变量是定义在函数外部的变量，这会使得代码难以阅读和维护。

使用`static`关键字声明的变量是不允许的。但是可以使用它来声明函数，来实现隐藏和封装。

### 函数

把大量重复，相似的代码重构成函数，这样可以提高你的代码能力和代码质量。

不允许函数定义和函数实现写在一起。

函数注释应该使用多行注释， 指定参数类型，返回值。

代码实例：

```c
/**
* author: Alex
* date: 2020/01/01
* args: none
* return: none
* just demostrate how to define a function.
*/
void func(void);
/**
* author: Alex
* date: 2020/01/01
* args: int a, int b
* return: int
* give 2 integer's sum.
*/
int sum(int, int);

// ...
void func(void) {
    printf("hello world!\n");
}

int sum(int a, int b) {
    return a + b;
}
```



### 推荐

- 避免使用switch， 使用if - else  if - else if
- 不要使用goto
- 不要使用 `：？`这样的三元运算符
- 避免使用 `do {} while(condition)`, 使用while循环来替代
- 不要使用联合体。（union）
- 避免指针的算数操作
- 避免类型转换。