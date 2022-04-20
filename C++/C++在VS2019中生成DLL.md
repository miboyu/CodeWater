# C++在VS2019中生成DLL

Created: 2022-04-19 10:37

Updated: 2022-04-20 00:40

- 添加 DLL 使用方法，通过配置项目属性实现。

#CPP 
#DLL 
#VS2019

---

[toc]

## 1 前言

最近需要在 Windows 平台下利用 C++ 代码生成[动态链接库](https://baike.baidu.com/item/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93%E6%96%87%E4%BB%B6/2667700) （Dynamic Link Library, DLL），相比 Linux 平台复杂许多，遂记录之。

生成工具为 Visual Studio 2019，其它版本的 VS 大同小异。

**相关文章推荐：**

- [深入详解C++创建动态链接库DLL以及如何使用它](https://blog.csdn.net/qq_27825451/article/details/103924017)
- [Windows静态链接库与动态链接库的创建和显式与隐式调用](https://blog.csdn.net/m0_37621078/article/details/88385101)

## 2 生成DLL

### 2.1 新建项目

(1) 打开 VS2019，选择 `新建项目(Create a new project)` ，或通过 `文件(File)` → `新建(New)` → `项目(Project)` 进行。

![attachments/Pasted image 20220418205200.png](attachments/Pasted%20image%2020220418205200.png)

![attachments/Pasted image 20220419114347.png](attachments/Pasted%20image%2020220419114347.png)

(2) 新建项目中选择 `空项目(Empty Project)` 。

> 如果选择动态链接库项目，VS 会自动生成一些文件和配置，这些文件会影响原有代码结构，因此建议使用空项目，然后手动配置。

![attachments/Pasted image 20220419114816.png](attachments/Pasted%20image%2020220419114816.png)

(3) 填写项目名称和路径。

![attachments/Pasted image 20220419120109.png](attachments/Pasted%20image%2020220419120109.png)

(4) 创建好空项目后，将自己的代码添加进来，项目创建完成。

![attachments/Pasted image 20220419120405.png](attachments/Pasted%20image%2020220419120405.png)

### 2.2 修改代码

> 在 Linux 平台下，生成 DLL 时默认导出所有符号，可直接使用原始代码。但在 Windows 平台下，默认不导出符号，需要添加导出标识，生成和使用都比较麻烦。。。

原始代码如下：

```cpp
// MyDLL.h
#ifndef MYDLL_H_
#define MYDLL_H_
#include <string>
namespace MyDLL
{
    double sign(double x);
    class Person
    {
    private:
        std::string m_name;
        int m_age = 0;
    public:
        Person() = default;
        Person(const std::string& name, int age);
        void print() const;
    };
}
#endif
```

```cpp
// MyDLL.cpp
#include "MyDll.h"
#include <cmath>
#include <iostream>
namespace MyDLL
{
    double sign(double x)
    {
        return (std::abs(x) < 1e-8) ? 0 : (x > 0) ? 1 : -1;
    }
    Person::Person(const std::string& name, int age) : 
        m_name(name), m_age(age)
    {}
    void Person::print() const
    {
        std::cout << m_name << " is of age " << m_age << std::endl;
    }
}
```

VS 编译器（MSVC）提供关键字 `__declspec` 来指定符号导入导出，生成 DLL 时在符号前添加导出标识 `__declspec(dllexport)`，使用 DLL 时在符号前添加导入标识 `__declspec(dllimport)`。因此头文件 `MyDLL.h` 需要进行修改，实现文件 `MyDLL.cpp` 则不用修改，修改后的头文件如下：

```cpp
// MyDLL.h 添加导出标识
#ifndef MYDLL_H_
#define MYDLL_H_
#include <string>
namespace MyDLL
{
    __declspec(dllexport) double sign(double x);      // 导出函数
    class __declspec(dllexport) Person                // 导出class
    {
    private:
        std::string m_name;
        int m_age = 0;
    public:
        Person() = default;
        Person(const std::string& name, int age);
        void print() const;
    };
}
#endif
```

### 2.3 设置属性

> 空项目默认生成可执行程序，此处要生成 DLL，需要对项目属性进行配置。

(1) 打开项目属性。

![attachments/Pasted image 20220419123639.png](attachments/Pasted%20image%2020220419123639.png)

(2) 在`通用`属性里注意以下几项：

> 注意顶部的 Configuration 和 Platform，通常 Configuration 有 Debug（含调试信息，生成文件较大）和 Release（无调试信息，用于发布），Platform 有 Win32（32位程序）和 x64（64位程序），组合起来就有4种，设置项目属性时要注意是对哪种组合生效。此处选择的是对4种组合全部生效。

- `Output Directory` 输出目录，生成目标文件 DLL 存放的位置。
- `Intermediate Directory` 中间目录，中间产生的临时文件存放的位置。
- `Target Name` 目标名称，生成目标文件 DLL 的文件名。
- `Configuration Type` <span style="color:red">配置类型，重点修改项，选择生成动态链接库。</span>

以上几个属性都有默认值，这里改成了自定义值。前两个目录都添加了 Platform 和 Configuration 的区分，不同组合下生成的文件会存放在不同目录下，便于打包。目标名称默认为项目名称，可以改为自定义名称。配置类型有可执行程序、动态链接库、静态链接库等，这里我们要生成动态链接库，因此这里要进行修改。后面还有几个平台工具、语言标准的属性，视需求修改。

![attachments/Pasted image 20220419124735.png](attachments/Pasted%20image%2020220419124735.png)

(3) `高级`属性中，如果有 UTF-8 需求，可修改对应的字符集，其他属性可采用默认值。Linux 平台默认都是 UTF-8 编码，代码里的中文支持较好，Windows 平台比较麻烦，非 ASCII 代码经常遇到乱码问题，建议统一改用 UTF-8。

![attachments/Pasted image 20220419131014.png](attachments/Pasted%20image%2020220419131014.png)

(4) `调试`属性中，需要命令行参数的，可在此处添加，一般用于代码测试阶段。这里我们认为已经测试好了，直接生成 DLL，因此不用给命令行参数。

![attachments/Pasted image 20220419131236.png](attachments/Pasted%20image%2020220419131236.png)

(5) `VC++ 目录` 中重点关注 Include 目录和 Library 目录。如果项目代码有自己的目录结构（如 include/, src/ 等），注意将相应的 include 目录添加到 `Include Directory`，本示例较为简单，因此没有添加 include 目录。对于第三方库的 include 目录，可添加到 `External Include Directories`。`Library Directory` 主要用于指定要使用的动态库或静态库所在的目录，可能因 Configuration 和 Platform 的不同而不同，注意区分，本示例用于生成DLL，并未使用到外部库。

![attachments/Pasted image 20220419135950.png](attachments/Pasted%20image%2020220419135950.png)

(6) `C/C++ 通用` 中同样可以指定附加的 include 目录，测试过程中可以调高 `Warning Level` 以增大发现潜在 bug 的机率，测试通过以后生成 DLL 的过程中可以不用管。

![attachments/Pasted image 20220419141051.png](attachments/Pasted%20image%2020220419141051.png)

(7) `C/C++ 优化` 主要用于 Release 版，优化级别越高，程序的执行速度可能越快。在 Debug 模式下最好不要优化，否则单步执行时可能出现莫名其妙的跳转，原因是部分小函数被优化掉了。

![attachments/Pasted image 20220419141402.png](attachments/Pasted%20image%2020220419141402.png)

(8) `C/C++ 预处理器` 中关注预定义的宏，与代码中的 `#define XXX` 功能一样，只不过是在编译器层面进行。后面为了使头文件同时满足生成和使用 DLL 以及跨平台要求，会在此处添加定义。

![attachments/Pasted image 20220419141931.png](attachments/Pasted%20image%2020220419141931.png)

(9) `C/C++ 代码生成` 中关注 `Runtime Library` 的属性值，这里有四个选项，`/MT` 代表 Release 版的静态链接库，`/MTd` 代表 Debug 版的静态链接库，`/MD` 代表 Release 版的动态链接库，`/MDd` 代表 Debug 版的动态链接库。在<span style="color:red">生成和使用</span>动态/静态链接库时，需要注意这里的设置是否匹配，按照 Configuration 为 Release/Debug，链接库为动态/静态的标准选择，<span style="color:red">这是经常容易出错的一个地方</span>。

![attachments/Pasted image 20220419142512.png](attachments/Pasted%20image%2020220419142512.png)

(10) `链接器 输入` 中注意这里的附加依赖项，这时使用其他动态/静态库时需要设置的地方，(5) 中设置了库目录，这里设置使用的库名，一般为 `xxx.lib`，动态链接库除了 `xxx.dll` 外还会附带一个 `xxx.lib`，静态链接库则直接为 `xxx.lib`（此处解释较为片面，具体见前面的推荐文章）。

![attachments/Pasted image 20220419143708.png](attachments/Pasted%20image%2020220419143708.png)

### 2.4 编译生成

设置好项目属性后，即可进行生成，一种方法是通过菜单栏 `生成(Build)` → `生成解决方案(Build Solution)`，另一种方法是方案管理中右键对应的项目，然后点击 `生成(Build)`。如果一个解决方案中包含多个项目，可采用第二种方法。

![attachments/Pasted image 20220419151346.png](attachments/Pasted%20image%2020220419151346.png)

![attachments/Pasted image 20220419151439.png](attachments/Pasted%20image%2020220419151439.png)

生成的文件会出现在设置好的目录中，如下图所示的 `Win32/Debug/`，其中 `MyDLL.dll` 和 `MyDLL.lib` 为使用 DLL 时所需要的。

![attachments/Pasted image 20220419152002.png](attachments/Pasted%20image%2020220419152002.png)

## 3 使用DLL

### 3.1 直接复制法
新建一个项目，将头文件 `MyDLL.h` 和库文件 `MyDLL.dll`、`MyDLL.lib` 复制到新项目下，在 2.3 (10) 中给出的位置添加依赖项 `MyDLL.lib`，同时修改头文件内容：

```cpp
// MyDLL.h 添加导入标识
#ifndef MYDLL_H_
#define MYDLL_H_
#include <string>
namespace MyDLL
{
    __declspec(dllimport) double sign(double x);      // 导入函数
    class __declspec(dllimport) Person                // 导入class
    {
    public:
        Person() = default;
        Person(const std::string& name, int age);
        void print() const;
    };
}
#endif
```

添加 main 函数：

```cpp
#include "MyDLL.h"
#include <iostream>
int main()
{
    std::cout << "sign(-1e-7) = " << MyDLL::sign(-1e-7) << std::endl;
    std::cout << "sign(1e-9) = " << MyDLL::sign(1e-9) << std::endl;
    std::cout << "sign(1e-7) = " << MyDLL::sign(1e-7) << std::endl;
    MyDLL::Person p1;
    MyDLL::Person p2("P2", 20);
    std::cout << "p1 print: ";
    p1.print();
    std::cout << "p2 print: ";
    p2.print();
    return 0;
}
```

测试结果：

```txt
sign(-1e-7) = -1
sign(1e-9) = 0
sign(1e-7) = 1
p1 print:  is of age 0
p2 print: P2 is of age 20
```

### 3.2 配置属性法

有时项目比较复杂，每次都等生成完再复制一次显得不够优雅，直接在项目属性里配置好就可以一劳永逸了。

对于 DLL 的使用需要注意<span style="color:red">两个目录</span>，一个是 `xxx.lib` 所在的目录，一个是 `xxx.dll` 所在的目录。通常生成时这两个都在一个目录下，但使用时要配置两个地方。

(1) lib 目录：在 `VC++ 目录` 下面的 `Library Directories`，直接添加对应的路径，如果生成时按照一定的规律指定输出路径，此处也可以使用类似的路径写法。如果不给定 lib 目录而又在依赖项里添加了 `xxx.lib`，则会在编译期报错。

![attachments/Pasted image 20220420005606.png](attachments/Pasted%20image%2020220420005606.png)

(2) dll 目录：在 `Debugging` 下面的 `Environment`，将对应的路径添加到 `PATH` 环境变量，DLL 文件相当于二进制的可执行文件，VS2019 在搜索时<span style="color:red">不是在 lib 目录下</span>查找的，因此这个要单独设置，否则会在运行过程中提示找不到对应的 DLL 文件。

![attachments/Pasted image 20220420010217.png](attachments/Pasted%20image%2020220420010217.png)

配置完成以后则可以正常使用 DLL 运行自己的程序。

## 4 头文件改造

前面可以看出，生成和使用 DLL 时的头文件是不一样的，一个使用了 `__declspec(dllexport)`，一个使用了 `__declspec(dllimport)`。为了使头文件更具通用性，改造成如下形式：

```cpp
// MyDLL.h 通用形式
#ifndef MYDLL_H_
#define MYDLL_H_
#include <string>

#ifdef _MSC_VER
  #ifdef NO_DLL
    #define DLL_API
  #elif defined DLL_EXPORT
    #define DLL_API __declspec(dllexport)
  #else
    #define DLL_API __declspec(dllimport)
  #endif
#else
  #define DLL_API
#endif

namespace MyDLL
{
    DLL_API double sign(double x);     // 改用 DLL_API 形式
    class DLL_API Person               // 改用 DLL_API 形式
    {
    private:
        std::string m_name;
        int m_age = 0;
    public:
        Person() = default;
        Person(const std::string& name, int age);
        void print() const;
    };
}
#endif
```

其中添加了预编译部分，以 `DLL_API` 这个宏代替原来的 `__declspec`。在 2.3 (8) 中提到的预定义宏的位置，定义 `NO_DLL`，则是以<span style="color:red">生成可执行程序</span>的方式使用头文件；定义 `DLL_EXPORT`，则是以<span style="color:red">生成 DLL</span> 的方式使用头文件；不定义，则是以<span style="color:red">使用 DLL</span> 的方式使用头文件；Linux 环境下，无需 `__declspec`。

当然，这只是给出的一个示例，用于说明一种解决问题的思路。

## 5 Linux环境中使用CMake

前面使用 VS2019 几经曲折终于得到了可用的 DLL，下面展示一下在 Linux 环境中使用 CMake 生成 DLL 的方法，目录结构如下：

```bash
.
├── build/             构建目录
├── MyDLL.h            头文件
├── MyDLL.cpp          源文件
├── main.cpp           主函数（测试）
└── CMakeLists.txt     CMake工程文件
```

其中 `MyDLL.h`、`MyDLL.cpp`、`main.cpp` 如前面所示，`CMakeLists.txt` 内容如下：

```cmake
# cmake 版本要求
cmake_minimum_required(VERSION 3.16)
# 项目基本属性：名称，版本，描述，语言
project(DLLTEST
    VERSION 1.0.0
    DESCRIPTION "A DLL Test"
    LANGUAGES CXX)

# 设置变量
set(SOURCE MyDLL.cpp)

# 添加静态库并设置输出名称
add_library(dlltest_static STATIC ${SOURCE})
set_target_properties(dlltest_static PROPERTIES OUTPUT_NAME "dlltest")
# 添加动态库并设置输出名称
add_library(dlltest_shared SHARED ${SOURCE})
set_target_properties(dlltest_shared PROPERTIES OUTPUT_NAME "dlltest")

# 添加可执行程序（测试），并链接到动态库
add_executable(dlltest main.cpp)
target_link_libraries(dlltest dlltest_shared)

```

编译及执行指令如下：

```bash
> cd build
> cmake ..
-- The CXX compiler identification is GNU 9.4.0
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /mnt/e/Program/C++/DLL_Test_Use/build
> make 
Scanning dependencies of target dlltest_shared
[ 16%] Building CXX object CMakeFiles/dlltest_shared.dir/MyDLL.cpp.o
[ 33%] Linking CXX shared library libdlltest.so
[ 33%] Built target dlltest_shared
Scanning dependencies of target dlltest
[ 50%] Building CXX object CMakeFiles/dlltest.dir/main.cpp.o
[ 66%] Linking CXX executable dlltest
[ 66%] Built target dlltest
Scanning dependencies of target dlltest_static
[ 83%] Building CXX object CMakeFiles/dlltest_static.dir/MyDLL.cpp.o
[100%] Linking CXX static library libdlltest.a
[100%] Built target dlltest_static
```

整个工程配置及生成过程非常简洁，同时生成静态库、动态库和可执行程序，生成后的 `build` 目录如下：

```bash
build/
├── CMakeFiles/
├── cmake_install.cmake
├── CMakeCache.txt
├── compile_commands.json
├── dlltest*                 可执行程序
├── libdlltest.a*            静态库
├── libdlltest.so*           动态库
└── Makefile
```

调用 `dlltest` 执行结果如下：

```bash
> ./dlltest
sign(-1e-7) = -1
sign(1e-9) = 0
sign(1e-7) = 1
p1 print:  is of age 0
p2 print: P2 is of age 20
```

## 6 结语

写了这么多，只有对 Windows 无尽的吐槽，唉……