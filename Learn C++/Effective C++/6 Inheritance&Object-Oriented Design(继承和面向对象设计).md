
---
***update :2024. 12. 25***
### 使`public`继承符合`“is-a”`原则
- **子类具有父类的所有属性和行为**：通过继承, 子类应该具备父类的所有特性, 包括数据成员和成员函数, 并能正确地使用这些功能  
- **子类可以替代父类**：子类的对象可以作为父类的对象使用, 且它应该具备和父类相同的行为. 即父类指针或引用应该能够指向子类对象  
**符合`“is-a”`原则的情况**
```cpp
class Shape // base 类
{
public:
    virtual void draw() const = 0;
    virtual ~Shape() = default;
};
//----------------------------------
class Circle : public Shape  // 圆派生类
{
public:
    // 重写draw函数 
    void draw() const override { std::cout << "circle" << std::endl };
};
```
**不符合`“is-a”`原则的情况**
```cpp
class Square // 正方形父类
{
public:
    virtual int getSideLength() const = 0;
private:
    int sideLength;
};
//----------------------------------
class Rectangle : public Square  // 矩形派生类
{
public:
    int getWidth();
    int getHeight();

private:
    int width;
    int height;
};
```
矩形类的属性和行为并不能当作父类来使用, 因此不符合`“is-a”`原则
##### 如何遵循 `"is-a"` 原则？
**确保子类是父类的特殊化:** 子类应该是父类的一种类型, 具有父类的行为和属性. 子类的功能是对父类功能的扩展或细化, 而不是完全不同的功能  
**继承应该遵循逻辑一致性:** 子类的行为应当符合父类的行为, 而不是添加一些完全不相关的行为. 例如，不应该让 `Rectangle(矩形)` 继承 `Square(正方形)`, 因为它们在逻辑上并不属于同一类型.  
**避免滥用继承:** 继承应当只用于表示 `is-a”` 的关系。如果存在“有一个`(has-a)`或“可以是”`(can-be)`关系时, 应该考虑使用其他设计模式 (如组合、接口等) 来替代继承

---
***update: 2024. 12. 25***
### 避免隐藏继承而来的变量跟函数
若继承过程中存在子类变量和函数跟父类相同, 在调用过程中, 可能会造成**隐藏**了父类或子类函数和变量, 造成不符合期望的现象
#### 父类与子类变量和函数名应该避免相同
```cpp
class Base // 基类
{
public:
    std::string getStr_1(); // 函数名不一样
private:
    std::string str1;      // 变量名不一样
};
//-------------------------------
class Son : public Base  // 派生类
{
public:
    std::string getStr_2();
private:
    std::string str2;
};
```
#### `override`和`final`在子类多态重写父类函数中的应用

可以使用 `override` 关键字来明确表明子类成员函数覆盖父类成员函数，而不是意外地隐藏它们
```cpp
class Base 
{
public:
    virtual std::string getStr() const {}; 
private:
    std::string str0;
};
//---------------------------------
class Son : public Base
{
public:
    std::string getstr() const override {} // error,函数名不相同,重写失败
    
    std::string getStr() const override {} // 重写成功,重写Base的get函数,使用override标记
private:
    std::string str1;
};
```
可以使用`final`控制继承, 明确指出某个类不能被继承或者某个成员函数不能再重写  
显示避免了因为函数名相同可能发生的问题
```cpp
class Base final
{}
class Son:public Base  // error, final标记Base不能被继承
{}
```
**`final`必须跟`virtual`一起使用**   
`virtual`用来标记函数是虚函数, 允许子类重写这个函数, `final`用做`virtual`的修饰符, 表示父类的`virtual`函数的方法不可被修改
```cpp
class Base
{
public:
    virtual std::string getStr() const final { return str_0; }
private:
    std::string str_0;
};
//-------------------------------------
class Son1 : public Base
{
public:
    std::string getStr() const override { return str_1; } // error,父类的get函数声明标记为final,不能被子类重写,即父类的get方法不能被修改了
private:
    std::string str_1;
};
```
---
***update :2024. 12. 26***
### 区分接口继承和实现继承
#### 接口继承
父类提供方法的声明,标记为纯虚函数`virtual`即不作实现  
子类继承父类的**接口**, **必须重写并实现对应的功能**
```cpp
class Base
{
public:
    virtual void print() const = 0; // print接口,不作实现
};
//---------------------
class Son: public Base
{
public:
    void print() override const  // 接口继承,子类做出自己的实现
    {
        std::cout << "this is son" << std::endl;
    }
};
```
#### 实现继承
父类 既提供**实现**也提供**接口**, 子类可重写父类的接口也可调用或修改父类的实现
```cpp
class Shape
{
public:
    virtual void draw() const = 0;  // 提供接口
    float area() { }                  // 提供实现
};
//—------------------------
class Rectangle: public Shape
{
    void draw()const override  // 子类重写父类函数并实现
    { }
};
Rectangle r;
r.area();  // 子类调用父类实现
```
#### 总结
- **接口继承** 关注的是定义行为的规范, 子类需要根据这个规范来实现自己的功能  
- **实现继承** 关注的是复用父类的实现, 子类可以直接使用父类的功能，或者对父类功能进行扩展
---
***update : 2024. 12. 26***
### 能够替代`virtual`(多态)的设计方法
尽管虚函数是实现多态性的重要工具, 但它并不是唯一的解决方案.  根据具体的设计需求, **可能存在其他更高效或更适合的替代方案**  
`virtual`实现的多态有额外的性能开销, 且过度滥用使得代码复杂化,  在不需要实现**动态多态(运行时多态)** 的情况下, 可以综合考虑使用其替代方案
#### 传统的策略(`Strategy`)模式（经典实现）
传统的策略模式通常通过抽象类和继承来实现, 其中基类定义了一个接口(例如  `draw()` )派生类提供具体的策略实现, 与函数指针方案不同, 这种方法利用多态性, 但仍然避免了虚函数的滥用
```cpp
namespace Graphics
{
    class DrawShape // 绘制图形方法抽象基类
    {
    public:
        virtual void draw() const = 0;  //draw方法的pure virtual
    };
    // —------------------------
    class DrawRectangle : public DrawShape // 绘制矩形方法类
    {
    public:
        void draw() const { std::cout << "this is rectangle draw" << std::endl; }
    };
    // —------------------------
    class DrawCircle : public DrawShape // 绘制圆方法类
    {
    public:
        void draw() const { std::cout << "this is circle draw" << std::endl; }
    };
    // —------------------------
    class Shape // 具体的图形类
    {
    public:
        Shape(DrawShape *DrawShape) : draw(DrawShape) {}
        void drawShape()
        {
            std::cout << "start draw shape" << std::endl;
            draw->draw();
            std::cout << "end draw shape" << std::endl;
        }
    private:
        DrawShape *draw;
    };
}
//        test
Graphics::DrawRectangle r; // 创建两个方法实例
Graphics::DrawCircle c;

Graphics::Shape rectangle(&r); // 创建具体的图形对象
Graphics::Shape circle(&c);

rectangle.drawShape(); // 调用draw方法
circle.drawShape();

//output
/*
start draw shape
this is rectangle draw
end draw shape

start draw shape
this is circle draw
end draw shape
*/
```
**优点 :**
- 比函数指针方案更为结构化和类型安全。
- 适用于需要定义一组行为的场景, 能够利用继承和多态
- 与虚函数类似, 但更加灵活, 避免了过多的类层次结构。
#### 使用`NVI(Non-Virtual Interface)`手法实现模板方法
在基类中提供一个**非虚函数接口**，将虚函数的实现 “隐藏” 在子类的私有方法中.   
这样做的好处是可以**控制接口的使用**, 避免直接暴露虚函数, 同时还能利用静态多态性(如模板技术)来避免虚函数的运行时开销
```cpp
class Shape // 基类
{
public:
    void draw() // 提供non-virtual函数接口,调用private-virtual函数方法
    {
        std::cout << "start Base draw" << std::endl;
        doDraw();
        std::cout << "end Base draw" << std::endl;
    }
private:
    virtual void doDraw() const = 0; // draw方法,声明一个接口
};
// —------------------------
class Rectangle : public Shape
{
public:
private:
    // 重写base darw方法
    void doDraw() const override { std::cout << "this is rectangle dodraw" << std::endl; }
};
// —------------------------
class Circle : public Shape
{
public:
private:
    // 重写base darw方法
    void doDraw() const override { std::cout << "this is circle dodraw" << std::endl; }
};
//         test
Shape *s;
Rectangle r;
Circle c;
s = &r;
s->draw();
s = &c;
s->draw();

// output
/*
start Base draw     r的调用
this is rectangle dodraw
end Base draw

start Base draw     c的调用
this is cricle dodraw
end Base draw 
*/
```
#### 函数指针实现策略模式（`Strategy Pattern`）
**策略模式**是一种行为型设计模式, 它允许将不同的算法或行为封装成独立的类, 并在运行时选择使用哪个策略.  与虚函数相比, 策略模式可以通过函数指针或回调机制来实现灵活的行为选择  
需包含`#incliude<functional>`
```cpp
namespace Graphics
{
    using drawCallBack = std::function<void()>; // 定义一个函数指针类型

    class Rectangle // 矩形类
    {
    public:
        // 构造传入一个函数
        Rectangle(drawCallBack draw) : drawFunction(draw) {}
        void draw() const // 矩形类的draw接口调用对应实现
        {
            std::cout << "this is rectangle draw " << std::endl;
            drawFunction();  // 调用回调函数
        }
    private:
        drawCallBack drawCallBack;
    };
    // —------------------------------------
    class Circle  // 圆类
    {
    public:
        // 构造传入一个函数
        Circle(drawCallBack draw) : drawFunction(draw) {}
        void draw() const // 矩形类的draw接口调用对应实现
        {
            std::cout << "this is circle draw " << std::endl;
            drawFunction();
        }
    private:
        drawCallBack drawCallBack;
    };
    // 矩形类的draw实现
    void drawRectangle() { std::cout << "draw rectangle" << std::endl; }
    // 圆类的draw实现
    void drawCircle() { std::cout << "draw circle" << std::endl; }
}
//              test
Graphics::Rectangle r(Graphics::drawRectangle);
r.draw();
Graphics::Circle c(Graphics::drawCircle);
c.draw();
// output
/*
start rectangle draw     r的调用 
draw rectangle
end rectangle draw

start circle draw        c的调用 
draw circle
end circle draw
*/
```
**优点 :**
- 减少了虚函数表的开销, 性能较好  
- 灵活性高, 回调函数可以在运行时动态改变  
- 适合场景: 当行为可能变化，且不依赖于复杂的类层次结构时

---
***update: 2024. 12. 26***
### 绝不重新定义继承而来的`non-virtual`函数
**即派生类的non-virtual函数名不要跟父类的函数同名**
```cpp
class Base
{
public:
    void print() { std::cout << "this is base" << std::endl; }
};
class Son : public Base
{
public:
    // 相同函数名
    void print() { std::cout << "this is son" << std::endl; }
};
//          test
Son s;
s.print();
// output
// this is son

// 必须如此显式调用
s.Base::print();
s.Son::print();
// output
// this is base
// this is son
```
派生类的`non-virtual`函数跟父类相同会造成调用麻烦和使代码复杂化的缺点, 因此尽量不是使两功能相似的`non-virtual`函数同名

---
***update: 2024. 12. 26***
### 绝不修改继承而来的函数参数值
```cpp
class Base
{
public:
    virtual void print(bool print = true)
    {
        if (print)
            std::cout << "this is base" << std::endl;
    }
};
class Son : public Base
{
public:
    // 相同函数名
    void print(bool print = false) override
    {
        if (print)
            std::cout << "this is son" << std::endl;
    }
};
//           test
Son s;
Base *b;
b = &s;
b->print();
//  output
/*
this is son
*/
```
虚函数的重载和默认参数会有一些微妙的行为.  当定义一个虚函数并提供默认参数时, **这个默认参数是与函数的声明绑定的, 而不是与实际的调用对象绑定** . 因此，基类的默认参数(print = true)仍然会生效, 即使你在派生类中重载了该函数并将默认参数设为不同的值  
**这造成了与期望相反(打印`print failed`)的结果**

---
***update: 2024. 12. 26***
### 考虑使用类组合(`composition`) 代替继承
- **灵活性 :** 可以动态地改变组合对象的行为, 而不需要改变类的继承结构  
- **降低耦合 :** 避免了继承中的紧耦合问题, 一个类不需要关心继承层次中的其他类, 只需要关心它包含的对象的接口  
- **避免继承的副作用 :** 继承可能导致意外的行为, 尤其是在派生类重写了基类的方法时. 组合可以避免这一点, 因为每个对象都是独立的
```cpp
class Engine
{
public:
    void run() { std::cout << "engine run" << std::endl; }
};
class Car
{
public:
    void run()
    {
        std::cout << "car run" << std::endl;
        engine->run();
    }
private:
    Engine *engine;  // car内拥有engine的方法类
};

//     test
Car c;
c.run();
// output
/*
car run
engine run
*/
```
----
### 慎用`private`继承的
#### `private`继承不符合继承关系的`is-a`原则(影响多态)
继承原则的目的就是为了完成`"is-a"`的目标, 即子类能使用父类的函数和实现多态等, 如果将一个类`private`继承另一个类, 外部代码无法直接访问基类的公共成员, 这会破坏继承的语义
```cpp
class Base
{
public:
    virtual void print()
    {
        std::cout << "this is base" << std::endl;
    }
};
class Son : private Base  // private继承
{
public:
    void print() override
    {
        std::cout << "this is son" << std::endl;
    }
};
//         test
Son s;
Base *b = &s;
b->print();
// error
// 'Base' is an inaccessible base of 'Son'  
// 不允许对不可访问的基类 "Base" 进行转换
```
#### `private`继承的适用场景
当你希望表示实现继承 **(is implemented in terms of)** 时,  而不是表示“is-a”关系    
**实现继承:**  如果你希望派生类使用基类的实现而不是对基类功能进行扩写和实现多态, 但并不希望外部代码依赖于基类的接口, 可以使用私有继承. 这样, 你可以通过基类提供的功能来构建派生类的实现, 而不暴露基类的接口给外部用户
```cpp
class Engine
{
public:
    void run() { std::cout << "engine run" << std::endl; }
};
class Car : private Engine
{
public:
    void run()
    {
        std::cout << "car run" << std::endl;
        engine->run();  // 只是要基类的功能
    }

private:
    Engine *engine;
};
```
### 慎用多重继承
#### 复杂性增加
多重继承带来的是更复杂的类结构. 当一个类继承自多个基类时, 它必须处理来自各个基类的接口和实现.  开发者需要小心设计类的成员, 确保它们不会互相冲突或者产生**二义性** (多个基类中存在相同的方法或成员变量)
```cpp
class A
{
public:
    void fun() { std::cout << "A's fun" << std::endl; }
};
class B
{
public:
    void fun() { std::cout << "B's fun" << std::endl; }
};
class C : public A, public B  // 多重继承
{
public:
    void fun() { std::cout << "C's fun" << std::endl; }
};
//           test
C c;
c.fun();      // error,存在二义性
c.B::fun();   // 适用作用域解析符才能明确调用的函数
//  output    B's fun
```
#### 多重继承的适用场景
如果你希望**一个类同时拥有多个功能**，  并且这些功能**没有直接的层次关系**， 可以考虑使用多重继承.
```cpp
class Engine  // 引擎类
{
public:
    virtual void runEngine() = 0;
};
class Motor  // 电机类
{
public:
     virtual void runMotor() = 0;
};
class Car : public Engine, public Motor
{
public:
    void runEngine() override { std::cout << "engine run" << std::endl; }
    void runMotor() override { std::cout << "motor run"} << std::endl; }
    void run()
    {
        std::cout << "car run" << std::endl;
        runEngine(); // 引擎类的run方法
        runMotor();  // 电机类的run方法
    }
};
//          test
Car c;
c.run();
//  output
/*
car run
engine run
motor run
*/
```






