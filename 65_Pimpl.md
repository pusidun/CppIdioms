# Pointer To Implementation

看一下下面的类

book.h
```
class Book {
public:
    Book();
    ~Book();
    void print();

private:
    std::string _name;
};
```

如果我们想修改Book类，比如添加一个成员std::_content，那么所有include了Book.h头文件的模块都需要重新编译。因为Book的结构发生了更改会影响所有使用了该类的模块。

我们可以使用Pimpl惯用法，将**类数据成员替换成一个指向具体实现的类**，将放在主类的数据成员放在实现类里。

```
// Book.h
class Book {
public:
    Book();
    ~Book();
    std::string name();

private:
    struct Pimpl;
    Pimpl* const _pimpl;
};

// Book.cpp
#include <string>
#include "Book.h"

struct Book::Pimpl{
    std::string name;
};

Book::Book():_pimpl(new Pimpl){};

Book::~Book(){
    delete _pimpl;
}

std::string Book::name() {
    return _pimpl->name;
}

// main.cpp
#include <iostream>
#include "Book.h"

int main() {
    Book book;
    std::cout<<book.name()<<std::endl;
}
```

这就是PIMPL的基本用法，通过在Book类里声明**未实现类**Pimpl的指针，实现了将依赖从头文件转到cpp文件。
当需要新增或者修改成员时，只需要修改Book::Pimpl即可，Book本身结构并没有改变，包含Book.h头文件的模块也不用重新编译了。

## Modern C++

我们可以使用std::unique_ptr替代原始指针,构造函数中使用make_unique初始化该智能指针

```
// Book.h
#include <memory>

class Book {
public:
    Book();
    ~Book();
    std::string name();

private:
    struct Pimpl;
    std::unique_ptr<Pimpl> _pimpl;
};

// Book.cpp
#include <string>
#include <memory>
#include "Book.h"

struct Book::Pimpl{
    std::string name;
};

Book::Book():_pimpl(std::make_unique<Pimpl>()){};

Book::~Book(){}

std::string Book::name() {
    return _pimpl->name;
}
```
