# 深入探索C++对象模型

# 第1章 关于对象

过程性语言：“数据”和“处理数据的操作（函数）”是分开来声明的。

C++在布局以及存取时间上主要的额外负担是由virtual引起的：

1. virtual function机制。用以支持一个运行期间绑定
2. virtual base class。用以实现“多次出现在继承体系中的base class，有一个单一被共享的实例”

## 1.1 C++对象模式

```c++
class Point{
public:
    Point(float xval);
    virtual ~Point();
    
    float x() const;
    static int PointCount();
    
protected:
	virtual ostream& print(ostream &os) const;
    
    float _x;
    static int _point_count;
};
```

### 简单对象模型

![image-20220903233605726](./深入探索C++对象模型.assets/image-20220903233605726.png)

简单布局模型中，成员本身不放在object中，放的是成员指针，这是为了解决成员有不同类型，因此需要不同存储空间的问题。

优点：降低了编译器的设计复杂度

缺点：空间和执行期效率低

### 表格驱动对象模型

![image-20220903234118949](./深入探索C++对象模型.assets/image-20220903234118949.png)

该模型将成员变量和成员函数分类，分为两个表格，而对象中包含着两个表格的**指针**

### C++对象模型

![image-20220903234625402](./深入探索C++对象模型.assets/image-20220903234625402.png)

虚函数实现机制：

1. 每个类产生一堆指向虚函数的指针，放在表格中，这个表格被称为virtual table(vtbl)
2. 每个对象被安插一个指针，指向相关的virtual table。通常这个指针被称为vptr。vptr的setting和resetting都由每一个类的构造函数、析构函数和赋值运算符自动完成

优点：空间效率和存取时间的效率

缺点：对象的非静态成员发生改变，即使应用程序没有改变，代码也得重新编译

## 1.2 关键词所带来的差异

主要讲述struct和class在古老C++编译器上产生的问题，内容古老，所以没有记录

## 1.3 对象的差异

三种程序设计范式

1. 过程模型。
2. 抽象数据类型模型。此模型所谓的“抽象”是和一组表达式（public接口）一起提供的，其运算定义仍然隐而未明
3. 面向对象模型。在此模型中有一些彼此相关的类型，通过一个抽象的基类封装起来。



C++多态：

1. 经由一组隐式的转化操作。例如把一个derived class指针转化为一个指向其public base type的指针
2. virtual function机制
3. 经由dynamic_cast和typeid运算符



C++对象内存构成

1. 非静态数据成员大小之和
2. 数据之间为了对齐所用的空间
3. 为了支持虚函数机制产生的空间



### 指针的类型

不同类型的指针之间的差异在于寻址出来的对象类型不同，而不是指针表示方法和内容。【这是实现多态的本质】



```c++
class ZooAnimal{
public:
    ZooAnimal();
    virtual ~ZooAnimal();
    virtual void rotate();
    
protected:
    int loc;
    String name;
};
```

其内存布局：

![image-20220904002530937](./深入探索C++对象模型.assets/image-20220904002530937.png)

加上子类

```c++
class Bear : public ZooAnimal{
public:
    Bear();
    ~Bear();
    
    void rotate();
    virtual void dance();
protected:
    enum Dances{...};			// 枚举值不占用对象空间，因为枚举属于类，而非对象
    Dances dances_know;
    int cell_block;
};
```

![image-20220904002918416](./深入探索C++对象模型.assets/image-20220904002918416.png)

# 第2章 构造函数语意学

## 2.1 默认构造函数的构造操作

### 带有默认构造函数的成员类

如果一个class没有任何constructor，但它内含一个member object，而后者有default constructor，那么这个类的implicit default constructor就是nontrivial，编译器需要为该类合成一个default constructor。不过这个合成操作只有在constructor真正需要被调用时才会发生。

```c++
class Foo {public: Foo(), Foo(int)...};
class Bar {public: Foo foo; char *str;};

void foo_bar()
{
    Bar bar;	// Bar::foo必须在此处初始化
    if(str){ } ...
}
```

合成的Bar默认构造函数能够调用Foo的**默认构造函数**来处理Bar::foo成员，但是不会对Bar::str进行初始化，对Bar::str进行初始化是程序员的责任。

```c++
class Dopey {public: Dopey;...};
class Sneezy {public: Sneezy(int); Sneezy();...};
class Bashful {public: Bashful();...};

class Snow_White{
public:
    Snow_White():sneezy(1024)
    {
        mumble = 2048;
    }
    Dopey dopey;
    Sneezy sneezy;
    Bashful bashful;
    
private:
    int mumble;
};

// 编译器扩张后的默认构造函数
// C++伪代码
Snow_White::Snow_White:sneezy(1024)
{
    dopey.Dopey::Dopey();
    sneezy.Sneezy::Sneezy(1024);
    bashful.Bashful::Bashful();
    
    // explicit user code
    mumble = 2048;
}
```

### 带有默认构造函数的基类

如果一个没有任何构造函数的类派生自一个有默认构造函数的基类，则这个派生类的默认构造函数会被视为nontrivial，并需要被合成，合成的构造函数调用上一层基类的**默认构造函数**（根据它们的声明顺序）

如果子类中写明了构造函数，但是没有默认构造函数，编译器不会合成默认构造函数，只会在已有的构造函数中进行扩展，方便调用父类的**默认构造函数**

### 带有虚函数的类

另外两种情况需要合成默认构造函数

1. 类声明（或继承）一个虚函数
2. 类派生自一个继承链条，其中有至少一个的虚基类

对于派生自虚类的类

1. 产生vtbl，然后在里面放入类的virtual function地址
2. 每一个对象中，放入一个额外的指针变量，指向与该类相关的vtbl的地址

### 带有一个虚基类的类

```c++
class X {public: int i;};
class A: public virtual X{public: int j;};
class B: public virtual X{puclic: double d;};
class C: public A, public B {public: int k;};

// 无法在编译时期决定出pa->X::i的位置
void foo(const A* pa) {pa->i = 1024;}
```

原先cfront的做法是子类中的每一个虚父类中插入一个指针，指针指向virtual base class，foo函数被改写如下

```c++
void foo(const A* pa){pa->__vbcX->i = 1024;}
```

其中__vbcX表示编译器所产生的指针，指向virtual base class X。

经常会有2个误解：

1. 任何class如果没有定义默认构造函数，就会被合成出一个【写了构造函数，编译器不会构造默认构造函数】
2. 编译器合成出来的默认构造函数会显式设定类内的每一个数据成员默认值【编译器合成的构造函数只会调用父类或者成员类的默认构造函数，成员变量不会被初始化】



## 2.1 拷贝构造函数的构造操作

### Default Memberwise Initialization

如果类没有提供拷贝构造函数，使用一个对象初始化另一个对象时，都是将每一个内建的或派生的数据成员的值，从一个对象拷贝到另一个对象上。
