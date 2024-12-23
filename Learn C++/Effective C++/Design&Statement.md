
---
***update : 2024. 12. 22***
### 使得接口容易被使用, 避免不必要的错误
#### 避免错误的构造或对传入进行合法性检查
当类构造时, 很有必有对传入的值进行合法性检查, 防止创建错误的对象, 造成隐患
```cpp
class Car
{
public:
    Car(int speed, int weight) : speed(speed), weight(weight)
    {
        if (speed <= 0 || weight <= 0) // 输入合法控制
        {
           // 若输入非法,则运行报错
            throw std::invalid_argument("Speed and weight must be positive!");
        }
    }
}
private:
    int speed;
    int weight;
};
void fun()
{
    Car c(0,0); // error, 传入值非法
}
```
#### 使得接口易用
下列例子中，若对`CarType`使用`std::string`, 这使得接口难以理解且易出错  
使用`enum class` 类型安全的方式, 不仅使得构造清晰易用且不容易出错
```cpp
enum class CarType // 使用类型安全的enum class控制 Car 的类型, 使得Car初始化接口易于理解
{
    AA,
    BB,
    CC,
    DD
};
class Car
{
public:
    Car(CarType type, int speed, int weight) : type(type), speed(speed), weight(weight){ //.... }
private:
    CarType type;
    int speed;
    int weight;
};
void fun()
{
    Car c(CarType::AA, 1, 1); // 创建一个清晰且正确的对象
}
```
#### 接口的权限控制
使用`const`控制成员函数的权限
```cpp
class Car
{
public:
    Car(CarType type, int speed, int weight) : type(type), speed(speed), weight(weight)
    {//...}
    void doSomething() const {} // 使用 const 约束函数不能更改成员属性
private:
     //...
};
```
---
***update : 2024. 12. 22***
### 传引用代替传值
值传递在数据过大时, 可能影响性能, 而引用就能避免这一问题
```cpp
class Base
{
public:
    std::vector<int> v;
    // Base(std::vector<int> v) : v(v) {}     // 性能受影响
    Base(const std::vector<int> &v) : v(v) {} // 使用const &可节省多余的拷贝操作,性能更好 
};
```
---
***update : 2024. 12. 22***
### 必须返回对象时, 不要返回对象的引用
如果真要返回一个对象, 不可返回局部对象的引用, 因为这试图返回一个已删除对象的地址  
由于**RVO**(返回值优化)的存在, 并不会造成额外的空间资源浪费和性能问题
```cpp
std::string& get()       // 返回局部对象的引用,error
{
    std::string str = "hello world";
    return str;
}
std::string get()       // 正确,返回一个副本
{
    std::string str = "hello world";
    return str; // 返回副本,而不是 &
}
```
---
***uodate : 2024. 12 .23***
### 将成员变量声明为`private`
为了封装性的原则, 成员变量最好声明为`private`, 访问使用`get`方法,更改使用`set`方法 , 这样统一接口
```cpp
namespace Graphics
{
    class Rectangle
    {
    public:
        Rectangle() = default;
        float getHeight() { return height; } // get方法访问private成员变量
        float getWidth() { return width; }
        void setHeight(const float &height); // set方法更改private成员变量
        void setWidth(const float &width);
    private:
        float height;
        float width;
    };
}
```
---
***update :2024. 12. 23***
### 成员(功能)函数与友元函数尽量变为非成员函数(`non-member`)和非友元函数(`non-friend`)
#### 使用`non-member`代替成员函数`member`
```cpp
class Rectangle
    {
    public:
        Rectangle() = default;
        float getHeight() { return height; }
        float getWidth() { return width; }
        void setHeight(float height) { this->height = height; }
        void setWidth(float width) { this->width = width; }
        // float area() { return height * width; } // 求面积的功能函数
    private:
        float height; // 高
        float width;  // 宽
    };
    float rectangleArea(Graphics::Rectangle &r) // non-member 求rectangle面积 
    { 
        return r.getWidth() * r.getHeight(); 
    }
```
- **松耦合**, 使得函数和类之间的耦合度较低, 低耦合的设计使得代码更容易理解和维护
-  **增强的重用性**, `non-member`是类外部的独立函数, 它们可以作用于多个类, 而不需要为每个类都提供一个成员函数
- **避免多继承带来的问题**, 如果将功能放在非成员函数中，就避免了成员函数多次继承的复杂性
#### `non-member`适用的场景
**当功能需要同时与多个类进行交互时**, 将该功能作为非成员函数比在每个类内部都实现成员函数要更合适. 通过非成员函数, 可以避免将通用功能绑到某个特定类中  
**如果函数不依赖于类的内部状态或数据，那么应该将其设计为非成员函数**  
使用`nameapeca`封装对应的类和`non-member`函数利于使用和理解
```cpp
namespace Graphics // 命名空间,封装对应的类和方法
{
    class Rectangle //矩形类
    {
    public:
        Rectangle() = default;
        float getHeight() { return height; }
        float getWidth() { return width; }
        void setHeight(float height) { this->height = height; }
        void setWidth(float width) { this->width = width; }
        // float area() { return height * width; } // 求面积的功能函数
    
    private:
        float height; // 高
        float width;  // 宽
    };
    
    class Circle //圆类
    {
    public:
        float getRadius() { return radius; }
        void setRadius(float r) { this->radius = r; }
    
    private:
        float radius; // 半径
    };
    // 求矩形面积,单类使用,传入对象即可实现功能
    float rectangleArea(Graphics::Rectangle &r) 
    { 
        return r.getWidth() * r.getHeight(); 
    }
    // 求两图形总面积,多类使用,传入多类的对象实现相应功能
    float totalArea(Graphics::Rectangle &r, Graphics::Circle &c)
    {
        return (r.getWidth() * r.getHeight() + 
        3.14 * c.getRadius() * c.getRadius());
    }
}
```
---




















---
