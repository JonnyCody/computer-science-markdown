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
