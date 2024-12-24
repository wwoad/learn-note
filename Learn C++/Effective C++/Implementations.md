
---
***update : 2024. 12. 23***
### 尽可能延后变量的创建(`create`和`use`应该尽量靠近)
#### 变量的创建和使用
在实际的代码中,当定义了一个变量 ,但在运行过程中因为代码错误(但并未停止运行), 而没有被使用时则会造成构造成本, 这一点在`class`对象足够大时, 会尤其明显
```cpp
std::string fun(std::string str)
{
    std::string str = "....";
    //if...... 检测不通过,运行异常等 导致提前代码在此提前结束,但并未报错,没有被排查出来
    // ......对str的操作
    return str;
}
```
上述例子将会导致创建`str`的构造成本
**优化如下:** 将变量的创建和使用尽量靠近, 易于于都和理解的同时，减少构建成本可能会发生的浪费
```cpp
std::string fun(std::string str)
{
    //if ....如果有的话,将检错或者输入合法性等放在最开始
    std::string str = "....";
    // ......对str的操作
    return str;
}
```
**总结就是要使用才创建, 创建和使用尽量靠近**
#### 创建变量的方式优化
主要的创建方式:   先`default`构造再赋值   创建时直接初始化(如果设计允许的话)
```cpp
void fun(std::string str)
{
   // 先default构造再赋值
    std::string str1;
    str = str;
   // 创建时直接初始化完毕
   std::string str2(str);
}
```
**前者效率差且`default`构造毫无意义, 后者效率高且易于理解**
#### 循环的情况下变量的创建和使用
```cpp
// A: 在循环前先创建,在循环内使用
std::string str;
if(int i = 0;i < 100;i++)
{
  //......使用 str
}

// B:每次循环时都创建这个变量
if(int i = 0;i < 100;i++)
{
    std::string str;
    // ......使用 str
}
```
**方案A** 效率高, 程序有阅读门槛, 可维护性差  
**档案B** 效率相对较低, 但程序易于理解, 可维护性好
**结论:** 当两个方案**效率相差非常多**的巨大的情况下 ,  或者是程序对**运行效率要求很敏感**的情况下,选择**A**方案  
否则, 一律使用**B方案**

---
***update : 2024. 12. 23***
### 尽量避免函数返回一个handles(句柄)
所谓`handles`包括有**指针** **引用** **迭代器**等  
不是非得返回handles的情况下(即`operator`的情况),  应该尽量避免这一情况发生, 不然会发生许多意料之外的问题, 容易违反封装性原则
```cpp
class Base
{
public:
    Base(int x) : x(x) {}
    Base &operator=(const Base &base); // 一定得返回handles的情况
    Base &doSomething()                // 没有必要返回handles
    {
        // ......
        return *this;
    }
    void setX(int x) { this->x = x; }
private:
    int x;
};
Base b(1);
b.doSomething().setX(2); // 这会造成很多不符合设计要求的调用,这违背了封装性的原则
```

---
***update : 2024. 12. 24***
### 注意异常安全(Exception Safety)
#### 使用RAII(资源获取即初始化)的思想设计程序
`RAII`利用对象的生命周期自动管理资源,如智能指针, 容器, 其他管理类等自动管理资源
```cpp
void addElement(std::vector<int> &v, int element)
{
    v.push_back(element); // 如果此时发生异常, 
    //vector容器会自动释放内存,而不会造成其他未知的异常
}
```
#### 使用Copy-and-swap idiom(拷贝与交换惯用法)
通过传值和交换的方式确保了强异常安全, 如果在交换过程中抛出异常, 原本的对象会保留其状态, 并且内存不会泄漏. 传值的方式意味着临时对象（`base`）会在返回之前交换它的内容
```cpp
class Base
{
public:
    Base(std::string *str, int size)
        : str(str), size(size) {}
    Base(Base &&base) noexcept : str(base.str), size(base.size) // 移动构造函数
    {
        base.str = nullptr; // 移动之后将原指针置空
        base.size = 0;      // 修改原数据
    }
    Base(const Base &base) // 普通的深拷贝
    {
        size = base.size;
        str = new std::string(*base.str);
    }
    Base &operator=(Base base) // 拷贝和交换惯用法,传参形式为值传递
    {
        swap(base); // 自定义一个类的交换函数,传入对象与调用对象交换属性
        return *this;
    }
    void swap(Base &base) // 自定义的交换函数
    {
        std::swap(str, base.str);
        std::swap(size, base.size);
    }
private:
    std::string *str;
    int size;
};
```
#### 避免异常产生资源泄漏
使用标准库的智能指针 容器 算法等管理资源, 确保异常安全
```cpp
void fun1() // 传统的指针操作
{
    int *ptr = new int[100];
    //......一些语句,导致提前return或者报出异常
    delete ptr; // 上述语句报出异常或者提前return,ptr无法正常释放
}
void fun2() // 使用智能指针管理资源
{
    std::unique_ptr<int[]> ptr = std::make_unique<int[]>(100);
    //......一些语句,导致提前return或者报异常
    // ptr会在函数结束后自动释放,不必担心指针无法正常释放的问题,提高安全性
}
void fun3(int arr[10]) //使用vector管理资源
{
    std::vector<int> v;
    for (int i = 0; i < 10; i++)
        v.push_back(arr[i]);
    std::cout << *v.begin();
}
```
---
### 理解`inline`的使用
#### `inline`的优点


#### 使用方法和适用场景









---














---




















---
