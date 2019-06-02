---
layout:     post
title:      C++ Primer 学习笔记(八)
subtitle:   C++ Primer 学习记录(八)
date:       2019-05-31
author:     王鹏程
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - C++
    - 基础编程
---

## 第15章 面向对象程序设计

### 15.1 oop:概述
面向对象程序设计的核心思想是数据抽象、继承和动态绑定。

**虚函数：** 基类希望它的派生类各自定义适合自身版本，将这些函数声明为**虚函数**；派生类必须通过派生类列表明确指明他是从那个基类继承而来的。即 **基类希望派生类能进行覆盖的函数**

**动态绑定**
通过**动态绑定**，我们能用同一段代码分别处理不同的对象。相同函数，根据动态绑定的对象实质进行区别。在运行时选择函数的版本，所以动态绑定有时又被称为 **运行时绑定**

注意：

- 在c++语言中，当我们使用基类的引用(或者指针)调用一个虚函数时将发生动态绑定。

### 15.2 定义基类和派生类

注意：

- 基类同城都应该定义一个虚析构函数，计时该函数不执行任何实际操作也是如此。
- 如果一个派生类没有覆盖其基类中的某个虚函数，则该函数的行为类似于其他成员，派生类会直接继承其在基类中的版本。
- 同一个对象中，继承自基类的部分和派生类自定义的部分不一定是连续存储的。

```c++
class Quote{
public:
    Quote()=default;
    Quote(const std::string &book,double sales_price):bookNo(book),price(sales_price){}
    std::string isbn() const {return bookNo;}
    //返回给定数量的书籍的销售总额
    
    //派生类负责改写并使用不同的折扣计算算法
    
    virtual double net_price(std::size_t n) const
    {return n*price;}

    virtual ~Quote()=default; //对析构函数进行动态绑定
private:
    std::string bookNo;  //书籍编号

protected:
    double price=0.0   //普通状态下不打折扣的价格 

}

```

可以将派生类的对象当成基类来使用，而且也能将基类的指针或者引用绑定到派生类对象中的基类部分上。

```c++
Quote item;  //基类对象

Bulk_quote bulk;  //派生类对象

Quote *p=&item;  //p指向Quote对象

p=&bulk; //p指向bulk的Quote的部分

Quote &r=bulk;  //r绑定到bulk的Quote部分

```
上述转换可以把派生类对象的指针用在需要基类指针的地方。
注意：

- 在派生类对象中含有与基类对应的组成部分，这一事实是继承的关键。
- 派生类不能直接初始化继承的基类成员，必须使用基类的构造函数来初始化它的基类部分；每个类控制它自己的成员初始化过程。
- 除非我们特别指出，否则派生类的基类部分会像数据成员一样执行默认初始化。如果需要使用基类的构造函数需要，使用`基类名(参数1，参数2)` 的形式进行显式调用。否则进行一般空参数的默认初始化。
- 首先初始化基类的部分，然后按照声明的顺序依次初始化派生类的成员。
-  **c++中类，是在实例化时才会查找相关代码，没有使用就不会生成对应代码，函数无论使用都会生成。**
-  类会自动生成一个`namespace`,其中的静态成员和静态变量，相当于`namespace`中的变量和函数。
-  如果基类定义了一个静态成员，则在整个继承体系中只存在该成员的唯一定义。不论从基类中派生出来多少个派生类，对于每个静态成员来说都只存在唯一的实例。
-  如果我们想要某个类用作基类，则该类函数必须已经定义而非仅仅声明。主要是构造函数和成员变量必须定义，因为子类的构造函数必须使用父类的构造函数。
-  在类后面添加关键字`final`可以有效防止类被继承。
-  和内置指针一样，智能指针类也支持派生类向基类的类型转换，意味着我们可以将一个派生类对象的指针存储在一个基类指针的只能指针内。

**基类和派生类**

不存在基类向派生类的隐式类型转换，但是当编译器无法确定某个特定的转换在运行时是否安全的时候，则可以，但这是很危险的，并且基类函数的析构函数最好是虚析构函数。([C++中虚析构函数的作用](https://www.cnblogs.com/lit10050528/p/3874703.html))

```c++
Bulk_quote bulk;
Quote *itemP=&bulk;  //正确；动态类型是Bulk_quote

Bulk_quote *bulkP=itemP;  //错误，不能将基类转换成派生类

```

派生类向基类的自动类型转换，支队指针或者引用类型有效，在派生类类型和基类类型之间不存在这样的转换。

当我们用一个派生类对象为一个基类对象初始化或赋值时，只有该派生类对象中的基类部分会被拷贝、移动或赋值，它的派生类部分将被忽略掉。

存在继承关系的类型之间的转换规则

- 从派生类向基类的类型转换只对指针或引用类型有效
- 基类向派生类不存在隐式类型转换
- 和任何其他成员一样，派生类向基类的类型转换也可能会由于访问受限而变得不可行。

### 15.3 虚函数

**虚函数的调用可能在运行时才被解析**

注意：

- 一旦某个函数被声明成虚函数，则在所有派生类中它都是虚函数。
- 一个派生类的函数如果覆盖了继承来的虚函数，则它的形参类型必须被它覆盖的基类函数完全一致;返回类型也必须相同。形参列表不同时会产生新的函数，继承的基类函数仍旧有效。
- 使用`override`关键字可以明确重载，原函数中没有函数，或者参数不对应则都会产生错误。
- 如果虚函数使用默认实参，则基类和派生类中定义的默认实参最好一致。

**回避虚函数的机制**

可以使用作用域运算符，实现虚函数的强行绑定，而非动态绑定；例如：

```c++
double undiscounted=baseP->Quote::net_price(42);
//强行调用基类中定义的函数版本而不管baseP的动态类型到底是什么

```

通常情况下，只有成员函数(或者友元)中的代码才需要使用作用域运算符来回避虚函数的机制。通常是一个派生类的虚函数调用它覆盖的基类的虚函数版本时。

注意：如果一个派生类虚函数需要调用它的基类版本，但是没有使用作用域运算符，则在运行时该调用将被解析为对派生类版本自身的调用，从而导致无限递归。

### 15.4 抽象基类

含有纯虚函数的类是抽象基类；不能创建抽象基类的对象，只能被继承

重构：负责重新设计类的体系以便将操作或数据从一个类移动到另外一个类中。

### 15.5 访问控制与继承

protect:希望派生类分享但是不想被其他公共访问使用的成员。

- 受保护的成员对于类的用户来说是不可访问的。
- 受保护的成员，对于派生类的成员和友元来说是可以访问的
- 派生类或友元只能通过派生类对象来访问基类的受保护成员。派生类对于一个基类的受保护成员没有任何访问特权。

```c++
class Base{
protected:
    int prot_mem;  //protected 成员

};
class Sneaky:public Base{
    friend void clobber(Sneaky&);  //能访问Sneaky::prot_mem

    friend void clobber(Base&);  //不能访问Base::prot_mem

    int j;  
}
void clobber(Sneaky& s) {s.j=s.prot_mem=0;} //正确能访问Sneaky对象的private和protected成员

void clobber(Base& b){b.prot_mem=0;}//错误不能访问protected的成员

```
private 不影响派生类的访问权限，主要影响，相关函数的使用。

**派生类向基类转换的可访问性**

- 只有当D公有地继承B时，用户代码才能够使用派生类向基类的转换；如果D继承B的方式是保护的或者私有的，则用户代码不能使用该转换。
- 不论D以什么方式继承B,D的成员函数和友元都能使用派生类向基类的的转换；派生类向其会直接基类的类型转换对于派生类的成员和友元来说是永远可以访问的。
- 如果D继承B的方式是公有的或者受保护的，则D的派生类的成员和友元可以使用D向B的类型转换；反之，如果D继承B的方式是私有的，则不能使用
- 对于代码中的某个给定节点来说，如果基类的公有成员是可以访问的，则派生类向基类的类型转换也是可访问的；反之则不行。

**友元与继承**

友元关系不能继承，友元关系也不能传递，基类的友元在访问派生类成员时，不具有特殊性，类似的，派生类的友元也不能随意访问基类的成员。-- **不能继承友元关系，每个类负责控制各自成员的访问权限**

**改变个别成员的可访问性**

通过`using`声明可以改变派生类继承的某个名字的访问级别。

```c++
class Base{
public:
    std::size_t size() const {return n;}
protected:
    std::size_t;
};

class Derived: private Base{
public:
    using Base::size;  //保持对象尺寸相关的成员的访问级别

protected:
    using Base::n; //使用using关键字改变成员变量的访问级别。

};

```

- `private using` 该名字能被类的成员和友元访问；
- `public using` 类的所有成员都能访问。
- `protectde using` 类的成员、友元和派生类是可访问的。

### 15.6 继承中的类作用域

每个类定义自己的作用域，；当存在继承关系时，派生类的作用域嵌套在其基类的作用域之内。如果一个名字在派生类的作用域内无法正常解析，则编译器将继续在外层的基类作用域中寻找该名字的定义。例如：

```c++
Bulk_quote bulk;
cout<<bulk.isbn();

//查找步骤：先找自身作用域内函数，再找父类，和父类的父类

```
静态类型：在编译时总是已知的，它是变量声明时的类型或表达式生成的类型
动态类型：变量或表达式表示的内存中的对象的类型。

注意：

- 派生类的成员将隐藏同名的基类成员
- 可用通过作用域运算符来使用一个呗隐藏的基类成员
```c++
struct Derived:Base{
    int get_base_mem(){
        return Base::mem;
    }
}

```

- 除了覆盖继承而来的虚函数之外，派生类最好不要中庸其它定义在基类中的名字。

**关键概念:名字夜找与继承**

------

理解函数调用的解析过程刘一于理解C++的继承至关重要，假定我们调用`p->mem()`(或者obj.mem())，则依次执行以下4个步骤:

- 首先确定p(或obj)的静态类型因为我们调用的是一个成员，所以该类型必然是类类型    
- 在p(或。bj)的静态类型对应的类中查找mem如果找不到，则依次在直接基类中不断查找直至到达继承链的顶端如果找遍了该类及其基类仍然找不到，则编译器将报错
- 一旦找到了mem,就进行常规的类型检查(参见6.1节，第183页)以确认对于当前找到的mem，本次调用是否合法
- 假设调用合法，则编译器将根据调用的是否是虚函数而产生不同的代码:      一如果mem是虚函数且我们是通过引用或指针进行的调用，则编译器产生的代          码将在运行时确定到底运行该虚函数的哪个版本，依据是对象的动态类型、      一反之，如果mem不是虚函数或者我们是通过对象(而非引用或指针)进行的          调用，则编译器将产生一个常规函数调用。

```c++
struct Base{      
int  memfcn();
};
struct DerivPd:Base{
    int memfcn(int);  //隐藏基类的memfn

};

Derived d;Base b;

b.memfcn();  //调用Base::memfn

d.memfcn(10);  //调用Derived::memfcn

d.memfcn();   //错误：参数列表为空的memfcn被隐藏了

d.Base::memfcn();  //正确：调用Base::memfcn()

```
**通过基类调用隐藏的虚函数**

```c++
class Base{
public:
    virtual int fcn();
};
class D1:public Base{
public:
    //隐藏基类的fcn,这个fcn不是虚函数
    
    //D1继承了Base::fcn()的定义

    int fcn(int);  //形参列表与Base中的fcn不一致

    virtual void f2();  //是一个新的虚函数，在Base中不存在
};

class D2:public D1{
public:
    int fcn(int);   //一个非虚函数，隐藏了D1::fcn(int)

    int fcn();    //覆盖了Base的虚函数fcn

    void f2();  // 覆盖了D1的虚函数f2
}



Base bobj;
D1 d1obj;
D2 d2obj;

Base *bp1=&bobj,*bp2=&d1obj,*bp3=&d2obj;

bp1->fcn();  //虚调用，将在运行时调用Base::fcn()

bp2->fcn();  //虚调用，将在运行时调用Base::fcn()

bp2->fcn();  //虚调用，将在运行时调用D2::fcn()

Base *pd=&d2obj;
D1 *p2=&d2obj;
D2 *p3=&d2obj;

p1->fcn(42);  //错误：Base中没有接受一个int的fcn

p2->fcn(42);  //静态绑定，调用D1::fcn(int)

p3->fcn(42);  //静态绑定，调用D2::fcn(int)

```

类内using声明的一般规则同样适用于重载函数的名字，基类函数的每个实例在派生类中都必须是可访问的，对派生类没有重新定义的重载版本的访问，实际上是对using 声明点的访问。

### 15.7 构造函数与拷贝控制

虚析构函数将阻止合成移动操作：
如果一个类定义了析构函数，即使它通过`=default`的形式使用了合成的版本，编译器也不会为这个类合成移动操作。

**派生类中删除的拷贝控制与基类的关系**

某此定义基类的万式也可能导致有的派产仁类成员成为被删除的函数：

- 如果基类中的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是被删除的函数或艺不可访问，则派生类中对应的成员将是被删除的，原因是编译器小能使用基类成员来执行派生类对象基类部分的构造、赋值或销毁操作。
- 如果在基类中有一个不可访问或删除掉的析构函数，则派生类l}，合IJ}G的默认和拷贝  构j查函数将是被删除的，因为编译器无法销毁派生类对象的基类部分。
- 和过去一样，编译器将不会合成一个删除掉的移动操作。当我们使用=default请  求4个移动操作时，如果基类中的对应操作是删除的或不可访问的，那么派生类中该函数将是被删除的，原囚是派生类对象的墓类部分不可移动卜」样，如果基类的析构函数是删除的或小，访问的，则派牛类的移动构造函数也将是被删除的。

注意：

- 当派生类定义了拷贝或者移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个对象。
- 在默认情况下，基类默认构造函数初始化派生类对象的基类部分如果我想拷贝(或移动)基类部分，则必须在派生类的构造函数初始值列表中显地使用基类的拷贝(或移动)构造函数。
- 如果构造函数或析构函数调用了某个虚函数，则我们应该执行与构造函数或析构函数所属类型相对应的虚函数版本一。
- 构造函数的using 声明不会改变该构造函数的访问级别。
- 基类构造函数的默认实参，并不会被继承。相反，派生类将获得多个继承的构造函数，其中每个构造函数分别省略掉一个含有默认实参的形参。如果一个构造函数接收两个形参后一个由模式实参，则派生类将获得两个构造函数一个函数接受两个形参(没有默认实参)，另外一个构造函数只接受一个形参，它对应于基类中最左侧的没有默认值的那个形参。
- 如果派生类定义的构造函数与基类的构造函数具有相同的参数列表，则该构造函数将不会被继承。定义在派生类中的构造函数将替换继承来的构造函数
- 继承的构造函数不会被作为用户定义的构造函数来使用，如果一个类只含有继承的构造函数，则他也将拥有一个合成的默认构造函数。

### 15.8 容器与继承

**当派生类对象呗赋值给基类对象的时候，其中的派生部分将被“切掉”，因此容器和存在继承关系的类型无法兼容；因此最好在容器中放置(智能)指针而非对象**

**模拟拷贝**
给类添加一个虚拷贝函数，该函数将申请一份当前对象的拷贝。处理动态的内存分配

```c++
class  Quote{
public:
    //该虚函数返回当前对象的一份动态分配的拷贝
    
    virtual Quote* clone() const& {return new Quote(*this);}
    virtual Quote* clone() && {return new Quote(std::move(*this));} 
}

```


## 第16章 模板与泛型编程










