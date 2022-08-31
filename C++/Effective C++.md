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

尽可能延后对象的定义，直到可以给定它初始实参为止。这样不仅能够避免构造和析构函数，还能避免无意义的default构造行为。

- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。

## 条款27 尽量少做转型动作

C++四种新式转型

- const_cast通常被用来将对象的常量性转除。
- dynamic_cast主要用来执行“安全向下转型”，也就是用来决定对象是否归属继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作。
- reinterpret_cast意图执行低级转型，实际动作可能取决于编译器。
- static_cast用来强迫隐式转换，将int转换为double



```c++
class Window{...};

class SpecialWindow: public Window
{
public:
    void blink();
    ...
};

typedef std::vector<std::tr1::shared_ptr<Window>> VPW;
VPW winPtrs;
...
for(VPW::iterator iter = winPtrs.begin();
   	iter != winPtrs.end();++iter)
{
    if(SpecialWindow * psw = dynamic_cast<SpecialWindow*>(iter->get()))
    {
        psw->blink();
    }
}
```

不应该采用dynamic_cast，应该如此使用：

```c++
typedef std::vector<std::tr1::shared_ptr<SpecialWindow>> VPSW;
VPSW winPtrs;
...
for(VPSW::iterator iter = winPtrs.begin();
   	iter != winPtrs.end();++iter)
{
    (*iter)->blink();
}
```

或者将blink定义成虚函数，尽量避免转型。

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_cast。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码
- 宁可使用新式转型，不要使用旧式转型。

## 条款28 避免返回handles指向对象内部成分

- 避免返回handles(包括references、指针、迭代器)指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”(dangling handles)的可能性降至最低。


## 条款29 为“异常安全”而努力是值得的

异常安全有两个条件：

- 不泄露任何资源。
- 不允许数据败坏。

```c++
class PrettyMenu{
public:
    void changeBackground(std::istream& imgSrc);
private:
    Mutex mutex;
    Image* bgImage;
    int imageChange;
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imageSrc);
    unlock(&mutex);
}
```

存在问题：

- mutex手动释放，容易出问题
- new 之前delete，如果new抛出异常，则会有悬空指针

改进：

```c++
class PrettyMenu{
public:
    ...
    std::tr1::shared_ptr<Image> bgImage;
    ...
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock m1(&mutex);
    bgImage.reset(new Image(imgSrc));
    ++imageChange;
}
```

存在问题，Image构造函数可能会出现异常，有可能输入流的读取记号已被移走

使用swap改进

```c++
struct PMImpl{
    std::tr1::shared_ptr<Image> bgImage;
    int imageChanges;
}

class PrettyMenu{
...
private:
    Mutex mutex;
    std::tr1::shared_ptr<PMImpl> pImpl;
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    using std::swap;
    Lock m1(&mutex);
    std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
    
    pNew->bgImage.reset(new Image(imgSrc));
    ++pNew->imageChanges;
    
    swap(pImpl, pNew);
}
```



- 异常安全函数即使发生异常也不会泄漏资源或允许任何数据结构被破。这样的函数分为三种可能的保证：基本型、强烈型、不抛异常型。
- “强烈保证”往往能够以copy-and-swap实现出来，但是“强烈保证”并非对所有函数都可以实现或具备现实意义。
- 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

## 条款30 透彻了解inlining的里里外外

inline缺陷：

- 造成程序体积变大，可能会导致额外的换页行为，降低指令高速缓存装置击中率，以及伴随这些而来的效率损失

inline只是对编译器的一个申请，不是强制命令。将函数定义在class内，也是inline。



- 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升最大化。【因为Inline函数代码已经嵌入进去，如果inline函数改变，则嵌入的客户代码也需要重新编译】
- 不要只因为function templates出现在头文件，就将它们声明为inline。

## 条款31 将文件间的编译依存关系降至最低【没有完全理解】

```c++
class Person{
public:
    Person(const std::string& name, const Data& birthday,
          const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::string theName;
    Date theBirthDate;
    Address theAddress;
};
```

如果Person和Data,Address分开，可以使用指针

```c++
class PersonImpl;
class Date;
class Address;

class Person{
public:
    Person(const std::string& name, const Data& birthday,
          const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::tr1::shared_ptr<PersonImpl> pImpl;
};
```

设计策略：

- 如果使用object references或object pointers可以完成任务，就不要使用objects。
- 如果能够，尽量以class声明式替换class定义式。
- 为声明式和定义式提供不同的头文件。

C++提供关键字export，允许将template声明式和template定义式分割于不同的文件内。



- 支持“编译依存最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和Interface classes
- 程序库头文件应该以“完全且仅有声明式”的形式存在。这种做法不论是否涉及templates都适用

# 6 继承与面向对象设计

## 条款32 确定你的public继承塑模出is-a关系

public继承主张，能够施行于base class对象身上的**每件事情**，也可以施行于derived class对象上。

is-a并非是唯一存在于classes之间的关系。另外两个常见的关系是has-a和is-implemented-in-terms-of。

- “public继承”意味着is-a。适用于base classes身上的每一件事情也一定适用于derived classes身上，因为每一个derived class对象也都是一个base class对象。

## 条款33 避免遮掩继承而来的名称

derived class的作用域被嵌套在base class作用域内。

```c++
class Base{
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
private:
    int x;
};

class Derived : public Base{
public:
    virtual void mf1();		// 会覆盖基类的virtual void mf1()和virtual void mf1(int)
    void mf3();				// 会覆盖基类的void mf3()和void mf3(double)
    void mf4();
    ...
};
```

子类的函数覆盖不单单覆盖一个函数，会覆盖父类的整个重载函数。如果要使用父类被覆盖的函数，可以使用using，或者转交函数

```c++
class Derived : public Base{
public:
    using Base::mf1;
    using Base::mf3;
    virtual void mf1();		
    void mf3();				
    void mf4();
    ...
};

class Derived : public Base{
public:
    virtual void mf1()		// 转交函数
    {
        Base::mf1();
    }
};
```



- derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此
- 为了被遮掩的名称重现天日，可使用using声明式或转交函数（forwarding functions）.

## 条款34 区分接口继承和实现继承

public继承概念分为函数接口继承和函数实现继承。

- 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口

- 声明简朴的（非纯）impure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现

- 声明non-virtual函数的目的是为了令derived classes继承函数的接口及一份强制性实现。

  



- 接口继承和实现继承不同。在public继承下，derived classes总是继承base class的接口
- pure virtual函数只具体指定接口继承
- impure virtual函数具体指定接口继承及缺省实现继承
- non-virtual函数具体指定接口继承以及强制性实现继承

## 条款35 考虑virtual函数以外的其他选择

本条款说明了strategy模式，在此先介绍strategy模式

**策略模式**：一个类的行为或其算法可以在运行时更改。

```c++
class GameCharacter{
public:
    int healthValue() const
    {
        ...
        int retVal = doHealthValue();
        ...
        return retVal;
    }
private:
    virtual int doHealthValue() const
    {
        ...
    }
};
```

non-virtual-interface(NVI)手法：令客户通过public non-vitual成员函数间接调用private virtual函数。



使用函数指针实现strategy模式，

```c++
class GameCharacter{
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf)
    {}
    int healthValue() const
    {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};

int loseHealthQuickly(const GameCharacter&);	// 健康指数计算函数1
int loseHealthSlowly(const GameCharacter&);		// 健康指数计算函数2
```



- virtual 函数的替代方案包括NVI手法及Stategy设计模式的多种形式。NVI手法自身是一个特殊形式的Template Method设计模式
- 将机能从成员函数移到class外部函数，带来一个缺点是，非成员函数无法访问class的non-public成员
- tr1::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标名式兼容”的所有可调用物。

## 条款36 绝不重新定义继承而来的non-virtual函数

重新定义继承而来的non-virtual函数破坏了is-a的关系。

- 绝对不要重新定义继承而来的non-virtual函数


## 条款37 绝不重新定义继承而来的缺省参数值

缺省参数值是静态绑定的。

```c++
class Shape{
public:
    enum ShapeColor{RED, GREEN, BLUE};
    virtual void draw(ShapeColor color = RED) const  = 0;
    ...
};

class Rectangle : public Shape{
public:
    virtual void draw(ShapeColor color = RED) const;
    ...
};
```

如果Shape内的缺省参数值改变了，所有“重复给定缺省参数值”的那些derived classes也必须改变，否则会导致“重复定义一个继承而来的缺省参数值”。

解决方法是使用NVI手法，令base class内的一个public non-virtual函数调用private virtual函数，后者可被derived classes重新定义。

- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而virtual函数是动态绑定。

## 条款38 通过复合塑模出has-a或“根据某物实现出”

```c++
// has-a
class Address{...};
class PhoneNumber{...};
class Person{
public:
    ...
private:
	std::string name;
    Address address;
    PhoneNumber voiceNumber;
    PhoneNumber faxNumber;
}

// is-implemented-in-terms-of
template<T>
class Set{
public:
    bool member(const T& item) const;
    bool insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
 private:
    std::list<T> rep;
};
```

- 复合的意义和public继承完全不同
- 在应用域，复合意味着has-a。在实现域，复合意味着is-implemented-in-terms-of

## 条款39 明智而审慎地使用private继承

如果classes之间的继承关系是private，编译器不会自动将一个derived class对象转换为一个base class对象。

由private base class继承而来的所有成员，在derived class中都会变成private属性，纵使它们在base class中原本是protected或public属性

private继承意味着只有实现部分被继承，接口部分应略去。如果D以private形式继承B，意思是D对象根据B对象实现而得。



- private继承意味着is-implemented-in-terms of。它通常比复合的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
- 和复合不同，private继承可以早场empty base最优化。这对致力于“对象尺寸最小化”的程序开发者而言，可能很重要。

## 条款40 明智而审慎地使用多重继承【存疑】

**不理解部分**：P198的私有继承，改为公有继承也可以达到相同效果

对于virtual继承

- 非必要不使用virtual base。平常使用non-virtual继承
- 必须使用virtual base classes，尽可能避免在其中放置数据。



- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要
- virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base classes不带任何数据，将是最具实用价值的情况
- 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两相组合。

# 7 模板与泛型编程

## 条款41 了解隐式接口和编译器多态

- classes和templates都支持接口和多态
- 对classes而言接口是显式的，以函数签名为中心，多态则是通过virtual函数发生于运行期
- 对template参数而言，接口是隐式的，奠基于有效表达式。多态则是通过template具现化和函数重载解析发生于编译期

## 条款42 了解typename的双重意义

```c++
template<typename C>
void print2nd(const C& container)
{
    C::const_iterator *x;
    ...
}
```

如果C::const_iterator恰巧是static成员变量，x是一个global变量名称，则上述代码是相乘的意思，此时就会产生歧义。

```c++
template<typename C>
void print2nd(const C& container)
{
    typename C::const_iterator *x;
    ...
}
```

所以需要在前面加上typename表明是类型。

任何时候想要在template中指涉一个**嵌套从属类型名称**，就必须在紧邻它后面的前一个位置放上关键字typename。

凡事都有例外，两种情况不能加typename

```c++
template<typename T>
class Derived: public Base<T>::Nested{	// base class list中不允许typename
public:
    explicit Derived(int x)
        :Base<T>::Nested(x)				// 初始化列表中不允许typename
    {
        typename Base<T>::Nested temp;	// 可以加typename
    }
}
```





- 声明template参数时，前缀关键字class和typename可互换
- 请使用关键字typename标识嵌套从属类型名称，但不得在base class list或初始化列表中将它作为base class修饰符

## 条款43 学习处理模板化基类内的名称

当全特化的模板中出现了一般模板中没有出现的函数，

- 可以在derived class template内通过"this->"指涉base class templates内的成员名称，或藉由一个明白写出的"base class 资格修饰符"完成。

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
