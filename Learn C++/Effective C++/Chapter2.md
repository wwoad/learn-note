
***update : 2024. 12. 18***
#Chapter2_default_function
### C++编译器默认生成类内函数
在声明了一个空类或者少写了某些构造, 析构, `operator=`的情况下, C++ 编译器会默认生成函数
```cpp
class Empty
{ };

//上面的空类默认生成如下
// class Empty
// {
// public:
//     Empty() {}
//     ~Empty() {}
//     Empty &operator=(const Empty &rhs) {}
// };
```
而所有这些默认的函数只能生成**最简单**的功能, 如默认构造函数只能按照成员属性的类型进行**最简单的初始化**  
**= 重载**也只能按照类内成员属性声明的顺序进行赋值操作, 但遇到 **const** 或者 **&** 类型的成员属性时将会失效, 无论如何, **默认生成类函数总是不靠谱的**.  
因此, 我们因该尽量避免这类情况的发生
```cpp
template <class T>
class Empty
{
public:
    Empty(std::string str, T value, int x) : str(str), value(value), x(x) {}
    ~Empty(){ }
private:
    std::string &str;
    const T value;
    int x;
};
Empty<int> empty1("haha", 1, 1);
Empty<int> empty2("hello world", 2, 1);
empty2 = empty1; // error, 因为类内存在 const 和 & 成员 
```
**特别的 :** 当`Base class` 的 `operator=` 函数定义在`private`中时,编译器将不会为 `derrived class` 生成默认的 `operator=` 函数
```cpp
class Base
{
public:
    Base(int x) : x(x) {}

private:
    Base &operator=(const Base &rhs)
    {
        this->x = rhs.x;
        return *this;
    }
    int x;
};
class Son : public Base
{
public:
    Son(int x, int y) : Base(x), y(y) {}
    int y;
};
Son s1(1, 1);
Son s2(2, 2);
s1 = s2; // error,Son 没有默认的operator=函数
```
因此,`operator=`写在`public`下最好

---

***update : 2024. 12. 18***
#Chapter2_refuse_default
### 明确控制编译器自生成的函数
#### 明确使用默认函数
```cpp
class Base
{
public:
    Base() = default; // 明确使用默认的构造函数
    Base &operator=(const Base &rhs) = default; // 明确使用默认的operator= 函数
};
```
#### 明确拒绝使用默认的函数
```cpp
class Base
{
public:
Base(const Base &rhs) = delete; // 明确拒绝使用拷贝构造函数
Base &operator=(const Base &rhs) = delete; // 明确拒绝使用复制操作
};
```

---
***update : 2024. 12. 18***
### 一定要为多态的基类声明`virtual` 函数
在使用多态时一定要给基类实现一个虚析构，防止造成`Base`类部分没有被释放的 '局部释放' 的情况，造成内存泄露
```cpp
class Base  //抽象类
{
public:
    Base(int x) { p1 = new std::vector<int>(x); }
    virtual ~Base() = 0; // 纯虚析构,必须在类外实现

private:
    std::vector<int> *p1;
};
Base::~Base() // 纯虚函数需要(只能)在类外实现,确保释放基类裸指针
{
    delete p1;
    std::cout << "释放Base" << std::endl;
}

class Son : public Base // 可实例化的派生类
{
public:
    Son(int x1,int x2) : Base(x1) {p2 = new std::vector<int>(x2);}
    ~Son();

private:
    std::vector<int> *p2;
};
Son::~Son() // 普通虚构,确保释放派生类裸指针
{
    delete p2;
    std::cout << "释放Son" << std::endl;
}

//output 释放Base 释放Son 
```
**特别的 :** 纯虚析构函数必须在类外有实现, 否则`error`. 若 `class` 设计出来不是为了作为`Base`或者多态的用途, 就不该使用 `virtual` 析构

----
***update : 2024. 12. 18***
### 不让异常逃离析构函数



---
