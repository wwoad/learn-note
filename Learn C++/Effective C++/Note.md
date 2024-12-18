---
cssclasses:
  - "2024."
  - "12."
  - "16"
---
---
***update : 2024. 12. 16***
#Note_enum_class
## *enum & enum class*
#### 作用域
`enum : ` 定义的枚举成员会直接进入外部作用域, 可能会与其他变量或符号发生冲突 .  
`enum class : ` 定义的枚举成员被限制在枚举类型的作用域内, 不会污染外部作用域.  
```cpp
enum A 
{
    Value_1 = 1
};
enum class B 
{
    Value_2 = 1
};
A value = Value_1;    // 可在外部作用域访问到
B value = B::Value_2; // 只能被限制在枚举类型的作用域内
```
#### 类型安全
`enum : `没有类型安全, 允许将枚举成员赋值给任何整数类型, 甚至可以与其他枚举类型的值进行混合 .  
`enum class : `提供了更强的类型安全, 不允许直接将不同枚举类型的值互换, 或者将枚举值赋给整数类型
```cpp
enum A
{
    Value_1 = 1
};
int a = Value_1;    // 编译成功, 类型不安全

enum class B
{
    Value_1 = 1
};
int b = B::Value_1; // error, 类型安全
```
#### 底层类型 
***enum 或者 enum class 只能显示指定为整型 int, char, unsigned char, unsigned int***  
`enum : `底层类型是隐式的，通常为 `int`(但可以使用`enum : underlying_type`来显式指定底层类型）
`enum class : `允许显式指定底层类型。如果不指定，默认底层类型是 `int`
```cpp
// 整型: int, unsigned int,  char, unsigned char
enum A : int                 // 只能显示指定为整型
{
    Value_1 = 1
};
enum class B : unsigned char // 只能显示指定为整型
{
    Value_1 = 1
};
```
#### 隐式转换
 `enum : `枚举值会隐式转换为对应的整数类型, 这可能导致意外的行为    
`enum class : `不允许隐式转换, 必须显式地将其转换为底层类型
```cpp
enum A 
{
    Value_1 = 1
};
enum class B
{
    Value_1 = 1
}
int a = Value_1;                      // 存在隐式转换,成功编译
int b = B::Value_1;                   // error,不存在隐式转换
int b = static_cast<int>(B::Value_1); // 只支持显示转换,成功编译

```

---
***update :  2024. 12. 16***



---




---



---

