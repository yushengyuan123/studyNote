# make的作用

http://www.ruanyifeng.com/blog/2015/02/make.html

make可以让用户编写一个Makefile文件，根据makefile文件可以让程序，根据你所编写的一些shell指令去执行。有点类似于orange-ci

# 环境准备

windows下需要安装两个东西：

- cmake，到官网安装
- make，make和cmake是不同的，make伴随着mingw安装，**只安装了cmake没有安装make，cmake命令是无法执行的**![image-20211231103056068](C:\Users\admini\AppData\Roaming\Typora\typora-user-images\image-20211231103056068.png)

# 编译指令

 `cmake -G "MinGW Makefiles" ..`必须要输入这个不然默认采用nmake，但是你又没有安装nmake，那你就报错了

`..`是相等于你cmakelist文件的路径的

每一次的构建都会参考cache文件，如果你的构建有改变，录入目录发生改变等等，就要把那些cache文件全部删除掉

# makefile文件格式

```makefile
<target> : <prerequisites> 
[tab]  <commands>
```

# 添加版本号

我们利用makefile的模板能力编写**模板头文件.xxx.h.in**，在cmakelists文件中定义模板变量，生成版本号模板文件。

xxx.h.in文件

```cpp
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

```cmake
cmake_minimum_required (VERSION 3.22.0)
project (Tutorial)

// 定义头文件模板输出变量
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

// 根据模板输出头文件
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)

include_directories("${PROJECT_BINARY_DIR}")

add_executable(Tutorial tutorial.cpp)
```

执行命令cmake make就会生成TutorialConfig.h文件，最后在cpp引用头文件定义变量即可

# 添加自己的库

总体思路就是：在根目录下创建一个子目录，这个子目录包含了cmakelist文件，反正就是一个可以单独编译的cmake目录

创建目录MathFunctions，目录下文件列表

```
│   ├── CMakeLists.txt
│   ├── MathFunctions.h
│   └── mysqrt.cxx
```

cmakelist文件添加代码：`add_library(MathFunctions mysqrt.cpp)`

这个代码的作用就是可以把这个库**生成一个链接文件，将来用来链接进入exe中**

根目录下cmakelist添加语句：

```cmake
cmake_minimum_required (VERSION 3.22.0)
project (Tutorial)

set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)
// 添加编译器搜索头文件目录，如果不加入这句话的那么你主cpp文件就会找不到你自定义的头文件
include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
// 作用是添加一个子目录并构建该子目录
add_subdirectory (MathFunctions)
set (EXTRA_LIBS MathFunctions)

# add the executable
add_executable (Tutorial tutorial.cpp)
// 链接文件
target_link_libraries (Tutorial ${EXTRA_LIBS})
```

# 安装测试自检系统

https://blog.csdn.net/qq_38410730/article/details/102837401

这个东西的主要作用就是类似于前端自动化检测一样，可以帮助你检测你的库是否有问题。

## install指令

install指令用于指定在安装时运行的规则。它可以用来安装很多内容，包括二进制，动态库，静态库等等

```
install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
```

我们可以用到有用的变量`CMAKE_INSTALL_PREFIX`，用于指定cmake install时的相对地址

目标文件的安装

```cpp
install(TARGETS targets... [EXPORT <export-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
          PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```

参数targets可以是很多中目标文件，最常见就是通过**ADD_EXECUTABLE或者ADD_LIBRARY定义的目标文件，即可执行二进制、动态库、静态库**：

| 目标文件           | 内容              | 安装目录变量                      | 默认安装文件夹 |
| -------------- | --------------- | --------------------------- | ------- |
| archive        | 静态库             | ${CMAKE_INSTALL_LIBDIR}     | lib     |
| library        | 动态库             | ${CMAKE_INSTALL_LIBDIR}     | lib     |
| runtime        | 可执行二进制文件        | ${CMAKE_INSTALL_BINDIR}     | bin     |
| public_header  | 与库关联的public头文件  | ${CMAKE_INSTALL_INCLUDEDIR} | include |
| private_header | 与库关联的PRIVATE头文件 | ${CMAKE_INSTALL_INCLUDEDIR} | include |

为了符合一般默认安装路径，如果设置了DESTINATION参数，推荐配置在安装目录变量下的文件夹

举个例子：

```cmake
INSTALL(TARGETS myrun mylib mystaticlib
       RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
       LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
       ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
```

上面的例子会将：可执行二进制`myrun`安装到`${CMAKE_INSTALL_BINDIR}`目录，动态库`libmylib.so`安装到`${CMAKE_INSTALL_LIBDIR}`目录，静态库`libmystaticlib.a`安装到`${CMAKE_INSTALL_LIBDIR}`目录。

**这个位置是11对应的**

## 普通文件的安装

```
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
```

`FILES|PROGRAMS`若为相对路径给出的文件名

## 目录安装

```
install(DIRECTORY dirs...
        TYPE <type> | DESTINATION <dir>
        [FILE_PERMISSIONS permissions...]
        [DIRECTORY_PERMISSIONS permissions...]
        [USE_SOURCE_PERMISSIONS] [OPTIONAL] [MESSAGE_NEVER]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL]
        [FILES_MATCHING]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]] [...])
```

该命令将一个或者多个目录的内容安装到给定的目的地，目录结构被诸葛复制到目标位置

## 安装时脚本的运行

有时候需要在install过程中打印一些语句，或者执行一些cmake指令

```cmake
install([[SCRIPT <file>] [CODE <code>]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL] [...])
```

就可以用到这个

例如：

```cmake
install(CODE "MESSAGE(\"Sample install message.\")")
```

## 一个例子

```cmake
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

通过将以下两行添加到mathfunction的CMakeLists.txt中来安装头文件和静态库

这连两个句子意思是：

- 把MathFunctions静态链接安装在bin目录下
- 头文件安装在include目录

然后在根目录添加：

```cmake
# add the install targets
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
         DESTINATION include)
```

执行指令：

```cmake
cmake .
make
make install
```

你就会发现你的默认目录下安装了这些文件

**`DESTINATION`在这里是一个相对路径，取默认值。在unix系统中指向 `/usr/local` 在windows上`c:/Program Files/${PROJECT_NAME}`。**

添加测试用例：

```cmake
include(CTest)

# does the application run
add_test (TutorialRuns Tutorial 25)
# does it sqrt of 25
add_test (TutorialComp25 Tutorial 25)
set_tests_properties (TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
# does it handle negative numbers
add_test (TutorialNegative Tutorial -25)
set_tests_properties (TutorialNegative PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")
# does it handle small numbers
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")
# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")
```

# cpack生成安装包

假如我们想把我们的项目分给其他人，以便他们可以安装并使用它

我们只需要在cmakelist加几行代码

# 语法

cmake能识别cmakelists.txt和*.cmake格式的文件。cmake能够以三种方式来组织文件：

| 类别           | 文件格式           |
| ------------ | -------------- |
| Directory    | CMakeLists.txt |
| `Script`（脚本） | <script>.cmake |
| `Module`（模块） | <module>.cmake |

## Directory

当cmake处理一个项目时，入口点是一个名为cmakelist.txt的源文件。假如我们的子文件夹需要编译时，就要使用`add_subdirectory(<dir_name>)`命令来为构建添加子目录，每一个子目录需要有CMakeLists.txt作为入口点

## script

一个单独的`<script>.cmake`源文件可以使用cmake命令行工具`cmake -P <script>.cmake`选项来执行脚本。

## module

# 外部构建

## 分割编译文件和源文件

如果我们单纯的直接`cmake .`那么所有编译的产物和我们的源文件就会混在一起，十分不好看，所以我们新建一个build目录，我们把我们的编译产物全部放在我们的build目录下

```cmake
mkdir build # 创建build目录
cd build # 进入build目录
// 输入这个指令前先删除所有的cache文件
cmake -G "MinGW Makefiles" ..
cmake .. # 因为程序入口构建文件在项目根目录下，采用相对路径上级目录来使用根目录下的构建文件
make
```

# 基础语法

```makefile
result.txt: source.txt
   rm result.txt
   cp source.txt result.txt
```

：前面的是是make xxx匹配的文件，后面的是它的依赖文件，如果没有依赖文件，指令就无法执行成功，你可以不写。

后面的command写的就是linux指令语句了，按照顺序执行。

# all含义

```makefile
all:
```

有时你想连续去执行多个语句，但是我们默认make只执行第一个条，无法执行多条，那么这个时候make就可以帮助我们实现这个目标

```makefile
all: result.txt test

result.txt: source.txt
   rm result.txt
   cp source.txt result.txt

.PHONY: clean
clean:
   rm result.txt

test:
   @echo $@
```

这样写输入make，就可以连续执行result和test

# 特殊符号含义

## @

这个符号的作用可以加在一些语句，我们不希望他在控制台打印出来，例如echo

```makefile
test:
   echo $@
```

![image-20210831201951341](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210831201951341.png)

增加了@之后：

![image-20210831202010622](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210831202010622.png)

# 自动变量

## $@

指代当前构建的那个目标

```bash
a.txt b.txt: 
    touch $@
```

等价于

```bash
a.txt:
    touch a.txt
b.txt:
    touch b.txt
```

## $<

$< 指代第一个前置条件

```bash
a.txt: b.txt c.txt
    cp $< $@ 
```

等价于

```bash
a.txt: b.txt c.txt
    cp b.txt a.txt 
```

## **$^**

$^ 指代所有前置条件，之间以空格分隔。比如，规则为 t: p1 p2，那么 $^ 就指代 p1 p2 。

# 判断和循环

判断

```bash
ifeq ($(CC),gcc)
  libs=$(libs_for_gcc)
else
  libs=$(normal_libs)
endif
```

循环

```bash
LIST = one two three
all:
    for i in $(LIST); do \
        echo $$i; \
    done

# 等同于

all:
    for i in one two three; do \
        echo $i; \
    done
```

# 函数

make里面有很多的内置函数

# cmakeLists语法

## **include_directories**

他将指定目录添加到编译器的头文件搜索路径之下，指定的目录被解析成为当前源码的路径的相对路径

例子：

├── CMakeLists.txt    #最外层的CMakeList.txt
 ├── main.cpp    #源文件，包含被测试的头文件
 ├── sub    #子目录
 └── test.h    #测试头文件，是个空文件，被外层的main,cpp包含

```makefile
# CMakeList.txt
cmake_minimum_required(VERSION 3.18.2)
project(include_directories_test)
add_executable(test main.cpp)
```

```cpp
//main.cpp
#include "test.h"
#include <stdio.h>
int main(int argc, char **argv)
{
    printf("hello, world!\n");
    return 0;
}
```

执行cmake 会报错找不到这个test.h

这时候我们就需要`include_directories`帮助我们找到这个搜索的目录

```cmake
# CMakeList.txt
cmake_minimum_required(VERSION 3.18.2)
project(include_directories_test)
include_directories(sub) #与上个场景不同的地方在于此处
add_executable(test main.cpp)
```

这样就能够找到了

## configure_file

作用就是将一个文件input参数指定的内容拷贝到output参数指定的内容。

- 它会将input文件中的@var@或者${var}替换成cmake指定的值
- 将input文件中的`#cmakedefine var`关键字替换成`#define var`或者`#undef var`，取决于cmake是否定义了var

需要准备两个文件config.h.in（input），cmakelist.txt（output）

概括：**就是它可以将根据你的cmakelist模板，生成出一个头文件.h。根据你在cmakelists中定义的变量模板去替换掉.h.in头文件的东西**

```cpp
// config.h.in
#cmakedefine var1
#cmakedefine var2 "@var2@" #注意：@@之间的名称要与cmakedefine后的变量名一样
#cmakedefine var3 "${var3}" #注意：${}之间的名称要与cmakedefine后的变量名一样
#ifdef _@var4@_${var5}_
#endif
```

```bash
cmake_mininum_required(VERSION 2.8)
project (configure_file_test)
option (var1 "use var1..." ON)  #定义var1，也可以使用cmake -Dvar1=ON替代
set (var2 13) #指定var2的值
set (var3 "var3string") #指定var3的值
set (var4 "VARTEST4")
set (var5 "VARTEST5")

configure_file (config.h.in config.h)
```

生成的config.h

```cpp
#define var1
#define var2 "13"
#define var3 "var3string"
#ifdef _VARTEST4_VARTEST5_
#endif
```

## add_library

该指令主要作用就是将指定的源文件**生成链接文件**（例如.a，.so后缀的，我们把他当作库时就需要这些链接文件），添加到工程里面去

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2] [...])
```

name就是生成库的名字，STATIC | SHARED | MODULE指的是生成动态库还是静态库

## link_directories

该指令的作用主要是指定要连接的库文件的路径，该指令有时候不一定需要。因为find_package和find_library指令可以得到库文件的绝对路径

```
link_directories(
    lib
)
```

## target_link_libraries

该指令的作用为将目标文件与库文件进行链接，与`link_directories`区别就是这个东西是**找路径的**，**这个指令才是真正的链接**

## add_subdirectory

作用是添加一个子目录并构建该子目录

```
add_subdirectory (source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```

- source_dir 该参数就是指令这个子目录，**子目录下应该包含`CMakeLists.txt`文件和代码文件**
- binary_dir 输出目录，默认的输出目录使用`source_dir`

## include指令

```cmake
include(<file|module> [OPTIONAL] [RESULT_VARIABLE <VAR>] [NO_POLICY_SCOPE])
```

这个指令作用是载入并运行来自文件或者模块的cmake代码

简单的来说，感觉就是加载别人的库，让后让你cmake编译获得别的能力

## 设置安装规则install

install用于在安装时指定运行的规则，它可以用来安装很多内容，包括二进制，动态库和静态库

它可以把你的头文件和可执行文件等进行打包，输出到一个目录中，让后你就可以直接用它

## list指令

`list(subcommand <list>[args...])`

subcommand为具体的列表操作子命令，例如读取，查找，修改。[args...]为对列表变量操作需要的参数

list命令即对列表的一系列操作，cmake中的列表

### 命令解析


