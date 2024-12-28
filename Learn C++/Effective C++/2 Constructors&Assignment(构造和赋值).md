
***update : 2024. 12. 18***
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
在使用容器时 ,当析构函数出了异常,一系列的析构都会出问题, 这对于编译器来说太多了,  会导致不明确的问题或者程序结束执行.  
所以最好不要让析构函数出异常, 更不能让异常逃离析构函数
```cpp
class Base
{
public:
    Base(int x) { p1 = new std::vector<int>(x); }
    virtual ~Base() = 0;

private:
    std::vector<int> *p1;
};
Base::~Base() noexcept // 析构函数
{
    delete p1;
    if (p1 == nullptr)
        std::cout << "释放了Base" << std::endl;
    else
        std::cout << "Base释放失败" << std::endl; // 若失败失败则打印错误, 精确定位错误
}

```
---
***update : 2024. 12. 19***
### 不在构造函数和析构过程中使用`virtual`函数
`virtual`函数的作用是实现动态多态性, 而多态性的条件如下:  
- **存在继承关系**   
- **子类重写父类的`virtual` 函数**    
- **使用父类指针指向子类对象**  
而在构造函数的时候, 派生类并**没有初始化完成**, 没有具体的派生类对象, 多态性无法实现. `virtual` 函数调用 的只能是基类的函数.
```cpp
class Base // 基类
{
public:
    Base() { fun(); } // 调用 fun 函数,只会调用Base类的virtual函数
    virtual void fun() { std::cout << "调用Base函数" << std::endl; }
};
class Son : public Base
{
public:
    // 重写Base类 virtual 函数
    virtual void fun() override { std::cout << "调用Son函数" << std::endl; }
};
```
而在析构过程中, 派生类中的派生类成员便会呈现未定义值, 编译器仿佛会无视它们, 因此析构过程中也不能调用`virtual`函数

---
***update : 2024. 12. 19***
### operator= (等号重载)返回一个 * this
```cpp
class Base
{
public:
    int x;
    Base(int x) : x(x) {}
    Base &operator=(const Base &base) // 重载 =
    {
        x = base.x;
        return *this; // 返回调用此函数的对象(reference),能实现链式赋值
    }
    void fun() { std::cout << "调用fun" << std::endl; }
};
Base b1(1);
Base b2(2);
Base b3(3);
(b3 = b2 = b1).fun(); // 链式
std::cout << ((b3 = b2 = b1)).x << std::endl; // 自左向右调用
// output 1
```

---
***update : 2024. 12. 20***
### 在operator = 中处理自我赋值
#### 自证测试(identity test)
对传入的对象进行测试, 若跟调用对象相等(即是同一个对象), 则不做任何操作, 直接`return`  
**缺点 :**  如果在`new`过程中内存不足或者`Base`拷贝构造函数本身出现问题, 最终会导致`p`指针指向一块被删除的内存, 这个方法本身不是完美的
```cpp
class A
{
public:
    int x;
};
class Base
{
public:
    Base &operator=(const Base &base)
    {
        if (this == &base)
            return *this;
        delete p;           // 先释放 p
        p = new A(*base.p); // 赋值操作
        return *this;
    }

private:
    A *p;
};
```
#### 异常安全性(`exception safety`)的角度解决问题 
即在赋值操作之前不释放(`delete`)指针,而是先存下副本`temp`, 在赋值成功之后再进行释放, 这样即使在 `new`过程中发生错误, `*this`并没有发生改变, 数据没有丢失
```cpp
class A
{
public:
    int x;
};
class Base
{
public:
    Base &operator=(const Base &base)
    {
        A *temp = p;        // 先存下一个副本
        p = new A(*base.p); // 进行赋值
        delete temp;        // 当new没有抛出异常,则释放掉临时副本
        return *this;
    }

private:
    A *p;
};
```

---
**update : 2024. 12. 20**
### 拷贝对象时不要忘了拷贝每一个成员属性
**赋值兼容原则 :**  
- 子类对象可作父类对象使用  
- 子类对象可赋值给父类对象   
- 子类对象可初始化父类对象  
- 父类指针可指向子类对象  
- 父类引用可引用子类对象
```cpp
class Base
{
public:
    Base(const Base &base)
        : name(base.name) {}

    Base &operator=(const Base &base)
    {
        if (this == &base)
            return *this;
        name = base.name;
        return *this;
    }
private:
    std::string name;
};
//-----------------------------------------
class Son : public Base                // 继承关系
{
public:
    Son(const Son &son)
        : Base(son), name(son.name) {} // 父类可引用子类对象

    Son &operator=(const Son &son)
    {
        if (this == &son)
            return *this;
        Base::operator=(son);          // 子类可调用父类函数 
        name = son.name;
        return *this;
    }
private:
    std::string name;
};
```
在拷贝构造函数和operator=函数时, 不要忘记给类中的每一个属性成员赋值, 最好使用深拷贝的方式 

---


