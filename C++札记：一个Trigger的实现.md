# C++札记：一个Trigger的实现

***提示***

------

阅读本文至少应了解C++的**class**, **function**及**lambda**相关知识，最好具有**template**及**variadic template**相关知识。



***问题***

------

日常编程中经常遇到一种需求：我想在某一特定条件下干一件事。也许是某个时间节点，也许是某个变量的状态，可能做一次，也可能反复做。



这种需求在代码调试过程中经常遇到，如果你很熟悉IDE，可以借助相应的debugger完成，但debugger的能力终究是有限的，当代码和你的需求变得复杂以后，debugger也会逐渐变得笨拙，这时你想要的是一个类似于触发器(Trigger)的工具。即使是在正常的程序运行过程中，对于Trigger的需求也是随处可见的，比如在某一时刻执行一次输出处理，在特定时间点修改边界条件，在关键位置报告执行状态，等等。



简单讲，Trigger可能就是一条if语句，让你觉得没有必要小题大做。但使用if语句需要解决单次执行的问题，承受if代码块及相应注释带来的逻辑混入，以及走到哪儿写到哪儿。这里我们希望用一种更优雅的方式解决这一需求。



***方案***

------

关于Trigger的实现，见仁见智，这里仅提供我的一些思考过程，供诸位参考。



首先想到的是定义一个Trigger类，并在类中提供operator()函数，让Trigger对象成为一个可调用对象(callable object)，这样原来的if语句就变成了Trigger对象的一次函数调用，含意明确，也不影响原代码的整体性。


```c++
// Trigger雏形
class Trigger
{
    ...
public:
    void operator()() { ... }
};

// 使用方式
Trigger tg(...);    // 定义触发器
while(...)
{
    ...
    tg();          // 使用触发器
    ...
}
```

现在问题的关键集中到了operator()函数，如何判断执行的条件？条件满足后该执行什么操作？前面说了，触发操作可能只需要执行一次，因此需要一个记数器，判断条件是由用户给定的，Trigger事先不可能知道，同样的还有执行操作，因此需要两个变量分别记录条件(Condition)和操作(Operation)，能够胜任这一功能的变量不是普通的int或double，而是std::function。下面给出一个实现版本：


```c++
代码看不完整请左右滑动
```

```c++
#include <functional>
class Trigger
{
private:
    std::function<bool(void)> m_cond;  // 条件函数
    std::function<void(void)> m_oper;  // 操作函数
    int m_count;  // 计数器，正数代表可执行的次数，0代表不再执行，负数代表可一直执行
    
public:
    // 构造函数
    Trigger(std::function<bool(void)> cond,
            std::function<void(void)> oper,
            int n = -1) :    // 构造时默认可一直执行
            m_cond(cond), m_oper(oper), m_count(n)
    {}
    
    void operator()() 
    {
        // 最先判断计数器情况，不满足直接结束，满足时再看条件函数是否为true
        if (m_count != 0 && m_cond())
        {
            m_oper();
            if (m_count > 0) --m_count;
        }
    }
};
```

写一个简单的测试

```c++
#include <iostream>
int main()
{
    int i = 0, j = 5, k = 0;
    Trigger tg([&]() {return i > j; },   // 判断条件为 i > j
        [&]() {                          // 执行操作为赋值并输出
            k = i + j;
            std::cout << "i = " << i << "\t";
            std::cout << "j = " << j << "\t";
            std::cout << "k = " << k << "\n";
        },
        1);                              // 仅执行1次
        
    for (i = 0; i < 10; ++i)
        tg();                            // 调用方式
    
    return 0;
}
```

输出结果为

![img](C++札记：一个Trigger的实现.assets/640)

说明只在i=6的时候执行了一次。如果改为一直执行，结果为

![img](C++札记：一个Trigger的实现.assets/641)



上述测试证明Trigger类的实现是正确的，基本上能够满足我们大部分的需求。对于条件函数而言，不接收任何参数而返回一个bool值是合理的，变量的传递可通过lambda捕捉实现，同理操作函数也可以用这种方式实现。除了示例中采用的lambda函数，其它合乎规定类型的函数指针或可调用对象均可作为Trigger构造函数的参数。



***进阶***

------

虽然bool(void)类型的条件函数和void(void)类型的操作函数很通用，但终究在功能上有所限制，如果能摆脱这一限制，Trigger类将有更大的使用自由度，实现这一点可借助template，将条件函数和操作函数的类型泛化成模板参数。

```c++
#include <functional>
template<typename Condition = std::function<bool(void)>,
         typename Operation = std::function<void(void)>>
class Trigger
{
private:
    //----------------------------------------------------------------------
    Condition m_cond;               ///< condition function
    Operation m_oper;               ///< operation function
    int m_count;                    ///< number of activation

    // nested struct definition
    // used as the return type of Trigger::operator()
    struct operation_t
    {
        Operation m_oper;
        
        // could be constructed from Operation object implicitly
        operation_t(Operation oper) : m_oper(oper)
        {}
        
        // template function to accept any number of arguments
        template<typename... Ts>
        typename Operation::result_type operator()(Ts... Vs)
{
            // the Operation object could be null if the Condition is not satisfied
            if (m_oper)
                return m_oper(Vs...);
            else        
                return typename Operation::result_type();
        }
    };

public:
    //----------------------------------------------------------------------
    Trigger(Condition cond, Operation oper, int n = -1) :
        m_cond(cond), m_oper(oper), m_count(n)
    {}

    template<typename... Ts>
    operation_t operator()(Ts... Vs)
    {
        if (m_count != 0 && m_cond(Vs...))
        {                 
            if (m_count > 0) --m_count;
            return m_oper;
        }
        else
        {
            return Operation();
        }
    }

    //----------------------------------------------------------------------
};  // !class Trigger
```

条件函数类型为模板参数Condition，操作函数类型为模板参数Operation，其默认类型均与前一版本Trigger中一致，因此原来代码中声明方式可作简单修改，即可保证二者效果一致。

```c++
Trigger tg(...);    // 旧版
Trigger<> tg(...);  // 新版
```

由于采用了模板，条件函数和操作函数的参数类型不局限于void，即可以接受参数，如果利用Trigger的operator()调用，将无法分清哪些参数传给条件函数，哪些参数传给操作函数。一个简单的办法是将Trigger的operator()函数所接收的全部参数将传递给条件函数，同时返回一个与操作函数有关的中间对象，用于继续调用操作函数的operator()，进而实现条件函数与操作函数的参数分离。具体示例如下，执行结果与前面一致。

```c++
#include <iostream>
int main()
{
    int i = 0, j = 5, k = 0;
    Trigger<> tg([&]() {return i > j; }, // 判断条件为 i > j
        [&]() {                          // 执行操作为赋值并输出
            k = i + j;
            std::cout << "i = " << i << "\t";
            std::cout << "j = " << j << "\t";
            std::cout << "k = " << k << "\n";
        },
        1);                              // 仅执行1次
        
    for (i = 0; i < 10; ++i)
        tg()();                          // 调用方式
        // 两个()分别放置条件函数和操作函数的参数
        // 注意参数类型必须与声明时给定的函数类型的参数类型一致
        // 此例中为bool(void)和void(void)，因此无参数
    
    return 0;
}
```

再看一个自定义模板参数的使用示例

```c++
#include <iostream>
int main()
{
    int j = 5;
    double a;
    Trigger<std::function<bool(int)>, 
            std::function<double(int)>> tg(
        [j](int ii) { return ii > j; },         // 用参数ii进行比较
        [j](int ii) { return double(ii) / j; }  // 执行操作为赋值并输出
        );                                      // 一直仅执
        
    for (int i = 0; i < 10; ++i)
    {
        a = tg(i)(i);                  // 调用方式
        std::cout << "i = " << i << "\t";
        std::cout << "j = " << j << "\t";
        std::cout << "a = " << a << "\n";
    }
       
    return 0;
}
```

输出结果为

![img](C++札记：一个Trigger的实现.assets/642)



***总结***

------

经过逐步的完善与改进，模板类Trigger可以让我们方便地定义触发操作，在日常工作中发挥重要作用。但本文给出的范本依然有自身的局限性，第一个版本相对第二个版本的功能更为简单，但执行效率更高，使用哪个取决于工作需求，大家也可以根据自己的实际情况进行定制。如果有更好的实现想法，欢迎交流！
