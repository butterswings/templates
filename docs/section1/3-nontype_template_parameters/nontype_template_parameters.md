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
