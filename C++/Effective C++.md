# Effective C++

# 1 让自己习惯C++

## 条款01 视C++为一个语言联邦

C++主要由四个次语言构成：

- C。C语言局限：没有模板、没有异常、没有重载。
- Object-Oriented C++。这部分就是C with Classes，主要包含classes（构造和析构），封装、继承、多态和virtual函数。
- Template C++。带来了模板元编程。
- STL。

C++高效编程取决于使用C++的哪一部分。

## 条款02 尽量以const,enum,inline替换#define

使用#define不足

- 使用常量出现错误时，错误信息不会出现记号名称，因为在编译之前已经被替换成常量了。
- #define没有作用域，无法称为class的专属常量，所以无法提供封装性。

对于形似函数的宏，最好改用inline函数替换#define

## 条款03 尽可能使用const

const修饰范围

- classes外部修饰global或namespace作用域中的常量
- 修饰文件、函数、或区块作用域中被声明为static的对象
- classes内部的static和non-static成员变量
- 可以修饰指针，指针所指物是否为const

const出现在星号左边表示被指物是常量，如果出现在星号右边表示指针自身是常量

成员函数设置为const目的是为了确认成员函数可作用于const对象

改善C++程序效率的一个根本办法是以*pass by reference to const*方式传递对象，此技术可行的前提是有const成员函数可用来处理取得的const对象

**bitwise const**认为成员函数只有在不更改对象的任何成员变量（static除外）时才可以说是const

**logic constness**一个const成员函数可以修改它所处理的对象的某些bits，但只有在客户端侦测不出的情况下才得如此

如果在const对象中需要更改成员变量，该变量可以声明为**mutable**

- 将某些东西声明为cons可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体
- 编译器强制实施bitwise constness，但你编写程雪时应该使用“概念上的常量性”（conceptual constness）【个人理解，概念上常量性即逻辑常量性，对外表现常量性即可，内部可以进行变动】
- 当const和non-const成员函数有着实质的实现时，令non-const版本调用const版本可避免代码重复

## 条款04 确定对象被使用前已先被初始化

永远在使用对象之前将它初始化，对于无任何成员的内置类型，必须手工完成此事，如果不是内置类型，则需要在构造函数上进行初始化

只调用一次的copy构造函数比先调用default构造函数再调用拷贝构造函数操作符高效

C++对于“定义于不同编译单元内的non-local static对象”的初始化次序无明确定义

使用单例模式确保不同文件的static成员在调用前被初始化

```c++
FileSystem& tfs()
{
    static FileSystem fs;
    return fs;
}

class Directory{...};
Dirctory::Dictory(params)
{
    ...
    std::size_t disks = tfs().numDists();
    ...
}
```

- 内置型对象进行手工初始化，因为C++不保证初始化它们
- 构造函数最好使用成员初始化列表，不要在构造函数本体内使用赋值操作。初始值列表列出的成员变量，其排列次序应该和它们在class中的声明次序相同
- 为免除“跨编译单元之初始化次序”问题，应该以local static对象替换non-local static对象

# 2 构造/析构/赋值运算

## 条款05 了解C++默默编写并调用哪些函数

对于一个空类，编译器会声明一个default构造函数、copy构造函数、copy assignment操作符和一个析构函数，只有当这些函数被调用，才会被创造出来。

编译器合成函数所做的工作

- 构造和析构函数：调用基类和non-static成员变量的构造函数和析构函数，编译器产生的析构函数是non-trivial
- copy构造函数和copy assignment操作符，编译器创建版本只是单纯的将来源对象的每一个non-static成员变量拷贝到目标对象
- 如果声明了一个构造函数，编译器不会再创建default构造函数



- 编译器可以暗自为class创建default构造函数、copy构造函数、copy assignment操作符，以及析构函数

## 条款06 若不想使用编译器自动生成的函数，就应该明确拒绝

阻止编译器生成函数一般有两种方法：

- 函数声明为私有，但是不定义：

```c++
class HomeForSale
{
public:
    ...
private:
    ...
    HomeForSale(const HomeForSale&);	// 只有声明
    HomeForSale& operator=(const HomeForSale&);
};
```



- 基类中函数声明为私有，子类私有继承

```c++
class Uncopyable
{
protected:
    Uncopyable(){}
    ~Uncopyable(){}
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale : private Uncopyable
{
    ...
};
```



- 为驳回编译器自动提供的机能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种方法。

## 条款07 为多态基类声明virtual析构函数

任何class只要带有virtual函数都几乎确定应该也有一个virtual析构函数。

- 带多态性质的基类应该声明一个virtual析构函数。如果class带有任何virtual函数，则析构函数也应该是virtual
- 类的设计目的如果不是作为基类使用的，或不是为了具备多态性，就不该声明为virtual析构函数。否则会给类带来多余的空间，不利于移植。

## 条款08 别让异常逃离析构函数

- 析构函数绝对不要吐出异常，如果一个析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作

## 条款09 绝不在构造和析构函数中调用virtual函数

无法使用virtual函数从base classes向下调用，在构造期间，可以让derived classes将必要的构造信息向上传递至base classes构造函数

- 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class

## 条款10 令operate=返回一个reference to *this

- 令赋值操作符返回一个reference to *this，因为可以进行连续赋值动作


## 条款11 在operate=中处理自我赋值



```c++
Widget& Widget::operator=(const Widget& rhs)
{
    if(this == &rhs) return *this;
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}

Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap* pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```

第一个例子无法做到异常安全，当new动作抛出异常，则pb指向一块被删除的Bitmap，第二个例子则可以实现异常安全，可以在new之后才进行delete



```c++
Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs);
    swap(temp);
    return *this;
};
```

使用copy and swap技术，这是一个常见而且够好的operator=撰写方法

- 确保当对象自我赋值时operator=有良好的行为，其中技术包括比较“来源对象”和“目标对象”的地址、静心周到的语句顺序、以及copy-and-swap
- 确保任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确

## 条款12 复制对象时勿忘其每一个成分

如果为class添加一个成员变量，必须同时修改copying函数，也需要修改class的所有构造函数以及任何非标准形式的operator=

既不能用copy assignment操作符调用copy构造函数，也不能令copy构造函数调用copy assignment操作符，如果二者有相近的代码，则应该将相近的代码建立一个新函数给两者调用

- copying函数应该确保复制“对象内的所有成员变量”以及“所有base class成分”
- 不要尝试以某个copying函数实现另一个copying函数，应该将共同机能放进第三个函数中，并由两个copying函数共同调用

# 3 资源管理

## 条款13 以对象管理资源

把资源放进对象内，可依赖C++的“析构函数自动调用机制”确保资源被释放。

以对象管理资源两个关键想法

- 获得资源后立即放进管理对象内。可以放进智能指针内部
- 管理对象运用析构函数确保资源被释放。

auto_ptr和tr1::shared_ptr两者都在其析构函数内做delete而不是delete[]动作。

- 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源
- 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr。前者通常是较佳选择，因为其copy行为比较直观。若选择auto_ptr，复制动作会使它（被复制物）指向null

## 条款14 在资源管理类中小心copying行为

- 复制RAII对象必须一并复制它所管理的资源，所以资源copying行为决定RAII对象copying行为
- 普遍而常见的RAII class copying行为是：抑制copying、施行引用计数法。不过其他行为也能被实现

## 条款15 在资源管理类中提供对原始资源的访问

- APIs往往要求访问原始资源，所以每一个RAII class应该提供一个“取得其所管理之资源”的办法
- 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

## 条款16 成对使用new和delete时要采用相同形式

- 如果你在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[]，一定不要在相应的delete表达式中使用[]。

## 条款17 以独立语句将newed对象置入智能指针

```c++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());

std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

第一个函数调用如果在new Widget和tr1::shared_ptr构造函数之间抛出异常，则会被异常干扰。

第二个调用，将newed对象放入独立语句，避免异常干扰。

- 以独立语句将newed对象存储于智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

# 4 设计与声明

## 条款18 让接口容易被正确使用，不易被误用

```c++
class Data
{
public:
    Data(int month, int day, int year);
};
```

这个接口容易被误用，因为 Data d(30, 3, 1995); Data d(3, 30, 1995); 为了避免被误用，可以单独设计Day, Month, Year结构体。

明智而审慎地导入新类型对预防“接口被误用”有神奇疗效。

任何接口如果要求客户必须记得某些事情，就是有着“不正确使用”的倾向。

“对象在动态连接程序库中被new创建，却在另一个DLL内被delete销毁”。在许多平台上，这一类“跨DLL之new/delete成对运用”会导致运行期间错误。

- 好的接口容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
- “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
- tr1::shared_ptr支持定制型删除器。这可防范DLL问题，可被用来自解除互斥锁等等。

## 条款19 设计class犹如设计type

设计类的时候需要考虑以下问题：

- 新type对象应该如何被创建和销毁
- 对象的初始化和对象的赋值应该有什么样的差别
- 新type的对象如果被passed by value，则需要定义copy构造函数。
- 注意新type的合法值
- 如果新type是继承而来，需要注意父类的设计约束
- 注意新type的类型转换
- 什么样的操作符和函数对新type而言是合理的
- 哪些函数应该被设计为private或者delete
- 什么是新type的“未声明接口”
- 你的新type有多么一般化？

## 条款20 宁以pass-by-reference-to-const替换pass-by-value

使用pass-by-value缺点：

- 构造函数和析构函数会有代价
- 多态特性会被切除



- 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可以避免切割问题
- 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适当。

## 条款21 必须返回对象时，别妄想返回其reference

- 绝不要返回一个指针或引用指向一个local stack对象，或返回引用指向heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。

## 条款22 将成员变量声明为private

成员变量的封装性与“成员变量的内容改变时所破坏的代码数量”成反比。

从封装的角度看，其实只有两种访问权限：private（提供封装）和其他（不提供封装）



- 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保证，并提供class作者以充分的实现弹性。
- protected并不比public更具封装性。

## 条款23 宁以non-member、non-friend替换member函数

面向对象守则要求，数据以及操作数据的那些函数应该被捆绑在一起。

- 宁可拿non-member non-friend函数替换member函数。这样做可以增加封装性、包裹弹性和机能扩充性。

## 条款24 若所有参数皆需类型转换，请为此采用non-member函数

```c++
class Rational
{
public:
    Rational(int numberator = 0, int denominator = 1);
    int numberator() const;
    int denominator() const;
private:
    ...
}

Rational oneHalf;
Rational result = oneHalf * 2;		//正确，可以隐式转换
result = 2 * oneHalf;				//错误，无法隐式转换

// 建议这种形式
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    ...
}
```



- 如果需要为某个函数的所有参数类型（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member。

## 条款25 考虑写出一个不抛出异常的swap函数

```c++
namespace std
{
    template<typename T>
    void swap<Widget<T>>(Widget<T>& a, Widget<T>& b) //错误，不合法
}

namespace std
{
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) //可以，是std::swap函数的重载
}
```

C++只允许对class templates偏特化，不允许在function templates上偏特化

如果swap缺省实现版的效率不足，则可以这样做

1. 提供一个public swap成员函数，让它高效地置换你的类型的两个对象值，这个函数绝不应该抛出异常
2. 在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap成员函数
3. 如果你正编写一个class（而非class template），为你的class特化std::swap。并令它调用你的swap成员函数。



- 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
- 如果你提供一个member swap，也该提供一个non-member swap来调用前者于class（而非templates也请特化std::swap。
- 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
- 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。

# 5 实现

## 条款26 尽可能延后变量定义式的出现时间



## 条款27 尽量少做转型动作



## 条款28 避免返回handles指向对象内部成分



## 条款29 为“异常安全”而努力是值得的



## 条款30 透彻了解inlining的里里外外



## 条款31 将文件间的编译依存关系降至最低



# 6 继承与面向对象设计

## 条款32 确定你的public继承塑模出is-a关系



## 条款33 避免遮掩继承而来的名称



## 条款34 区分接口继承和实现继承



## 条款35 考虑virtual函数以外的其他选择



## 条款36 绝不重新定义继承而来的non-virtual函数



## 条款37 绝不重新定义继承而来的缺省参数值



## 条款38 通过复合塑模出has-a或“根据某物实现出”



## 条款39 明智而审慎地使用private继承



## 条款40 明智而审慎地使用多重继承



# 7 模板与泛型编程

## 条款41 了解隐式接口和编译器多态



## 条款42 了解typename的双重意义



## 条款43 学习处理模板化基类内的名称



## 条款44 将与参数无关的代码抽离templates



## 条款45 运用成员函数模板接受所有兼容类型



## 条款46 需要类型转换时请为模板定义非成员函数



## 条款47 请使用traits classes表现类型信息



## 条款48 认识template元编程



# 8 定制new和

## 条款49 了解new-handle的行为



## 条款50 了解new和delete的合理替换时间



## 条款51 编写new和delete时固守常规



## 条款52  写了placement new也要写placement delete



# 9 杂项讨论

## 条款53 不要轻忽编译器的警告



## 条款54 让自己熟悉包括TRI1在内的标准程序库



## 条款55 让自己熟悉Boost
