
-----
***update: 2024. 12. 27***
### 隐式接口和编译期多态
#### 隐式接口
隐式接口的优点在于它减少了显式接口定义的冗余代码, 使得代码更加简洁、灵活.  缺点是编译时错误可能更加难以诊断,  尤其是在错误信息没有明确指出哪个成员缺失时
```cpp
template <class T>     
void print(const T &t)   // 使用模板函数,可以打印任何实现了 print() 方法的对象
{
    t.print();
}
//-------------------------
class A
{
public:
    void print() const { std::cout << "A's print" << std::endl; }
};
class B
{
public:
    void print() const { std::cout << "B's print" << std::endl; }
};
//    test
A a;
B b;
print(a);   // 隐藏A类的print()
print(b);
//    output
/*
    A's print
    B's print
*/
```
依赖于编译器在编译时检查类型是否满足某些条件（如是否有`print()`成员函数）
#### 编译时多态(静态多态)
##### 模板实现静态多态
```cpp
template <class T, class U>
inline T add(T value_1, U value_2)
{
    return value_1 + value_2;
}
```
##### 模板特化
###### 普通的模板特化
针对形参类型 `double` 和 `int` 的特化版本
```cpp
template <>
inline double add<double, int>(double value_1, int value_2)
{
    std::cout << "这是一个特化的模板函数" << std::endl;
    return value_1 + static_cast<double>(value_2);
}

//   test
auto sum = add(1.0, 1);
std::cout << sum << std::endl;
// output  这是一个特化的模板函数   2
```
###### 使用`SFINAE` 和 `std::enable_i`实现模板特化
对`T`为`double`,  `U`为`int`的特化版本
```cpp
template <typename T, typename U>
typename std::enable_if<std::is_same<double, T>::value && std::is_same<int, U>::value, T>::type inline add(T value_1, U value_2)
{
    std::cout << "这是一个特化的模板函数" << std::endl;
    return value_1 + static_cast<T>(value_2);
}
```
#### `constexpr`实现编译时多态
`constexpr`允许在编译时执行计算.  这意味着函数或表达式可以在编译期间计算出来,  从而提高程序的效率并使代码具有编译时多态性
```cpp
template <typename T, typename U>
constexpr inline T add(T value_1, U value_2)
{
    return value_1 + static_cast<T>(value_2);
}
```
---
***update: 2024. 12. 28***
### typename的双重意义
在模板声明是`typename`和`class`没有区别
```cpp
template <typename T>
template <class T>
inline T add(T value_1,T value_2) {return value_1 + value_2;}
```
但在模板类的嵌套使用时, 会遇到要使用**被嵌套模板类**的模板类型`T`  
`typename`能明确指出**被嵌套模板类**的中声明的**value_type**是模板类的模板类型而不是其他的什么成员
```cpp
template <typename T>
class Base
{
public:
    using value_type = T; // 类型别名
    // 打印此模板类的模板类型 T
    void printValue() { std::cout << "Base's value type: " << typeid(value_type).name() << std::endl; }
};
//--------------------------------------
template <typename T>
class Son : public Base<T>
{
public:
    // 使用typename 明确指出value_type是模板类型名而不是其他的什么东西
    using value_tpye = typename Base<T>::value_type;
    void printSonValue()
    {
        // Base<T> 中的类型别名 'value_type' 被派生类访问
        std::cout << "Son's value type: " << typeid(value_tpye).name() << std::endl;
    }
};
```
---
***update: 2024. 12. 28***
### 将与模板无关的部分移出模板函数(或类)`template`
```cpp
void vectorSwap(std::vector<int> &v)
{
    if (!v.empty())
    {
        std::cout << "start sort" << std::endl;
        for (std::vector<int>::iterator it_1 = v.begin(), it_2 = v.end() - 1; it_1 < it_2; it_1++, it_2--)
        {
            std::swap(*it_1, *it_2);
        }
    }
    else
        std::cout << "vector is empty" << std::endl;
}

template <typename T>
void sortSwap(std::vector<T> &v)
{
    vectorSwap(v);   // 将与模板无关的部分移出模板函数
}
```

















---
























---








