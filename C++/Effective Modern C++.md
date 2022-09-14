# Effective Modern C++

# 第1章 型别推导

## 条款1：理解模板型别推导

```c++
template<typename T>
void f(ParamType param);
```

### 情况1：ParamType是个指针或引用，但不是个万能引用

```c++
template<typename T>
void f(T& param);
```

1. 若expr具有引用型别，先将引用部分忽略
2. 尔后，对expr的型别和ParamType型别执行模式匹配，来决定T的型别

```c++
int x = 27; // x is an int
const int cx = x; // cx is a const int
const int& rx = x; // rx is a reference to x as a const int

f(x); 	// T is int, param's type is int&
f(cx); 	// T is const int,
		// param's type is const int&
f(rx); 	// T is const int,
		// param's type is const int&
```

### 情况2：ParamType是个万能引用

```c++
template<typename T>
void f(T&& param);
```

1. 如果expr是个左值，T和ParamType都会被推导为左值引用。这个结果具有双重的奇特之处：首先，这是在模板型别推导中，T被推导为引用型别的唯一情形。其次，尽管在声明时使用的是右值引用的语法，它的型别推导结果却是左值引用。
2. 如果expr是个右值，则应用情况1中的规则

```c++
template<typename T>
void f(T&& param); 		// param is now a universal reference
int x = 27; 			// as before
const int cx = x; 		// as before
const int& rx = x; 		// as before
f(x); 					// x is lvalue, so T is int&,
						// param's type is also int&
f(cx); 					// cx is lvalue, so T is const int&,
						// param's type is also const int&
f(rx); 					// rx is lvalue, so T is const int&,
						// param's type is also const int&
f(27); 					// 27 is rvalue, so T is int,
						// param's type is therefore int&&
```

### 情况3：ParamType既非指针也非引用

```c++
template<typename T>
void f(T param); 		// param is now passed by value
```

1. 一如之前，若expr具有引用型别，则忽略其引用部分
2. 忽略expr的引用性之后，若expr是个const对象，也忽略之。若其是个volatile对象，同忽略之。

```c++
int x = 27; 		// as before
const int cx = x; 	// as before
const int& rx = x; 	// as before
f(x); 				// T's and param's types are both int
f(cx);				// T's and param's types are again both int
f(rx); 				// T's and param's types are still both int
```

### 数组实参

```c++
const char name[] = "J. P. Briggs"; // name's type is const char[13]
const char * ptrToName = name; 		// array decays to pointer

template<typename T>
void f(T param); 					// template with by-value parameter
f(name); 							// name is array, but T deduced as const char*

template<typename T>
void f(T& param); 					// template with by-reference parameter
f(name); 							// T dedeced as const char [13]
```

可以利用声明数组引用这一能力创造出一个模板，用来推导出数组含有的元素个数

```c++
template<typename T, std::size_t N> // see info
constexpr std::size_t arraySize(T (&)[N]) noexcept // below on
{ 													// constexpr
    return N;
}
```



### 函数实参

```c++
void someFunc(int, double); // someFunc is a function;
							// type is void(int, double)
template<typename T>
void f1(T param); 			// in f1, param passed by value
template<typename T>
void f2(T& param); 			// in f2, param passed by ref
f1(someFunc); 				// param deduced as ptr-to-func;
							// type is void (*)(int, double)
f2(someFunc); 				// param deduced as ref-to-func;
							// type is void (&)(int, double)
```

