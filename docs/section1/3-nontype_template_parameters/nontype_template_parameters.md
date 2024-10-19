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
