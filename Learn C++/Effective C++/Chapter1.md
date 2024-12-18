
---

***update :  2024. 12. 16***
#Chapter1_define
### 尽量使用const,  enum class,  inline 替换#define

因为宏可能会在编译器未处理源码前被预处理器移走, 导致宏没有进入记号表(symbol table), 而在错误报错中报错信息为其值. 在下面的例子中为 100. 而当我们在使用非自写的宏时, 对相关宏毫无概念导致浪费时间查找其出处
#### 使用 const 
`const` 相比 `#define` 提供了更强的类型安全, 作用域控制, 调试支持, 并且更容易与现代 C++ 的特性（如模板、类等）兼容
```cpp
#define VALUE 100

template<typename T>
const int Value = 100;  // globle veriables use 'upper camel case' (大驼峰命名法) 

// 在类内使用const 代替 #define
class A
{
pubilc:
    static const int Value = 100; // 使用static 保证 Value 只有一份  
};
```
#### 使用 enum class
好处:  类型安全, 有作用域, 枚举值只能在定义的枚举类型中访问, 防止污染全局命名空间  
缺点:  `enum class` 需要使用 `enum` 类型, 可能有微小性能开销
```cpp
#define VALUE 100

enum class A: int  // 显示指定枚举类型,  
{
    Value = 100
};

// #define 类型不安全
int value_1 = VALUE;
float value_1 = VALUE;

// enum class 类型安全
A value_2 = A::Value; 

// 好处: 类型安全,有作用域,枚举值只能在定义的枚举类型中访问,防止污染全局命名空间
// 缺点: enum class 需要使用 enum 类型,可能有微小性能开销

```
#### 使用 inline
`#define` 宏会直接文本替换, 可能导致副作用. 例如, 如果你写了一个宏 `SQUARE(x)`, 并传递了 `x+1`, 它会错误地扩展为 `((x+1)*(x+1))`, 这可能会导致意外的结果.  
使用 `inline` 函数时, 编译器会检查类型并避免类似的副作用, 且可以兼容现代C++的模板, 类等  

***性能  :***  `inline` 声明的函数, 直接插入到调用处, 在简单的函数使用时, 减少了函数调用的开销  
对于简单的运算类的宏函数, 使用 inline 替换是不错的方案 
```cpp
#define ADD(value1,value2) ((value1) + (value2))

template<class T>
inline T add(T value1, T value2)
{
    return ((value1) + (value2));
}
// 类型相同的正常调用
std::cout << ADD(1, 1) << std::endl; // output 2
// 类型不相同的'异常'调用
std::cout << ADD(1.1, 1) << std::endl; // output 2.1
// 类型不安全

std::cout << add_1(1, 1) << std::endl;   // output 2
std::cout << add_1(1.1, 1) << std::endl; // error          
// 类型安全       
```

---

***update :  2024. 12. 16***
#Chapter1_const
### 尽量使用 const
#### const 与指针
```cpp
std::string str = "hello world";
std::string arr = "haha";
//const 在*左边
const std::string *pstr1 = &str;       // pstr1 所指为常量
pstr1 = &arr;                          //成功, 能更改
//const 在*右边
std::string *const pstr2 = &str;       // pstr2 本身为常指针
pstr2 = &arr;                          // error, 不可更改常指针所指
//const 同时在*的左右两边
const std::string *const pstr3 = &str; // pstr3 本身和所指都是常量
```
#### const 与迭代器 iterator
```cpp
std::vector<int> arr;
arr.emplace_back(1);
const std::vector<int>::iterator it_1 = arr.begin(); // 类似 T* const
*it_1 = 2;                                           //成功,能改变所指的内容
++it_1;                                              //error,it 为常指针

std::vector<int>::const_iterator it_2 = arr.begin(); // 类似 const T*
*it_2 = 2;                                           //error,不能改变所指的内容
++it_2;                                  //成功,能更改指针,it 指向begin + 1位置
```
#### const 成员函数
```cpp
class TextBlock
{
public:
    const char &operator[](std::size_t position);     // 返回一个常引用
    TextBlock &operator=(const TextBlock &textblock); // 形参使用const 使传入的对象(值)不会被更改
    const int getValue() const
    {
        value_1 += 1; // error,value 是 non-const, const函数不可修改non-const成员属性
        value_2 += 1; // 成功,const 成员函数可修改mutable 修饰的成员属性
    }
private:
    std::string text;
    int value_1 = 1;
    mutable int value_2 = 2; // mutable 修饰的成员属性可被const 成员函数访问并修改
};
```
---

***update :  2024. 12. 16***
#Chapter1_object_init
### 确定对象被使用前已经初始化
#### 保证初始化内嵌类对象
```cpp
class A
{
public:
    A(int value) : value_1(value) {}
private:
    int value_1;
};

// 构造函数初始化对象尽可能使用初始化列表来初始化成员属性, 且列表顺序跟类内声明顺序相同
class B
{
public:
    B(int value) : value_2(value), a(0) {} // 保证在使用对象时初始化其 内嵌类成员
private:
    int value_2;
    A a;
};
```
#### *避免跨编译单元初始化次序的问题*
***问题 :***  在编译多个.cpp文件过程中, 可能存在non-local static 即 ***全局静态或静态类成员变量***   
在初始化顺序不当时，会存在使用未初始化的全局静态变量, 这会导致未定义的行为  

***解决办法 :*** 将这些 non-local static 搬到独属于自己的函数中, 这样，当我们不调用函数时，改变量将  不会被创建，这就避免了使用次序错乱而导致的未定义行为 
```cpp
class Point
{
public:
    int x;
    int y;
    Point(int x, int y) : x(x), y(y) {}
    Point &getInstance() // 得到一个全局静态实例,用于调用类中的成员
    {
        static Point p(0, 0); // 将 non-local static p 包裹在函数中
        return p;
    }
};
```
---
