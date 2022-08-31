# Qt5开发及实例

# 第1章 Qt概述

## 1.3 Qt5开发步骤及实例：概念解析

### 1.3.1 信号和槽机制

Qt提供了信号和槽机制用于完成界面操作的响应，是完成两个任意Qt对象之间的通信机制。

信号会在某个特定情况或动作下触发，槽是等同于接收并处理信号的函数。

#### 信号和槽的连接方式

1. 一个信号和一个槽相连

   ``````c++
   connect(Object1, SIGNAL(signal1), Object2, SIGNAL(slot1));

2. 一个信号对多个槽

   ```c++
   connect(Object1, SIGNAL(signal2), Object2, SIGNAL(slot2));
   connect(Object1, SIGNAL(signal2), Object3, SIGNAL(slot1));
   ```

3. 多个信号对一个槽

   ```c++
   connect(Object1, SIGNAL(signal1), Object2, SIGNAL(signal1));
   connect(Object3, SIGNAL(signal1), Object2, SIGNAL(signal1));
   ```

SIGNAL和SLOT是Qt的定义的两个宏，他们返回其参数的C语言风格的**字符串**(const char*)，所以一下两个语句相同

```c++
connect(button, SIGNAL(clicked()), this, SIGNAL(showArea()));
connect(button, "clicked()", this, "showArea()");
```



# 第2章 Qt5模板库、工具类及控件

## 2.1 字符串类

## 2.2 容器类

存储在Qt容器中的数据必须是**可赋值的数据类型**，也就是说这种类型必须提供**一个默认的构造函数、一个复制构造函数和一个赋值操作运算符**。

Qt的**QObject及其他的子类**（如QWidget和Qdialog等）是**不能够**存储在容器中的，因为它们没有**复制构造函数**和**赋值操作运算符**。替代方案是使用其指针。

### 2.2.1 QList类、QLinkedList类和QVector类

![image-20220824155755646](E:\Computer Science\个人笔记\C++\Qt5开发及实例.asset\image-20220824155755646.png)

需要注意QList的查找效率

#### QList类

QList<T>维护了一个**指针数组**，该数组存储的指针指向QList<T>存储的列表项的内容。

- 如果T是一个指针类型或者指针大小的基本类型（即该基本类型占有的字节数和指针类型占有的字节数相同），QList<T>会将数值直接存储在它的数组中。
- 如果QList<T>存储对象的指针，则该指针指向实际存储的对象。

