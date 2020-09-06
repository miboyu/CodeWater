# C++札记：二分搜索算法

***摘要***

------

本文主要讲解C++算法库`<algorithm>`中的二分搜索算法：`lower_bound`、`upper_bound`、`equal_range`和`binary_search`，尤其是各算法对自定义比较函数(comparer)要求的不同，这是容易导致误用的地方。



***简介***

------

二分搜索算法，顾名思义是采用二分法进行搜索的算法，特点是对数时间找到目标（前提是支持随机访问，如vector），要求是区间已排序。`<algorithm>`中提供了`sort`等排序算法，可用于二分搜索前的准备工作，本文不展开讲排序算法，假定搜索区间已按`operator<()`排序，其它排序类型可依此类推。各算法的详细说明见[手册页](https://zh.cppreference.com/w/cpp/algorithm)。



`lower_bound`算法搜索区间范围内第一个满足`operator>=()`的目标；

`upper_bound`算法搜索区间范围内第一个满足`operator>()`的目标；

`equal_range`算法将`lower_bound`和`upper_bound`的结果放在一个`pair`中返回；

`binary_search`算法搜索区间范围内是否存在指定目标。



各算法的调用形式如下：

```c++
1. algorithm_name(begin_iterator, end_iterator, target);
2. algorithm_name(begin_iterator, end_iterator, target, compare_fun);
```

其中*begin_iterator*和*end_iterator*定义搜索的起止区间，*target*为目标值，*compare_fun*为自定义的比较函数。



***调用方式1***

***

------

`lower_bound`算法和`upper_bound`算法非常相似，比较函数只差一个等于号(=)，可能你会觉得有点多余、重复，下面通过一个例子展示它们之间的区别：

```c++
std::vector<int> v { 1, 3, 5, 7, 9 };
int target = 3;
// lower 将指向数据3
auto lower = std::lower_bound(v.begin(), v.end(), target);
// upper 将指向数据5
auto upper = std::upper_bound(v.begin(), v.end(), target);

target = 4;
// 此时 lower 将指向数据5
lower = std::lower_bound(v.begin(), v.end(), target);
// 此时 upper 也指向数据5
upper = std::upper_bound(v.begin(), v.end(), target);
```

可见当区间内不包含目标值时，`lower_bound`和`upper_bound`返回的结果一致。下面的例子更能体现这两个算法的作用：

```c++
std::vector<int> v { 1, 3, 3, 3, 5, 7, 9 };
int target = 3;
// lower 将指向第一个3
auto lower = std::lower_bound(v.begin(), v.end(), target);
// upper 将指向数据5
auto upper = std::upper_bound(v.begin(), v.end(), target);
// 下面将打印 3, 3, 3, 
while(lower != upper)
{
    std::cout << *lower << ", ";
    ++lower;
}
```

这也正是`lower_bound`算法和`upper_bound`算法名称的含义，[lower, upper)半开区间正好表示所有target序列。同理，`equal_range`将返回lower和upper组成的`pair`，表示这个区间(range)内的值与target相等(equal)。



`binary_search`算法的功能比较简单，只用于检查给定区间是否包含target，包含则返回true，不包含则返回false。



***调用方式2***

------

对于简单数据类型（如内置类型），调用方式1可以满足大部分需求。对于自定义的class，如果定义了比较运算符，也可以按方式1使用；如果没有，则需要指定比较函数(compare_fun)，即调用方式2。

```c++
// 定义一个结构体
struct person_t
{
    std::size_t ID;
    std::string name;
    ...
};
std::vector<person_t> persons;
...  // 按ID从小到大顺序填充persons
std::size_t target_ID = ... ;  // 目标ID
// 注意lower_bound和upper_bound中自定义比较函数的参数位置
// lower_bound的比较函数中，第一个参数是person_t，第二个参数是size_t
auto lower = std::lower_bound(
    persons.begin(), persons.end(), target_ID,
    [](const person_t& p, std::size_t id) { return p.ID < id; });
// upper_bound的比较函数中，第一个参数是size_t，第二个参数是person_t
auto upper = std::upper_bound(
    persons.begin(), persons.end(), target_ID,
    [](std::size_t id, const person_t& p) { return id < p.ID; });   
```

`lower_bound`算法和`upper_bound`算法的比较函数要求第一个参数 < 第二个参数，即满足排序规则。`lower_bound`算法中，对于某个区间值(p.ID)，如果比较函数返回true，说明该值 < target（要求找到第一个 >= target的值），那么继续向右搜索，否则向左搜索。`upper_bound`算法中，对于某个区间值(p.ID)，如果比较函数返回true，说明该值 > target（要求找到第一个 > target的值），那么继续向左搜索，否则向右搜索。这里理解起来有点绕，只需记住target参数的位置即可（`lower_bound`中target参数在右，`upper_bound`中target参数在左），然后在左边参数小于右边参数时返回true。



由于`lower_bound`算法和`upper_bound`算法对比较函数的要求不同，导致`equal_range`算法的比较函数更为特殊，本文不过度介绍这一复杂机制，真正有需求时可以参考[这里](https://zh.cppreference.com/w/cpp/algorithm/equal_range)中给出的示例。



`binary_search`算法的比较函数形式与`lower_bound`一致。



***总结***

------

日常使用二分搜索算法时，最多使用的是调用*方式1*，实在有特殊情况再考虑调用*方式2*。由于各算法对自定义比较函数有格式要求，使用过程中需格外注意。祝大家学习进步！
