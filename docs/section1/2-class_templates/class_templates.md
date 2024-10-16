# 2.类模板

## 2.1.实现类模板`stack`

使用`std::vector`作为适配器，包装对应接口

```cpp
template <typename T>
class Stack
{
private:
  std::vector<T> elems; // elements
  // ...
};
```

### 2.1.1.声明类模板

类似函数模板，必须将一个或多个标识符声明为模板参数，`T`仍作为惯用标识符，关键字`typename`也可使用`class`替代

```cpp
template <typename T>
class Stack;
```

在模板内部，`T`可以像其他类型一样被用来声明成员和成员函数，例如此处用于成员函数的参数或者返回值

```cpp
template <typename T>
class Stack
{
private:
  std::vector<T> elems; // elements

public:
  void push(T const &elem); // push element
  void pop();               // pop element
  T const &top() const;     // return top element
  bool empty() const
  { // return whether the stack is empty
      return elems.empty();
  }
};
```

该类的类型为`Stack<T>`，只要是在使用该类类型，就必须使用它进行声明，除非进行类型推导(比如从另外一个类型的`Stack<U>`转换到`Stack<T>`)，在类内部使用类名，表示类的模板实参与模板参数一致

```cpp
template <typename T>
class Stack
{
  Stack(const Stack&);
  Stack& operator=(const Stack&);
  
  // same as
  Stack(const Stack<T>&);
  Stack<T> operator=(const Stack<T>&);
};
```

可以注意到，需要类名的地方，只能使用Stack，如上文的拷贝构造函数

> [!NOTE]
> 与普通类不同的是，不能在块作用域或者函数内部定义类模板，类模板只能在全局/命名空间或类声明内部

### 2.1.2.实现成员函数

对于成员函数empty，其非常简短故定义于类内，让编译器自动内联

```cpp
template <typename T>
class Stack
{
  // ...
public:
  // ...
  bool empty() const
  { // return whether the stack is empty
      return elems.empty();
  }
};
```

其余函数类外定义，在类外定义成员函数需要显式指定是`Stack<T>`的成员函数，将`std::vector`作为适配器，整体实现并不复杂，不做过多解释

需要注意的是，pop函数返回值为void，并未返回刚刚弹出栈的元素，这涉及到一个异常安全的问题

```cpp
template <typename T>
void Stack<T>::push(T const &elem)
{
  elems.push_back(elem); // append copy of passed elem
}

template <typename T>
void Stack<T>::pop()
{
  assert(!elems.empty());
  elems.pop_back(); // remove last element
}

template <typename T>
T const &Stack<T>::top() const
{
  assert(!elems.empty());
  return elems.back(); // return copy of last element
}
```

## 2.2.使用类模板`Stack`

before C++17，在使用类模板时必须提供每个实参

since C++17，若模板参数可以从构造函数导出，则可以不给定实参直接跳过(*class template argument deduction - [CTAD](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)*)

```cpp
int main()
{
  Stack<int>         intStack;       // stack of ints
  Stack<std::string> stringStack;    // stack of strings

  // manipulate int stack
  intStack.push(7);
  std::cout << intStack.top() << '\n';

  // manipulate string stack
  stringStack.push("hello");
  std::cout << stringStack.top() << '\n';
  stringStack.pop();
}
```

上述代码中`int`和`std::string`作为模板实参，`Stack<int>`和`Stack<std::string>`分别使用`std::vector<int>`和`std::vector<std::string>`作为适配器

> [!NOTE]
> 代码只会对使用过的成员函数进行实例化，并且允许部分使用模板，因此`Stack<int>`比`Stack<std::string>`少实例化成员函数`pop`，*对于类模板的静态成员会对每个类型进行实例化*

可以像使用其他类型一样使用实例化的类模板类型

cv限定，引用类型，复合类型，别名声明

```cpp
void foo(Stack<int> const& s)
// parameter s is int stack(cv-qualified lval-reference)
{
  // alias
  using IntStack = Stack<int>;
  
  // compound
  Stack<int> istack[10];
  IntStack istack2[10];
}
```

作为其他模板的实参

```cpp
// as argument of other templates
Stack<Stack<int>> intStackStack; // stack of stack of ints
```

before C++11，`>>`是非法的，since C++11该问题已经被hack

## 2.3.部分使用类模板`Stack`

类模板通常会对它的模板实参应用多种操作，如构造和析构，这或许让我们觉得模板实参必须保证所有操作都是支持的，事实上模板实参仅要保证必要操作即可

例如输出`Stack`类模板持有的所有元素，未受`operator<<`支持的类型不使用该成员函数`printOn`代码就不会出错，但仍然可以使用其他成员函数

```cpp
template <typename T>
void Stack<T>::printOn(std::ostream& strm) const
{
  for (const T& elem : elems)
    strm << elem << ' ';
}
```

### 2.3.1.`concepts`(概念)

如何约束模板需要满足哪些条件才能实例化？比如约束类型支持默认构造

- 静态断言，功能简单缺乏类型推导

```cpp
template <typename T>
class C
{
  static_assert(std::is_default_constructible<T>::value);
};
```

- SFINAE，复杂type_traits和晦涩的语法

```cpp
template <typename T,
          typename = std::enable_if_t<std::is_default_constructible_v<T>>>
class C
{ /* ... */ };
```

- 概念与约束：良好的错误诊断与代码风格，C++20带来的福音

```cpp
template <typename T>
requires std::default_initializable
class C
{ /* ... */ };
```
