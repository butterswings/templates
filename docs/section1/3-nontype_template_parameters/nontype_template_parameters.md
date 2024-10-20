# 3.非类型模板参数

## 3.1.非类型类模板参数

使用`std::array`作为前章节实现的模板`Stack`的容器适配器

```cpp
template <typename T, std::size_t MaxSize>
class Stack
{
private:
  std::array<T, MaxSize> elems;
  std::size_t numElems; // cursor
public:
  // ...
};
```

`MaxSize`决定是否栈满，`numElems`用于出入栈和判断栈空

使用它时，只需额外提供一个栈最大容量

```cpp
Stack<int, 20> int20Stack;
```

非类型模板参数也可指定默认参数，此处可以规定栈默认大小

```cpp
template <typename T, std::size_t MaxSize = 100>
class Stack
{ /* ... */ };
```

## 3.2.非类型函数模板参数

一个例子：

```cpp
template <int Val, typename T>
T addValue(T x)
{
  return x + Val;
}
```

配合`std::transform`使用

```cpp
std::transform(source.begin(), source.end(), dest.begin(), addValue<5, int>);
```

对range`source`的每个元素，为其增加5

最后一个参数，是一个可调用对象，对于函数模板，必须显式指定所有实参，已明确是重载集中的哪一个函数

其他用法：

- 推导非类型模板参数类型

```cpp
template <auto Val, typename T = decltype(Val)>
T foo();
```

- 使用基于未知类型的非类型模板参数

```cpp
template <typename T, T Val = T{ }>
T foo();
```

## 3.3.非类型模板参数的限制

原书的限制貌似只到C++17，左值引用以及大部分标量(排除浮点)，对于C++20此限制被进一步放宽，下放浮点，未捕获的`lambda`，非闭包的字面量类型(`constexpr`)

[类型要求](https://en.cppreference.com/w/cpp/language/template_parameters)

before C++20 (C++17)不允许浮点类型

```cpp
template <double VAT>
double process(double v)
{
  return v * VAT;
}
```

before C++20 (C++17)不允许class-type object

```cpp
template <std::string name>
class MyClass { /* ... */ };
```

向模板参数传递引用或指针，不能是字符串字面量，临时值，数据成员及其子对象，在C++17前此要求逐步放宽

- C++11，外部链接
- C++14，外部或内部链接性
- C++17，无链接性

```cpp
extern char const s03[] = "hi"; // external linkage
char const s11[] = "hi"; // internal linkage
int main()
{
  Message<s03> m03; // OK (all versions)
  Message<s11> m11; // OK since C++11
  static char const s17[] = "hi"; // no linkage
  Message<s17> m17; // OK since C++17
}
```

补充：

1. 声明外部链接的变量的方法是在代码块外面声明它。此变量是全局变量，多文件中亦可用
2. 声明内部链接的变量的方法是在代码块外面声明它并加上static限定符. 此变量是全局变量，但仅在本文件中可用
3. 声明无链接的变量的方法是在代码块里面声明它并加上static限定符. 此变量是局部变量，但仅在本代码块中可用

## 3.4.模板参数类型`auto`

Since C++17，可以推导非类型模板参数类型

```cpp
template<typename T, auto Maxsize>
class Stack {
public:
  using size_type = decltype(Maxsize);
private:
  std::array<T,Maxsize> elems; // elements
  size_type numElems;          // current number of elements
  // ...    
}
```

跟前节的内容一致，只不过栈大小类型可被推导

```cpp
int main()
{
  Stack<int,20u>        int20Stack;     // stack of up to 20 ints
  Stack<std::string,40> stringStack;    // stack of up to 40 strings

  // manipulate stack of up to 20 ints
  int20Stack.push(7);
  std::cout << int20Stack.top() << '\n';
  auto size1 = int20Stack.size();

  // manipulate stack of up to 40 strings
  stringStack.push("hello");
  std::cout << stringStack.top() << '\n';
  auto size2 = stringStack.size();

  if (!std::is_same_v<decltype(size1), decltype(size2)>) {
    std::cout << "size types differ" << '\n';
  }
}
```

很明显地可以看到，`int20Stack`的`size_type`为`unsigned`，而`stringStack`为`int`

before C++20，使用浮点仍然受限

```cpp
Stack<int, 3.14> s; // ERROR
```

传递无链接性的字符数组(C++17)

```cpp
template<auto T>       // take value of any possible nontype parameter (since C++17)
class Message {
  public:
    void print() {
      std::cout << T << '\n'; 
    }
};

int main()
{
  Message<42> msg1;
  msg1.print();        // initialize with int 42 and print that value

  static char const s[] = "hello";
  Message<s> msg2;     // initialize with char~const[6] "hello"
  msg2.print();        // and print that value
}
```

`decltype(auto)`也可以作为占位符，区别在于`auto`与`decltype`推导的不同

```cpp
template <decltype(auto) N>
class C { /* ... */ };

int i;
C<(i)> c; // type N deduced int&
```
