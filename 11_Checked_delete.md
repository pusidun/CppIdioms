# Checked delete

## 简介

删除一个动态分配的对象时，必须调用它的析构函数。如果这个类型是不完整的，即只有声明没有定义，那么析构函数可能会没被调用。这是一种潜在的危险状态，所以应该避免它。对于类模板及函数模板，风险会更大，因为无法预先知道会使用什么类型。

boost提供了checked_delete函数用来检测是否能安全delete

```
template<typename T>
inline void checked_delete(T* x) {
    typedef char type_must_be_complete[sizeof(x)?1:-1];
    (void)sizeof(type_must_be_complete);
    delete x;
}
```
利用sizeof,完成对参数T的检查。如果T是个未定义的类型,根据编译器sizeof会返回0或者报错。即使编译器返回0，也会因为定义一个-1大小的数组而报错。

`(void)sizeof(type_must_be_complete);`这一行是为了避免编译器优化掉type_must_be_complete数组