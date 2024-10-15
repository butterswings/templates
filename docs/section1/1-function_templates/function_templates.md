# 1.函数模板

## 1.1.初识函数模板

一组由编译器生成的函数(a family of funtion)， 未指定的参数为参数化信息

### 1.1.1.定义模板

一个返回最大值的函数模板

```cpp
template <typename T>
T max(T a, T b)
{
  return b < a ? a : b;
}
```

- T为模板参数，一个未知的类型，T为惯用名，标准库通常使用_Tp(libstdc++、libcxx)，_Ty(msvc stl)

- 由于历史原因，此处的keyword `typename`等价于`class`，用于声明类型

- 额外地，类型T在此处必须支持`operator<`，以支持代码中的三目运算符，可以使用`enable_if`对类型T施加简单约束

> [!NOTE]
> 在C++17前要求类型至少T可被拷贝构造，在C++17之后由于[复制消除和返回值优化](https://en.cppreference.com/w/cpp/language/copy_elision)，即使类型T的拷贝和移动构造函数均无效，仍然可正确传递值

### 1.1.2.使用模板

使用函数模板max

```cpp
int main()
{
  int i = 42;
  std::cout << "max(7,i):   " << ::max(7,i) << '\n';

  double f1 = 3.4;
  double f2 = -6.7;
  std::cout << "max(f1,f2): " << ::max(f1,f2) << '\n';

  std::string s1 = "mathematics";
  std::string s2 = "math";
  std::cout << "max(s1,s2): " << ::max(s1,s2) << '\n';
}
```

- 使用::是为了寻找到全局`namespace`中的`max`，也就是我们所编写的`max`函数模板生成的实例。倘若`using namespace std;`直接调用会发生名字冲突

![ambiguous_call](../../../assets/section1/1-function_templates/ambiguous_max_call.png)

程序输出：

```bash
max(7,i): 42
max(f1,f2): 3.4
max(s1,s2): mathematics
```

编译器不会生成所有类型的实体，只会生成所使用不同类型的不同实体

```cpp
// for type int
template <>
int max(int a, int b)
{
  return b < a ? a : b;
}
```

类似地,`max()`的其他调用实例化了`double`和`std::string`的`max`模板

```cpp
double max(double a, double b);
std::string max(std::string a, std::string b);
```

如果代码有效，`void`也是有效模板参数

```cpp
template <typename T>
T foo(T*)
{
}

void* vp = nullptr;
foo(vp); // deduces void foo(void*)
```

### 1.1.3.两阶段翻译

- 第一阶段模板定义：对独立的成员(不依赖于模板参数)进行检查
  - 现语法错误，如缺少逗号
  - 未知名称
  - 独立的静态断言
- 第二段模板实例化：检查非独立的成员

```cpp
template <typename T>
class A
{
public:
  void add()
  {
    printf("A add\n");
  }
};

template <typename T>
class B : public A<T>
{
public:
  void cal()
  {
    add();       // 此处报错，第一阶段查找时，add()定性为独立，认为它是非成员变量，但又找不到
    this->add(); // 此处正确，第一阶段认为它是非独立（因为与this有关，即与Ｔ有关）
                 // 第二阶段查找时，Ｂ中找不到，则在基类Ａ中能找到
  }
};

int main()
{
  B<float> b;
  b.cal();
}
```

> note: (if you use '-fpermissive', G++ will accept your code, but allowing the use of an undeclared name is deprecated)
---
<!-- 仍在施工，链接待补 -->
> 更详细的两阶段查找见[14.3.1](.)

若上文中的`max`函数模板实例化的类型不支持`operator<`，则会直接导致编译时错误，如：

```cpp
std::complex<float> c1, c2;
::max(c1, c2); // compilation error
```

> [!NOTE]
> 编译与链接，有时编译器需要在实例化时查看模板的定义：Two-phase translation leads to an important problem in the handling of templates in practice: When a function template is used in a way that triggers its instantiation, a compiler will (at some point) need to see that template’s definition. This breaks the usual compile and link distinction for ordinary functions, when the declaration of a function is sufficient to compile its use. Methods of handling this problem are discussed in Chapter 9. For the moment, let’s take the simplest approach: Implement each template inside a header file.

## 1.2.模板参数推导

模板参数可能是参数类型的一部分，如参数类型声明为const reference

```cpp
template <typename T>
T max(const T& a, const T& b)
{
  return b < a ? a : b;
}
```

- 若传递int，T在此处推导为int，而参数a和b的类型为const int&

类型推导时的类型转换

1. 引用声明参数，简单转换(trivival)不适用于类型推导，同一模板参数声明的实参必须完全匹配

2. 按值声明参数，去除顶层cv限定和引用，数组和函数类型转换为对应的指针，该转换事实上即为[`std::decay`](https://github.com/butterswings/tiny_stl/blob/main/docs/type_traits.md#decay)

```cpp
max(4, 7.2); // ERROR: T can be deduced as int or double

template <typename T>
void foo(T& a, T& b);

std::string s;
foo("hello", s); // ERROR: T can be deduced as char const[6] or std::string
```

修正：

1. 将实参强转 `max(static_cast<double>(4), 7.2)`
2. 显示指定模板参数，要求实参能够转换到T `max<double>(4, 7.2)`

默认实参类型推导

![deduction_for_default_template_param](../../../assets/section1/1-function_templates/deduction_for_default_template_param.png)

为了使`f()`能够成功调用，需要同时给定模板参数默认类型以及函数参数列表默认实参

```cpp
template <typename _Tp = std::string>
void f(_Tp = "") { }

int main(int argc, char **argv)
{
  f(1);
  f();

  return 0;
}
```

## 1.3.多模板参数

```cpp
template <typename T1, typename T2>
T1 max(T1 a, T2 b)
{
  return b < a ? a : b;
}

auto m = ::max(4, 7.2); // OK
```

- 返回值的类型依赖于第一个实参的类型，如何解决？
  1. 为返回值引入额外的模板参数
  2. 自动推导(`auto/decltype(auto)`)
  3. `common_type`

### 1.3.1.返回类型的模板参数

对于函数模板参数的类型可以被推导，但仍然可以被显式指定

```cpp
template <typename T>
T max(T a, T b);

::max<double>(4, 7.2);
```

若模板参数与参数类型无关，无法通过推导确定，则必须显式指定

```cpp
template <typename T1, typename T2, typename Rt>
Rt max(T1 a, T2 b);

::max<int, double, double>(4, 7.2);
```

很显然，手动提供所有函数模板的模板参数是不可接受的，可以考虑将类型`Rt`前置，从而提供`Rt`，推导`T1`和`T2`

```cpp
template <typename Rt, typename T1, typename T2>
Rt max(T1 a, T2 b);

::max<double>(4, 7.2);
```

### 1.3.2.推导返回类型

since C++14可以直接使用`auto`推导返回值，而不必尾置返回值

如果由有多个`return`语句则必须保证所有返回值类型均一致

```cpp
// C++14
template <typename T1, typename T2>
auto max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```

C++11，`auto`推导返回值还没这么智能，仍然需要尾置返回值类型，在此处可以使用三元运算符确认返回值类型，其中`decltype`括号内为不求值语句，此处条件可以任意

```cpp
// C++11
template <typename T1, typename T2>
auto max(T1 a, T2 b) -> decltype(false ? a : b)
{
  return b < a ? a : b;
}
```

某些情况下，C++11推导返回值可能会被推导为引用类型，但是此处返回的是`prvalue`，应对返回值decay，将箭头后的返回类型更正为`std::decay_t<decltype(false ? a : b)>`

对于C++14，则不存在这样的问题，其返回类型始终是`decay`的

[C++14-auto推导返回值规则同1.2节decay](https://github.com/butterswings/tiny_stl/blob/main/docs/type_traits.md#decay)

### 1.3.3.返回common_type

since C++11，标准库提供类模板[`common_type`](https://github.com/butterswings/tiny_stl/blob/main/docs/type_traits.md#common_type)以计算公共类型，注意`common_type`内部也使用三元运算符进行类型计算，并且会对类型应用`decay`

```cpp
template <typename T1, typename T2>
typename std::common_type<T1, T2>::type
// std::common_type_t<T1, T2>
max(T1 a, T2 b)
{
  return b < a ? a : b;
}
```
