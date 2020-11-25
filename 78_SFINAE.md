# Substitution Failure Is Not An Error

发生于模板函数重载决议。

当一个函数名称和某个函数模板名称匹配时，重载决议过程大致如下：

1. 根据名称找出所有适用的函数和函数模板对于适用的函数模板，要根据实际情况对模板形参进行替换；

2. 替换过程中如果发生错误，这个模板会被丢弃

3. 在上面两步生成的可行函数集合中，编译器会寻找一个最佳匹配，产生对该函数的调用

4. 如果没有找到最佳匹配，或者找到多个匹配程度相当的函数，则编译器需要报错。

步骤2即Substitution Failure，但这不是一个会让程序退出的错误(Error)，编译器会继续寻找下一个最佳匹配。

## SFINAE与enable_if

std::enable_if有2个模板参数，当第一个为true时有typedef成员type，和T类型相同；否则没有

很容易实现我们自己的版本：

```
template<bool B, typename T = void>
struct my_enable_if {};

template <typename T>
struct my_enable_if<true, T> {
    typedef T type;
};
```