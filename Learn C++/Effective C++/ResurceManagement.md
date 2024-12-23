___
***update : 2024. 12. 20***
### 尽量使用智能指针管理资源生命周期
#### 传统资源管理
传统的`new`和`delete`的创建与释放过程中,会因为潜在存在的提前`return`或是`continue` ,`goto`等, 导致资源得不到释放导致内存泄露的弊端
```cpp
void fun1()
{
    Base *ptr = new Base("name", 18);
    // ....             
    if (ptr != nullptr) // 若上面过程中存在return或是抛出异常
        delete ptr;     // 导致fun提前结束, 则ptr不会被正确释放
    return;
}
```
#### 使用智能指针管理资源
**智能指针 :** `unique_ptr` 与 `shared_ptr`等管理资源, 在指针的生命周期结束时, 会被自动释放掉
```cpp
void fun2()
{
    // 使用unique_ptr 管理资源
    std::unique_ptr<Base> ptr = std::make_unique<Base>("name", 18);
}
```

---
***update :2024. 12. 20***
### 提供对底层资源的访问
在实际的设计中, 为了灵活性和性能, 还有希望直接修改底层类的数据的需求, 提供直接的访问并修改底层类中数据的函数是由必要的
#### 传统设计思路
类提供`get`接口 -> 外部函数修改底层数据 -> 原始数据更新 -> 使用修改后的数据
```cpp
class Base
{
public:
    Base(std::vector<std::string> v) : v(v) {}
    std::vector<std::string> *getV() { return &v; } // 提供访问原始成员数据的函数
private:
    std::vector<std::string> v;
};
void fun()
{
    std::vector<std::string> v;
    v.emplace_back("aa");
    std::unique_ptr<Base> ptr = std::make_unique<Base>(v);
    ptr->getV()->emplace_back("bb");                     // 直接访问并修改类的原始资源
    std::cout << *(ptr->getV()->begin() + 1) << std::endl; // 访问修改后的原始数据
    // output bb
}
```
**缺点 :** 上述代码中会破坏封装性的原则, 还有当用户修改了底层数据, 而资源管理类并没有同步更新数据状态时, 会造成数据不一致的情况, 因此上述的代码还是存在缺陷的
#### 现代设计思路
**优点 :** 智能指针自动管理资源, 提供get方法访问原始指针,如`std::unique_ptr<int[]> ptr`返回`ptr.get()`即返回`int*` ,内存泄露风险低，智能指针支持线程安全的资源管理, 防止竞争条件

```cpp
class Base
{
public:
    Base(std::vector<std::string> &v) 
    { pv = std::make_unique<std::vector<std::string>>(v); }
    std::vector<std::string> *getV() { return pv.get(); } // get 返回原始指针std::vector<std::string>*
private:
    std::unique_ptr<std::vector<std::string>> pv; // 使用智能指针管理资源
};
void fun1()
{
    std::vector<std::string> v;
    v.emplace_back("a");
    Base b(v);
    std::cout << *b.getV()->begin() << std::endl; // 访问原始数据
    // output a
    b.getV()->emplace_back("b"); // 修改原始数据
    std::cout << *(b.getV()->begin() + 1) << std::endl;
    // output b
}
```
----