# 指针

指针是“指向”另外一种类型的复合类型。指针也实现了对其他对象的间接访问

指针实现了**对其他对象**的间接访问，是实现对对象的访问、

**指针存放某个对象的地址**，不是存其他东西

## 指针和引用类型区别

## 引用

**引用类型在定义的时候就需要初始化，否则就会报错**

```cpp
    int i = 1;

    int &r;
	// 这种写法是错误的
    r = i
```

## 获取对象地址

指针存放某个对象的地址，获取地址，就需要用到&取址符

## 指针值

- 指向一个对象
- 指向临近对象所占空间的下一个位置
- 空指针
- 无效指针，就是上面以外其他的值

## 区别&的多重身份

在类型隔壁，他就是指引用的意思

```
    int ival = 1;
    int &r = ival;
    printf("%d", r);
    r = 3
    printf("%d", ival);
```

在赋值符号的后面，他就是指取地址的意思

## 空指针

nullptr和null区别：

初始化空指针三种方法：

![image-20211115152447116](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211115152447116.png)

## 指针比较

- 任何非0指针对应的条件值都是true
-  ==  != 比较。如果两个指针地址值相同，他们相等，否则不等（为null，指向同一个对象，指向同一个对象的下一个地址）
- 一个指针指向某个对象，同时另一个指针指向另一个对象的下一个地址，此时两个指针值可能出现相同的情况

## void* 指针

它可以存放任意对象的地址。

- 不能直接操作void *所指向的对象，因为我们并不知道这个对象到底是什么



# const

## const引用

const绑定到对象上，修饰后我们不能修改它所绑定的对象

## 指针和const

指向常量的指针，不能改变其所指对象的值，要想存放常量对象地址，只能使用指向常量的指针

```cpp
    const double pi = 3.14;
    double *d = pi; // 错误
    const double *a = pi; // 正确
    *a = 321; // 错误
```

```cpp
    int i = 1;
    int j = 2;

    const int *p = &i;
    const int *p1 = &j;

    printf("%d\n", *p);

    p = p1; // 正确，允许修改

    printf("%d", *p);
```



## const指针

```cpp
    int errNum = 0;
    int *const curEle = &errNum;

    int *p = 0;

	*p = 3;// 错误，不允许改变里面的值

    curEle = p; // 错误不能修改指针指向

    printf("%d\n", *curEle);

    *curEle = 1;

    printf("%d", *curEle); // 正确可以改变里面的内容
```

# 类型别名

```cpp
typedef double wages; // wages是double的别名

typedef wages base, *p; // base也是double的别名，p是double *的别名
```

## using别名

```cpp
using SI = int;

int main() {
    SI i = 2;
    pint a = &i;

    printf("%d", *a);
}
```



## 指针，常量和类型别名

```
typedef int *pint; // pint是int * 的别名
```

# auto

编程时，我们常需要把表达式赋值给变量，这就要求我们声明的时候知道这个变量是什么类型。有的时候，这个我们就可以用的auto，auto可以帮助我们推算出这表达式是什么类型

```
auto item = val1 + val2 // 如果val1和val2都是int型，那么这个item就是int
```

# 自定义数据结构

```
struct Person
{
    int i;
    double j;
};
```

# 编写自己的读文件

## 头文件保护符

```
#ifndef Sales_data_h
#define Sales_data_h
#include <string>
struct Sales_data
{
    std::string bookNo;
    double revene;
};
#endif
```

- ifndef 如果没有定义
- define定义

使用：

```cpp
#include <stdio.h>
#include "Sales_data.h"

int main() {
    Sales_data person;

    person.revene = 1.0;

    printf("%f", person.revene);
}
```



# using

他的作用就是帮助我们不需要每一次调用库的某个方法的时候都需要`xxx::xxx`这样十分繁琐，可以用using声明一次就可以了。

```cpp
#include <stdio.h>
#include <iostream>

using std::cin;

int main() {
    cin
}
```

# string标准库

这个库可以帮助我们不需要像c语言那样定义一个字符串这么麻烦

## 初始化一个字符串

```cpp
int main() {
    string a = "123";
    string b("value"); // 等同于 b = "value"
    string c(4, '1'); // 等同于 c = "1111"

    cout << "str:" << a << endl;
    cout << "str:" << b << endl;
    cout << "str:" << c << endl;
}
```

string类型通过下表访问也是可以的

```cpp
    string a = "123";

    cout << a[1] << endl;
```

## 字符串方法

1. `os << s`将s写到输出流os，返回os

2. `is >> s`从is中读取字符串赋给s，返回is

3. `s.empty`判断字符串是否为空

4. `s.size`，返回s中字符个数

5. getline读取一行

   ```
       string s;
       // 一次读取一整行，直到文件读取结束为止
       while (getline(cin, s))
       {
           cout << s << endl;
       }
   ```

   

# vector

vector表示对象的集合，其中所有的对象相同。

## 定义和初始化vector对象

```cpp
    vector<T> v2(v1); // v2中包含v1所有副本
    vector<T> v2 = v1; // 等价上面那个
    vector<T> v2(n, val) //包含n个重复元素，每个元素都是val
    vector<string> vec = {"1", "1"};
```

## vector的方法

向vector中添加元素

```cpp
    vector<int> vec;

    for (int i = 0; i < 10; i++)
    {
        vec.push_back(i);
    }
```

## vector打印方式

```cpp
    vector<int> vec = {1, 2};

    for (auto i = 0; i != vec.size(); i++)
    {
        cout << vec[i] << endl;
    }

 for (auto c : vec) cout << c << " ";
```

# 迭代器

迭代器的作用可以用来对一些容器类似进行遍历操作，例如vector类型，字符串不是容器，但是字符串也支持这个迭代器操作

## 迭代器操作

- *iter 返回迭代器iter所指元素的引用
- iter->mem 获取mem成员，和结构提类似
- ++iter 指向下一个元素
- --iter 指示上一个元素
- iter1 == iter2 看两个迭代器是否相同
- != 和上面相反

例子：

```cpp
    string a = "a23";

    if (a.begin() != a.end()) {
        auto it = a.begin();
        *it = toupper(*it);
    }

    cout << a << endl;
```

# 数组

数组和vector不同之处在于，数组的大小是确定的，vector大小是不确定的。

当我们不知道大小的时候，我们就是用vector

由于数组大小是确定的，他的性能在某些场景下更加优一些

## 数组误区

不允许拷贝和赋值

```
int a[] = {0,1,2};
int a2[] = a; // 错误
```

## 复杂数组

```
int (*p)[10] = &a; // 指向有10个元素的数组d
```

## 数组和指针

大多数表达式中，数组类型的对象其实是使用一个指向该数组首元素的指针

```cpp
    string b[10] = {};

    string *p = b;// 等价于p = &b[0]
```

## 数组遍历

```
    int a[5] = {1,2,3,4,5};

    for (int value : a) {
        cout << value << endl;
    }
    
    // 这样也是可以的，建议这么写，有两个原因：第一假如你需要改变数组元素值，你就需要取引用。第一个原因，如果说你的数组是一个二维数组
    for (int &value : a) {
        cout << value << endl;
    }
```



# 运算符

## sizeof

sizeof返回一个表达式或者一个类型名字的字节数

```cpp
    string a[5] = {"1","2","3","4","5"};

    cout << sizeof(a) << endl;

    cout << sizeof(int) << endl;
```

## 类型转换

### 显示类型转换

#### 命令强制类型转换

```
cast-name<type>(expression)
// tpye是转换的目标类型expression是要转换的值
// cast-name是static_cast、dynamic_cast、const_cast和reinterpret_cast中的一种
```

- 任何具有明确定义类型转换，只要不包含底层const，都可以使用static_cast

- const _cast只能改变运算对象的底层const，这个作用就是把一个常量对象转变为非常量对象

  ```
      int b = 321;
      const int *a = &b;
      int *b = a; // 这样是会报错的
      
      
      // 你这样写之后 就可以修改b的指向了
      int d = 321;
      int c = 123;
      const int *a = &c;
      int *b = const_cast<int*>(a);
  
      b = &d;
  
      cout << *b << endl;
  ```

  

- reinterpret_cast

- 



# 函数

## 函数声明

函数使用之前需要使用函数声明，函数声明不需要写函数体，用一个分号代替函数体就可以。

**我们的函数声明一般放在头文件中比价合适**

```cpp
#include "Sales_data.h"

int main() {
    int a = func();

    cout << a << endl;
}

int func() {
    return 1;
}

// Sales_data.h文件
int func();
```

## 分离式编译

举个例子，假设fact函数的定义位于一个名为fact.cc的文件中，它的声明位于名为Chapter6.h的头文件中。显然与其他所有用到fact函数的文件一样，fact.cc应该包含Chapter6.h头文件。另外，我们在名为factMain.cc的文件中创建main函数，main函数将调用fact函数。要生成可执行文件（executable file），必须告诉编译器我们用到的代码在哪里。对于上述几个文件来说

![image-20211123141404101](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211123141404101.png)

分离式编译优点在于：当我们的fact函数有变动那么我们只需要重新编译fact文件就可以了。

大多数的编译器提供了这种分离式编译的机制，window上通常会产生一个后缀名.obj，unix上.o文件。后缀名的含义是该文件包含对象代码

接下编译器负责把对象文件链接在一起行程可执行文件

![image-20211123141442906](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211123141442906.png)

## 分离式编译步骤

- 将.cpp文件编译为对象文件，.obj等
- 将对象文件链接到可执行文件

## g++下的分离编译

分离式编译例子：

```cpp
// index.cpp
#include <stdio.h>
#include <iostream>
#include "Sales_data.h"

using namespace std;

int main() {
    int a = funcShare();

    cout << a << endl;
}

// share.cpp
#include "Sales_data.h"

int funcShare() {
    return 1;
}

// Sales_data.h
int funcShare();


// 编译指令
g++ -o index index.cpp
g++ -o index index.cpp // 这条指令不行，他会找不到index.cpp
```

```cpp
// 分离式编译指令添加-c参数，执行完这条指令之后就会产生index.o和share.o文件
g++ -c index.cpp share.cpp
// 将他们链接为可执行文件
g++ index.o share.o -o index
    
// 如果下一次share.cpp发生了改变，那么们只需要执行接可以了
g++ -c share.cpp
g++ index.o share.o -o index
```

## 参数传递

### 传递指针

```cpp
void funcPointer(int *p) {
    *p = 3;
}
```

### 传递引用

```cpp
void funcRef(int &p) {
    p = 2;
}
```

传递引用和传递指针，效果差不多

### 避免使用拷贝

假设当我们需要把一个较长的字符串作为参数传入函数的时候，我们尽量通过引用传参，而不是值传参。避免了拷贝带来的开销

const 的意思式这个参数只读不写

```cpp
void copyString(const string &str1, const string &str2)
```

## 函数的返回类型

### 返回引用类型

```cpp
int &smaller(int &i, int &j) {
    return i > j ? i : j;
};

int main() {
    int a = 1;
    int b = 2;
    
    int &small = smaller(a, b);

    small = 4;
	// 这种方式b是可以被正确修改的
    cout << b << endl;
    cout << small << endl;
}
```

### 不要返回局部引用或者指针

```cpp
const int &smaller() {
    const int c = 0;
    return c;
};
// 这种写法是不行的，因为c是一个内部变量，函数结束之后这个内部变量的存储空间就会被回收，这个函数会报错
int main() {
    int a = 1;
    int b = 2;
    
    const int &small = smaller();

    cout << small << endl;

}
```

这种写法是不行的，因为c是一个内部变量，函数结束之后这个内部变量的存储空间就会被回收，这个函数会报错，**同样返回一个指针也是错误的**

### 引用返回左值





## 使用引用形参返回额外信息

## const形参和实参

当你的函数参数是一const的时候，你的实参可以是const类型，也可以不是const类型。换句话说顶层的const被忽略掉了

```cpp
void funcPointer(const int *p) {
    int c = 1;
    int *b = &c;

    p = b;
}


int main() {
    int a = 2;
    int *p = &a;
    funcPointer(p);

    cout << *p << endl;
}
```

我们可以使用一个非const类型的变量去初始化一个const类型变量，但是反过来不行。

## 尽量使用常量

- 给人一种误解函数可以修改他的实参值

- 使得你传入的参数收到了限制，你不能传入**const类型，字面量**给普通应用形参

  ```
  void funcPointer(const int &p) {
  }
  
  funcPointer(1); // 编译通过
  
  /////
  void funcPointer(int &p) {
  }
  
  funcPointer(1); // 编译不通过
  ```

- 有时候会有意想不到的错误。假如说你的函数被另外一个函数调用但是这个函数的形参是一个const，你不是const，那么传值给你就会翻车

  ```cpp
  void funcPointer(int &p) {
  
  }
  
  void otherFunc(const int &p) {
      funcPointer(p); // 编译不通过，一个const无法传值给非const
  }
  ```



## 数组形参

数组的两个性质对于我们定义和使用作用在数组上的函数有影响

- 不允许拷贝数组，我们无法以值传递的形式使用数组参数

  ```
  int a[1] = {1};
  
  int b[1] = a;
  ```

- 使用数组的时候将转成指针，所以我们为函数传递一个数组时，实际上传递的是指向数组首元素的指针

因为数组有这个几个特点，所以这三种定义数组的格式是等价的

```cpp
void arrFunc(int *) {}
void arrFunc(int []) {}
void arrFunc(int [10]) {}
```

从这个int * 看除，其实我们的数组被看成了一个整形的指针，所以我们一下几种入参都是有效的：

```cpp
void arrFunc(const int [10]) {}

int main() {
    int c = 1;
    int a[1] = {1};
    int *b = &c;

    arrFunc(b);
    arrFunc(a);
    arrFunc(&c);
}
```

因为数组传递本质上传的是指针，所以我们无法知道数组的长度，所以，我们遍历的时候就需要用到一些技巧

- c语言遍历法，不推荐

- 传入size长度，一般

- 使用标准库规范，我们传入数组的首指针和尾指针，推荐

  ```cpp
  void print(const int *begin, const int *end) {
      while (begin != end) {
          cout << *begin << endl;
          begin++;
      }
  }
  
  int a[2] = {1, 2};
  print(begin(a), end(a));
  ```

## 数组形参和const

如果我们不需要修改数组的元素值，那么我们就把参数定为const。

如果说我们确实有修改数组值得医院，那么我们可以不要const

## 数组引用形参

## 含有可变形参的函数

- 假如说可变形参的类型是一样的，那么我们可以使用`initializer_list`库
- 假如说类型不同，就是用函数重载



`initializer_list<string>`和vector一样是一个泛型，你需要提前传入里面的而类型是什么，但是他和vector不同的地方在于，里面的元素是不能够更改的

```cpp
void errorMessage(initializer_list<string> li) {
    for (auto beg = begin(li), endless = end(li); beg != endless; beg++) {
        cout << *beg << endl;
    }
}

errorMessage({a, b, "c"});
```

## 省略符形参

## 函数重载

当你的声明出现了同名的函数，但是参数列表不相同的时候，这个就是函数重载

当时使用这个函数的时候，他就会从这么多个函数里面去找，找到参数匹配那个就对了

### 重载和const形参

顶层const无法和没有顶层const的形参区分开

```cpp
void test(int a);
void test(const int a);
// 这两种是一样的区分不开
```

如果你的参数是指针或者引用，这个时候const又是有效的

```
void test(int &a);
void test(const int &a);
// 这样又是可以区分的
```

### const_cast和重载

const_cast在重载函数的情境中最有用

```cpp
const string &test(const string &a, const string &b) {
    return a.size() > b.size() ? a : b;
}
```

这个函数接受两个const 字符串引用，返回的一个const字符串引用，假如说我们

把参数改为非const的

```cpp
const string &test(string &a, string &b) {
    return a.size() > b.size() ? a : b;
}
```

我们返回的依旧是一个const类型的，如果说我们想返回一个不是const类型的那怎么办呢

```cpp
string &test(string &a, string &b) {
    auto &r = test(const_cast<const string&>(a), const_cast<const string&>(b));

    return const_cast<string&>(r);
}
```

## 默认实参

类似于js的默认赋初值

```cpp
int defaultFunc(int height = 3, int width = 4) {
    return height > width ? height : width;
}
```

如果说当你默认赋初值了，你调用的时候可以不传入参数，这个时候编译器不会报错

如果说你没有进行默认赋初值，那么你不传入参数就会发生错误

```cpp
int ht() {
    return 1;
}

int defaultFunc(int height = ht(), int width = 4) {
    return height > width ? height : width;
}
// 你搞成一个函数也是合法的
```

## 内联函数和constexpr函数

### 内联函数作用背景

我们代码中可能存在大量的一些工具函数，他只有一行：

```cpp
int test(int a, int b) {
	return a > b ? a : b
}
```

这个函数有一个缺陷，可能当我们使用这个函数的时候，函数调用的开销可能远超过了表达式计算的开销。这样显然是划不来的

这个时候我们就可以使用内联函数，他是作用可以帮我我们编译成为例如：

![image-20211126155932169](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211126155932169.png)

这样子就没有了函数的调用的开销

### 如何定义内联函数

在函数前面加上一个inline就可以了

```js
inline int defaultFunc(int height = ht(), int width = 4) {
    return height > width ? height : width;
}
```

### constexpr函数

这个函数是指能用于常量表达式的函数

定义constexpr函数有几条规则：

- 函数的返回类型以及所有的形参类型都是字面值类型
- 函数体有且只有一条return语句

```cpp
constexpr int haha () {return 1;}
```

**编译的时候，编译器会把constexpr函数调用的替换成为结果值。**

## 调试帮助

### assert

assert是一种预定义宏，assert使用一个表达式作为他的条件，首先对表达式进行求值，如果false，输出信息中止程序执行，true就什么都不做

```cpp
#include <stdio.h>
#include <assert.h>
int main(void)
{
   int  i;
   i = 1;
   assert(i++);
   printf(“%d\n”,i);
   return 0;
}
```

![image-20211126161744616](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211126161744616.png)

### NDEBUG

asset的行为依赖于一个叫走NDEBUG的预处理变量。如果定义了NDEBUG那么assert就不会起作用，如果没有定义，assert就会执行运行时检查

通过这个语句开启`#define NDEBUG` 调式模式

## 函数匹配

对于一些重载的函数系统内部有自己的一套匹配的机制

## 实参的类型转换

## 函数指针

函数指针指向的是函数而非对象

形如：`int (*p)(int, int)`

```cpp
int defaultFunc(int height, int width) {
    return height > width ? height : width;
}

constexpr int haha () {return 1;}

int main() {
    int (*p)(int, int);

    p = defaultFunc;
    // 等价上面
    p = &defaultFunc;

    int a = p(1,2);
    // 等价上面，不需要解引用
    int b = (*p)(1,2);

    cout << a << endl;
    cout << b << endl;
}
```

### 作用

他的作用就是让**我们把函数当作参数传入函数中**

```cpp
void print(int a) {
    cout << a << endl;
}

int defaultFunc(int height, int width, void (*p)(const int)) {
    p(1);
    return height > width ? height : width;
}

defaultFunc(1,2, print);
```

- cpp不允许我们返回返回，但是我们可以通过返回指向函数的指针的形式来返回函数

## 返回指向函数的指针





# 类

```cpp
struct Sale_data {
    string BookName;
    string isbn() const { return BookName; }
    Sale_data &combin (const Sale_data &);
}
```

`return BookName`这里默认省略了this对象

引入const成员是什么含义，**不允许修改this里面成员的指向**，默认情况下this的类型是指向类类型非常量版本的常量指针。例如在Sales_data成员函数中，this的类型是Sales_data ＊const。

一般来说我们是不需要改动我们的this，这个时候我们可以把this声明为常量，这个时候就可以在函数结束的末尾加上一个const，意思就是这个this是常量的。

像这样使用const的成员函数被称作常量成员函数

## 类外部定义成员函数

```cpp
struct Sale_data {
    string BookName;
    string isbn() const { return BookName; }
    double avg_price() const;
    Sale_data &combin (const Sale_data &);
};
// 你能够这么定义的前提是你struct内部已经有avg_priced
double Sale_data::avg_price() const  {
    return 1.1;
}
```

## 类的构造函数

构造函数和类名相同和其他函数不同，他没有返回类型。

类可以定义多个构造函数和函数重载差不多，不同构造函数参数有不同的地方

```cpp
struct Sale_data {
    Sale_data() = default;
    Sale_data(const string &s): BookNo(s) {}
    string BookName;
    string BookNo;
    string isbn() const { return BookName; }
    double avg_price() const;
    Sale_data &combin (const Sale_data &);
};

int main() {
    // 这种方式不需要new
    Sale_data a = Sale_data("213");
    cout << a.BookNo << endl;
}
```

`Sale_data() = default;`这个东西如果在你**定义了自定义的构造函数**的时候就写上吧，防止各种的异常行为，不解释了。

`Sale_data(const string &s): BookNo(s)`中的`: BookNo(s)`意思是使用s对BookNo进行初始化。

函数的体为空，原因在于只进行了一个数据的初始化操作，没有其他逻辑执行，如果你有其他逻辑执行，那你可以在函数体中加东西。

## 拷贝，赋值和析构

## 访问控制与封装

对于访问控制符出现的次数没有限制

```cpp
class Sale_data {
public:
    Sale_data() = default;
    Sale_data(const string &s): BookNo(s) {}
    string BookName;
private:
    string BookNo;
    string isbn() const { return BookName; }
    double avg_price() const;
    Sale_data &combin (const Sale_data &);
};
```

### struct和class区别

其实都差不多，区别：

- class需要new，struct不需要
- struct再出现第一个访问说明符之前所有的成员都是pulic，而class相反，都是private
- 所以，当我们希望所有成员都是public的时候可以使用struct，所以成员都是private时候就是用class

## 友元

对于一些private的公园，我们就无法在类的外部使用它。对于一些非共有成员如果我们想从外部访问它，方法就是领其他类或者函数成为它的友元。增加一个friend关键字就可以了。

```cpp
class Sale_data {
friend double avg_price();
public:
    Sale_data() = default;
    Sale_data(const string &s): BookNo(s) {}
    string BookName;
private:
    string BookNo;
    string isbn() const { return BookName; }
    Sale_data &combin (const Sale_data &);
};

double avg_price() {
    return 1.1;
}

int main() {
    Sale_data a = Sale_data("213");
    avg_price();
}
```

从友元函数的调用和声明可以看出，它其实不属于类的成员，他连this都访问不到，本质上他和外部定义的一个函数似乎没有什么区别。

**但是友元函数一个最大的特点就是可以通过参数来访问类的私有成员**

```cpp
class Sale_data {
friend void avg_price(Sale_data);
public:
    Sale_data() = default;
    Sale_data(const string &s): BookName(s) {}
private:
    string BookNo;
    string BookName;
    string isbn() const { return BookName; }
    Sale_data &combin (const Sale_data &);
};

void avg_price(Sale_data x) {
    // 通过友元函数访问是合法的
    cout << x.BookName << endl;
}

int main() {
    Sale_data a = Sale_data("213");
    avg_price(a);
}
```

## 类的其他特性

### 定义一个类型成员

有两种方法，可以使用typedef或者使用using

```cpp
class Sale_data {
friend void avg_price(Sale_data);
public:
    Sale_data() = default;
    Sale_data(const string &s): BookName(s) {}
    typedef string pos;
    using newString = string;
private:
    string BookNo;
    string BookName;
    pos Date;
    newString ReturnDate;
    string isbn() const { return BookName; }
    Sale_data &combin (const Sale_data &);
};
```

### 重载成员函数

和普通的函数重载是一样的

### 可变数据成员

有时我们希望修改某个数据成员，即使是在一个const成员函数内。可以通过再变量的声明中加入mutable关键字做到这一点。

```cpp
class Sale_data {
friend void avg_price(Sale_data);
public:
    void some_member() const;
private:
    int accessCount = 0;
};

void Sale_data::some_member() const {
	// 声明了const直接修改accessCount是不合法的
    accessCount++;
    //mutable int accessCount = 0;
}
```

通过mutable进行改写`mutable int accessCount = 0;`改成这样的话就是合法的了。

### 类数据成员的初始值

省略

### 返回*this的成员函数

这个东西十分存在迷惑性，返回值有&和没有&是完全不同的

```cpp

class Screen {
    public:
        Screen() = default;
        Screen(int ht, int wd, int c): height(ht), width(wd), content(ht * wd) {}
        int pointer = 1;
        Screen &set() {
            return *this;
        }

        Screen &changePointer() {
            pointer = 2;
            return *this;
        }
    private:
        int height = 0;
        int width = 0;
        int content;
};

int main() {
    Screen screen = Screen();

    Screen a = screen.set().changePointer();
    
    cout << screen.pointer << endl;
    cout << a.pointer << endl;
}
```

`Screen &set()`看这段代码，这个返回值是带有&号的，说明返回的是引用，这个时候我们调用changePointer方法本质上screen对象是同一个对象，所以，输出是2，2

假如说`Screen set()`没有&号，那么虽然你返回的是this，但是你返回的本质上是一个副本，你对副本操作是没有办法改变screen对象的。所以输出是1，2

假如说`Screen a = screen.set().changePointer();`改为：

```cpp
    Screen a = screen.set();

    Screen b = a.changePointer();
```

**从js的角度其实是一样的，但是cpp的角度虽然你的返回值带了&，但是本质上这个时候它把a，还是看作是副本**，所以最后输出是1，2，没有修改成功。

通过地址判断这两个东西确实不是一个东西

```cpp
if (&(screen.set()) == &screen) {
   cout << "111111" << endl;
} // true

// false
if (&a == &screen) {
   cout << "相同" << endl;
}

// 有点神奇
```

**似乎只要做过了赋值操作，就是拷贝出来副本，这个和js完全不同**

### 从const成员返回*this

假如代码修改成为这样

```cpp
const Screen &set() {
    return *this;
}
// 你尝试这么去调用就会报错，因为return是一个const，所以你无权修改他的成员
Screen a = screen.set().changePointer();
// 但是你这么写就是能够通过编译的，通过这个说明这上面那一点，这个a必定是一个副本
Screen a = screen.set();
a.changePointer();
```

### 基于const重载

你可以同时定义一个返回const类型的版本和一个非const的版本，所以这个叫做const重载

```cpp
        // 不加后面的const会报错
		const Screen &set() const {
            return *this;
        }

        Screen &set() {
            return *this;
        }
```

这个写法是合法的，那**调用哪一个版本取决于你是怎么定义screen**

```cpp
    Screen a = screen.set();// 调用非常量版本
    const Screen a = screen.set(); // 调用常量版本
```

### 其他友元元素

友元不仅可以是函数，还可以是一个类，如果是一个类的话，那么这个友元类可以访问包含类的包括非私有成员的所有东西

```cpp
class Screen {
    friend class window;
    public:
        Screen() = default;
        Screen(int ht, int wd, int c): height(ht), width(wd), content(ht * wd) {}
        int pointer = 1;
        const Screen &set() const {
            return *this;
        }

        void changePointer() {
            pointer = 2;
        }
    private:
        int height = 0;
        int width = 0;
        int content;
};

class window {
    public:
        window() = default;

        void vistor() {
            Screen screen = Screen();

            cout << screen.pointer << endl;
            cout << screen.height << endl;
        }
};

int main() {
    window win = window();

    win.vistor();

}
```

可以令整一个类作为元素，也可以令类的某个一函数作为友元

```cpp
class Screen {
    friend void window::vistor();
};
```

改成这样也是合法的

## 类的作用域

类型名特殊处理，一般来说内层作用域可以重新定义外层作用域的东西，但是对于类类型不然。

```cpp
typedef int Money;

class Account {
    public: 
        Money val;
        Money blance() {
            typedef double Money;
            return val;
        }
};
```

这段代码在类里面重新定义了一次money类型，**这种行为是错误的，但是编译器不会告诉你是错误的，编译一样通过**

**类型名的定义通常出现在类的开始处，这样就能确保所有使用该类型的成员都出现在类名的定义之后。**

## 构造函数

### 构造函数初始化和区别

初始化是指，类似这种语句

```cpp
Screen(int ht, int wd, int c): height(ht), width(wd), content(ht * wd) {}
```

赋值是指这种语句

```cpp
Screen::Screen(int a) {
    height = a;
}
```

这两种东西都是对构造函数的操作，但是其实本质上还是有区别的，对于成员函数是**const类型或者是一个引用**，那么复制语句将会发生错误。const语句显然定义了就不能够改变指向，引用了类型

### 构造函数初始化顺序

c++对于构造函数参数的初始化顺序没有什么特别的要求，你不一定要按照参数列表的顺序初始化参数。

**但是如果你的初始化过程是前后依赖的话，那么这个初始化顺序就比较关键了**

```cpp
        Person(int a): age(a), name(age) {};
        int name;
        int age;
```

这个例子看起来，像是先用a初始化了age，然后age初始化name。

**但是实际上是name先被初始化，但是这个时候显然age是没有被初始化的。所以这个初始化的结构是不符合预期的**

所以最后初始化顺序和声明的顺序是一致的

### 默认实参和构造函数

这两种写法是相同的效果

```cpp
class Person {
    public: 
        Person(int a = 1): age(a) {};
        int age;
};

int main() {
    // 这两种调用的方式是一样的
    Person tony = Person();
    Person jenry = Person(1);
}
```

### 委托构造函数

这个意思就是：某个类使用它所属类的其他构造函数初始化他自己的过程

一个委托构造函数也有一个成员初始值的列表和一个函数体。

```cpp
class Person {
    public: 
        int age;
        int sex;
        Person(int a, int b): age(a), sex(b) {};
        Person(int c): Person(c, 3) {
            cout << "hhhhh" << endl;
        };
        Person(): Person(4) {}
};
```

这段代码除了第一个构造函数以外，剩下两个都是委托构造函数。

第二个构造函数委托第一个构造函数，进行初始化，然后执行hhhh

第三个构造函数委托第二个构造函数进行初始化

执行规律：

当一个构造函数委托给另一个构造函数时，受委托的构造函数的初始值列表和函数体被依次执行。

### 默认构造函数作用

- 当我们不使用任何初始化去初始化类的时候

  ```cpp
  class Person {
      public: 
          int age;
          int sex;
          static int c;
          Person(int a, int b): age(a), sex(b) {};
  };
  // 这种情况下如果没有默认构造函数就会报错
  Person tony = Person();
  ```

- 当一个类本身含有类类型的成员且使用合成的默认构造函数（不懂）

- 当类类型的成员没有在构造函数初始值列表显示地初始化时，举个例子

  ```cpp
  class Person {
      public: 
          int age;
          int sex;
  };
  // 假如你是这样初始化类的，这样也是合法的不需要 xxx = person()
  Person tony;
  cout << tony.age << endl; // 这个打印就是不正常的
  
  // 加上一个默认构造函数的话那你你的打印才是可控符合预期的
  class Person {
      public: 
          Person() {
              age = 1;
          };
          int age;
          int sex;
  };
  ```

总结：不管什么时候加上默认构造函数就对了

### 使用默认构造函数的易错点

```cpp
// 有人可能这么初始化类，这个是错误的，编译器把他看成是一个函数声明
Person tony();
// 这个才是正确的
Person tony;
```

### 隐式类类型转换

意思就是**你通过一个传入的实参却调用构造函数**。看一个例子

```cpp
class Person {
    public: 
        int age;
        Person(int newAge) {
            age = newAge;
        }
        bool isTheSameAge(Person tony) {
            return tony.age == age;
        }
};

int main() {
    Person tony = Person(1);
    Person henry = Person(2);
    bool res = tony.isTheSameAge(henry);

    cout << res << endl;
}
```

这个main函数里面的传参是符合预期的

```cpp
int main() {
    Person tony = Person(1);

    bool res = tony.isTheSameAge(1);

    cout << res << endl;
}
```

假如我改为这样`isTheSameAge`函数里面传入一个1，理论上来说这样子传参是不合法的。但是cpp可以借助构造函数的能力，**隐式帮我们调用构造函数，生成一个person对象，传进行比较**。所以上面的写法能够得出正确结果的

我们可以通过在构造函数中加上`explicit`关键字，这样子就可以抑制这种转换行为

```cpp
class Person {
    public: 
        int age;
        explicit Person(int newAge) {
            age = newAge;
        }
        bool isTheSameAge(Person tony) {
            return tony.age == age;
        }
};
```

### 聚合类

聚合类是的用户可以直接访问其成员，**并且具有特殊的初始化语法**

满足以下条件的就是聚合类：

- 所有成员都是public
- 没有定义任何构造函数
- 没有类初始值
- 没有基类和虚函数

举个例子：

```cpp
class Data {
    public:
        int a;
        int b;
};

int main() {
    Data val = {1, 2};

    cout << val.a << endl;
}
```

`Data val = {1, 2};`这种就是**特殊的初始化语法**，按照成员的顺序进行初始化

### 字面值常量

## 类的静态static

### 静态函数

静态成员函数不包含this指针，这个和js是一样的

cpp和js一样也拥有静态成员

```cpp
class Data {
    public:
        static int getData() {
            return 1;
        }
        int a = 1 ;
        int b;
};

int main() {
    Data val;
	// 居然这么调用也是合法的？
    int a = val.getData();
    int b = Data::getData();
    // 通过指针调用也是可以的
    Data *c = &val;

    cout << b << endl;
}
```

### 静态成员

对于类的静态成员，我们不能够在内的构造函数对静态成员进行初始化，我们只能在定义的时候初始化或者在外部初始化

```cpp
class Data {
    public:
        static int getData() {
            return 1;
        }
        int a = 1 ;
        static int b = 1;
};

Data::b = 2;
```

### 静态成员的类内初始化

```cpp
class Data {
    public:
        static int getData() {
            return 1;
        }
        int a = 1 ;
        static int b = 1;
        int array[b] = {1};
};
// 上面那种初始化使用b的方式是不合法的，需要改正这样才是合法的
class Data {
    public:
        static int getData() {
            return 1;
        }
        int a = 1 ;
        static constexpr int b = 1;
        int array[b] = {1};
};
```

# c++标准库

## IO库

三种不同的io库

- iostream，定义了读写流的基本类型
- fsstream，定义了读写命名文件类型
- sstream，定义了读写内存string对象类型

![image-20211230115455503](C:\Users\admini\AppData\Roaming\Typora\typora-user-images\image-20211230115455503.png)

### io对象无拷贝或者复制

```cpp
    ofstream out1, out2;
    out1 = out2; // 错误
```

### 检查一个流状态是否良好

使用一个条件判断：

```cpp
while (cin >> word) {

}
```

### 使用文件流对象

### string流

# 序容器

## 顺序容器

一个容器就是一些特定类型对象的集合。顺序容器为程序提供控制元素存储和访问顺序的能力。他以来元素值二十一来假如容器时候位置相对应

顺序容器列表：

- vector
- deque 双向队列
- forward_list：单向链表
- array
- string

每一个容器都定义在一个头文件中，文件名字和类型名字对应

# 泛型算法

这个泛型算法和我理解的泛型是不同的。出现这个算法的背景在于：

c++中有很多的这些容器对象，每个容器对象定义的操作又不同，对于某些公共需求，可能某种数据结构又不提供这个方法。

所以就出现了这个泛型算法，把这些不同数据结构操作的差异性统一起来

## find查找函数

```cpp
#include <algorithm>

int main() {
    vector<int> v1 = {1,2,3,4};
    int val = 4;

    auto result = find(v1.cbegin(), v1.cend(), val);

    if (result == v1.cend()) {
        cout << 11 << endl;
    } else {
        cout << 22 << endl;
    }
}
```

## 只读算法

accumulate，这个函数可以用来求和

```cpp
#include <numeric>

int main() {
    vector<int> v1 = {1,2,3,4};
    int val = 4;

    int res = accumulate(v1.cbegin(), v1.cend(), 0);

    cout << res << endl;
}
```



equal，用来确定两个序列是否保存相同的值。他把第一个序列的每个元素和第二个序列的对应的元素进行比较，如果对应的元素相同返回true，否则返回false.

他接受三个参数，前两个参数表示第一个序列的范围，第三个参数是第二个序列的首元素

```cpp
int main() {
    vector<int> v1 = {1,2,3,4};
    vector<int> v2 = {1,2,3,5};
    int val = 4;

    bool res = equal(v1.cbegin(), v1.cend(), v2.cbegin());

    cout << res << endl;
}
```

## 写容器元素的算法

```cpp
fill(v1.cbegin(), v1.cbegin() + v1.size / 2, 0);
```

将容器的一般的元素重置为0

## back_inserter

这个东西可以帮助我们在迭代器后面插入一个元素

```cpp
auto it = back_inserter(v1);
*it = 2;
cout << v1.at(v1.size()-1) << endl;
```

## 拷贝算法

拷贝算法是**目的位置迭代器指向输出序列中元素写入数据**

```cpp
int main() {
    int a1[] = {1,2,3};
    int a2[3];

    auto ret = copy(begin(a1), end(a1), a2);

    for (auto ele : a2) {
        cout << ele << endl;
    }   
}
```

## 排序算法

sort

## 定制操作

sort还接受第三个参数，可以定制sort操作。这个操作叫做谓词

谓词：是一个可调用的表达式，返回结果是一个能用做条件的值（其实和js是一样的，但是在c++中的叫法不一样而已）

谓词分为两类：

- 一元谓词，意味着他只收一个参数
- 二元谓词，意味着他接受两个参数

写一个二元谓词例子：

```cpp
bool isShort(string s1, string s2) {
    return s1.size() < s2.size();
}

int main() {
    string a3[] = {"123", "32", "1"};

    sort(begin(a3), end(a3), isShort);

    for (auto ele : a3) {
        cout << ele << endl;
    }  
}
```

## lambda表达式

### 基础使用

我们可以向算法传递任何类别的可调用对象，**可调用对象的含义就是对于一个对象和表达式如果能够像函数一样使用()让他执行的话，就叫做可调用对象**

比较典型的可调用对象就是：函数和**函数指针**，意味着我们上面的例子不一定你要传入一个函数进去，你可以传入一个函数指针进去。

另外一个中可调用对象就是lambda表达式，lambda表达式的格式是：

`[capture list](参数列表)- > {函数体}`

捕获列表是一个lambda所在函数中定义的局部变量列表，通常是null的

```cpp
int main() {
    auto f = [](int a){ return a; };

    cout << f(1) << endl;
}
```

```cpp
int main() {
    string a3[] = {"123", "32", "1"};

    sort(begin(a3), end(a3), [](string s1, string s2) {
        return s1.size() < s2.size();
    });

    for (auto ele : a3) {
        cout << ele << endl;
    }
}
```



使用捕获列表

`[sz](int a){ return a > sz; };`这个sz是怎么来的呢，他是外部定义来的，举另外一个例子：

```cpp
int main() {
    int a = 1;
    auto f = [a]() { return a; };
    cout << f() << endl;   
}
```



### lambda是捕获返回的

#### 值捕获

```cpp
int main() {
    int a = 1;
    auto f = [a]() { return a; };
    cout << f() << endl;   
}
```

#### 引用捕获

```cpp
    int a = 1;

    auto f = [&a]() { return a; };

    a = 2;
    int b = f();

    cout << b << endl;
```

### 隐式捕获

除了显示列出我们希望使用的变量之外，我们还可以让编译器根据lambda体中的代码来推断我们要使用哪些变量。为了指示编译器，我们可以在捕获列表中写一个&或者=。

&告诉编译器采用捕获引用的方式

=是采用值捕获

```cpp
    int a = 1;

    auto f = [&]() { return a; };

    int b = f();
    b = 2 ;
    cout << b << endl;

/////////////////
    int a = 1;

    auto f = [=]() { return a; };

    int b = f();
    b = 2 ;
    cout << a << endl;
```

我们也可以混用显示和隐式捕获

### 可变lambda

一般来说我们lambda函数体内是不允许改变变量值的，如果你试图改变它，那么就会发生报错

```cpp
int main() {
    int a = 1;
	// 这样子改变是会报错的
    auto f = [=]() { return a++; };

    int b = f();
    b = 2 ;
    cout << a << endl;   
}
```

如果你真的想改变可以假如mutable

```cpp
int main() {
    int a = 1;

    auto f = [=]() mutable { return a++; };

    int b = f();
    b = 2 ;
    cout << a << endl;  
}
```

### 指定lambda的返回类型

默认情况下lambda体包含return之外的任何语句，则编译器假定此lambda返回void.

如果你只是一个单一的return语句，那么lambda可以根据你返回的东西判断出来你的返回类型是什么

```cpp
// 例如这个返回的就是int
auto f = [=]() { return a; };
```

`auto f = [=]() { if (1) { return } else { return } };`

如果你是这么写的就会发生报错，因为你包含了return意外其他语句，默认返回void

`auto f = [a]() -> string { if (a) { return 1; } else { return 2;} };`

你可以通过这样指定你的返回类型

# 关联容器

关联容器和顺序容器有着根本的不同，关联容器是按照关键字来保存的。就是js中的对象或者map

## 关联容器类型

他们存在于头文件map和set中

按照关键字有序保存元素：

- map
- set
- multimap 关键字可以重复的map
- multiset 关键字可以重复的set

无序集合

- unordered_map 用哈希函数存储
- unordered_set
- unordered_multimap
- unordered_multiset

## 使用关联容器

使用map：

```cpp
int main() {
    map<string, int> wordCount;

    wordCount["123"] = 1;
    wordCount["321"] = 2;

    for (auto &ele : wordCount) {
        cout << ele.first << endl;
        cout << ele.second << endl;
    } 
}
```

使用set，打印在exclude中存在的键

```cpp
int main() {
    map<string, int> wordCount;
    set<string> exclude = {"123"};

    wordCount["123"] = 1;
    wordCount["321"] = 2;
    
    // 也可这样来进行初始化
    wordCount = { {"xxx", "xxx"}, {"xxx", "xxx"};

    for (auto &ele : wordCount) {
        if (exclude.find(ele.first) != exclude.end()) {
            cout << ele.first << endl;
        }
    } 
}
```

## 初始化set方法

字面量法：就是上面例子中一个一个列举出来

```cpp
    vector<string> v1 = {"123", "321"};
    set<string> exclude(v1.begin(), v1.end());
```



## 关联容器特点

关联容器没有像顺序容器一样的插入操作等。但他它有一些顺序容器没有的操作，无序容器还提供了一些调整哈希性能的操作

## 关键字类型的要求

有序容器-map，set这些关键字类型必须定义元素的比较方法，默认情况下使用关键字类型的 < 比较。

我们可以提供一个算法自己定义这个比较操作，我们的比较函数不许满足一些特性

## pair类型

pair类型定义在头文件utility中。pair用来生成特定的模板，他需要提供两个类型名，两个类型名不要求是一样的

```cpp
int main (int argc, char *argv[])
{
  pair<string, string> atom;
}
```

这样写没有指令初始化内容的话，pair的默认构造函数会对pair进行默认初始化，上面的例子包含两个空string的pair

我们可以提供初始化器，通过first和second对下面的元素进行访问：

```cpp
int main (int argc, char *argv[])
{
  pair<string, string> atom{"123", "321"};
  // 这样也是可以的
  pair<string, string> atom("123", "321");
  // 这样也是可以的
  pair<string, string> atom = {"123", "321"};
  cout << atom.first << endl;
  cout << atom.second << endl;
}
```

### pair类型操作

`make_pair(v1, v2)`，使用v1，v2进行初始化返回一个pair类型，自动根据v1，v2类型推断pair类型

`p1 == p2`，当first和second纯官员分别相等时两个pair宪哥的那个

`p1 != p2`

### 通过函数初始化pair

```cpp
pair<string, int>
process(vector<string> &v) {
  if (!v.empty()) {
    return {v.back(), v.back().size()};
  } else {
    return pair<string, int>();
  }
}

int main (int argc, char *argv[])
{
  vector<string> vec = {"213", "3211"};
  
  // process返回pair类型
  pair<string, int> atom = process(vec); 

  cout << atom.first << endl;
  cout << atom.second << endl;
}
```

**当我们便利或者使用begin或者last操作的时候，map的返回值就是一个pair类型，我们可以通过first和second使用里面的值**

## 关联容器操作

### 遍历操作

可以使用while或者for循环都是可以的

```cpp
  while (mapBegin != test.cend()) {
    cout << mapBegin->first << endl;
    cout << mapBegin->second << endl;
    mapBegin++;
  }
```



### 添加元素

通过insert

```cpp
int main (int argc, char *argv[])
{
  map<int, string> test;
  test.insert(make_pair(1, "2"));
  test.insert(make_pair(2, "3"));
  auto mapBegin = test.cbegin();

  for_each(test.cbegin(),test.cend(),[](const pair<int,string> &it)
                {
                    cout<<"first:"<<it.first<<" second:"<<it.second<<endl;
  });
}
```

**注意点：你添加的时候第一个元素如果是一样的，即使后面的值不同打印出来的只会有一个元素。例如第一个insert改称为`test.insert(make_pair(1, "3"));`那么只会打印`1，2`**

### 删除元素

删除方法erase，他接受一个key_type参数，此版本删除所有匹配给定关键字元素，返回值是0或者1。0说明删除元素不存在容器中，1表示存在

```cpp
int main (int argc, char *argv[])
{
  map<int, string> test;
  test.insert(make_pair(1, "2"));
  test.insert(make_pair(2, "3"));
  auto mapBegin = test.cbegin();
  test.erase(1);
  for_each(test.cbegin(),test.cend(),[](const pair<int,string> &it)
                {
                    cout<<"first:"<<it.first<<" second:"<<it.second<<endl;
  });
}
```

### map下标操作

`c[k]`

`c.at(k)`

### 访问元素

find，我们值关心某个元素是否存在于容器中

```cpp
int main (int argc, char *argv[])
{
  set<int> v = {1,2};

  // 返回一个迭代器指向key == 1的元素
  v.find(1);
  // count区别于find，count还可以知道有多少个这个key的元素存在于容器中
  v.count(2);
}
```

![image-20220104114822601](C:\Users\admini\AppData\Roaming\Typora\typora-user-images\image-20220104114822601.png)

## 无序容器

这些容器不使用比较运算符来组织元素而是使用一个**哈希函数和关键字类型的==运算符**。某些情境下维护元素的序列的代价是十分高的，此时无序容器就显得比较有用

无序容器提供的操作和有序容器的类似的

定义的是 `unordered_map<string, int> map`

### 管理桶

无序容器在存储上组织为一组桶，每个桶保存零个或者多个元素。无序容器使用哈希函数将元素映射到桶。为了访问每一个元素，容器首先计算元素的哈希值，他指出应该搜索哪个桶。容器将具有特定哈希值的所有元素保存在相同的桶中。如果容器允许重复的关键字，所有具有相同关键字的元素也都会在同一个桶中。**因此无序容器性能以来于哈希函数的质量和桶的数量和大小**

计算一个哈希值和在桶中的搜索通常都是十分快的，但是假如说一个桶里面存放了十分多的元素，那么当你需要顺序查找的某个特定的元素的时候这个就需要比较大量的操作

### 管理桶的函数

桶接口：

```cpp
  unordered_map<string, int> map;  
  // 正在使用的桶的数目
  map.bucket_count();
  // 容器能容纳的最多的桶的数量
  map.max_bucket_count();
  // 第n个桶中有多少个元素
  map.bucket_size(n);
  // 关键字k存放在第几个桶中
  map.bucket(k);
  // 每个桶的平均元素数量，返回float值
  map.load_factor();
  // map试图维护的平均桶的大小，返回float值，map会在需要时添加新的桶，是的load_factor <= max_load_factor
  map.max_load_factor();
  // 重组存储，使得bucket_count >= n
  map.rehash(n);
  // 重组存储，是的c可以保存n个元素且不必rehash
  map.reserve(n)
```

### 无序容器对类型关键字的要求

默认情况下无序容器使用关键字类型的==运算比较元素，由于c++里面为内置类型提供了hash模板了，所以我们是可以直接使用内置类型作为无序容器的关键字。

但是不能直接使用自定义类型作为关键字，因为他没有hash模板。我们需要自己写hash模板

那么如果我们需要使用自定义类型的话，我们需要提供两个函数的实现：

- 一个函数代替==运算符
- 一个函数计算hash

具体写法还没有懂

# 动态内存

目前我们写的代码大多数都是使用的时候分配，使用完了就会释放。c++还支持动态内存分配，这个和前面那些的区别就是，**由用户来指定释放的时期，而不是用完就没了**

目前我们使用到的内存为**静态内存**和**栈内存**静态内存用来保存一些static定义的变量和任何函数以外的全局变量。栈内存保存非static对象和函数里面定义的变量，**栈内存和静态内存都是由编译器自动销毁和自动创建**

除了上面两个内存还有一个内存池，这部分被叫做**自由空间**或者**堆**。堆内存用来存储动态分配的对象——就是程序运行时分配的对象，动态对象的生存期由程序来控制，当它不再使用的时候，我们的代码必须显示销毁他

管理他们是十分麻烦的

## 动态存储与智能指针

动态内存管理通过**new关键字**完成。delete可以用来销毁这个对象，这个和js是一样的。

c++为了我们辅助我们管理这些动态内存。它提供了两种智能指针用来管理动态对象。智能指针行为和常规指针差不多，**不同在于智能指针负责自动释放所指向的对象**。

- shared_ptr允许多个指针指向同一个对象
- unique_ptr则“独占”所指向的对象
- weak_ptr，他是一个弱引用，指向shared_ptr所管理的对象

他们定义在memory头文件中

## shared_ptr类

智能指针是一个模板，我们创建它的时候需要提供指针可以指向的类型

`shared_ptr<string> p1`

用法和普通的指针是一样的

shared_ptr和unique_ptr支持的操作

- p，用作判断，若p指向一个对象返回true
- *p,，解引用
- p->mem
- `p.get()`返回p中保存的指针，如果指针释放了对象，**那么返回的指针所指向的对象就消失了**
- swap(p, q)，交换p和q的指针
- `p.swap(q)`,效果和上面一样

shared_ptr独有操作

- `make_shared<T>(args)`返回一个shared_ptr，指向一个动态分配的类型T对象，使用args初始化对象
- `shared_ptr<T>p(q)`p是shared_ptr q的拷贝，这个操作会递增q的引用计数器
- p = q，这个操作和递减p的引用计数，递增q的应用技术，若p的引用技术变为0那么他管理的内存就会被释放
- `p.unique()`若p.use_count()为1，返回true，否则false
- p.use_count()，返回与p共享对象的智能指针的数量，可能很慢，用于debug

## make_shared函数

make_shared函数是最安全的分配和使用动态内存，这个函数在动态内存中分配一个对象并且初始化他，返回指向这个对象的shared_ptr。

```cpp
// 指向42的int的shared_ptr
shared_ptr<int> p1 = make_shared<int>(42);
auto q(p)// p和q指向相同的对象这个对象有两个引用
```

## 使用动态生存期的资源的类

- 程序不知道自己需要使用多少对象
- 程序不知道所需对象的准确类型
- 程序需要在多个对象间共享数据

### 定义一个类，它使用动态内存共享相同底层数据

我们目前使用过的类中，一般分配的资源都是和对应的生存期是一致的。

```cpp
  vector<int> v1;
  {
    vector<int> v2 = {1,2,3};
    v1 = v2
  }
```

这段代码，v2赋值给v1，但是v2的生命周期结束了，**v2会被释放并且它里面的元素也一样会被释放，v1里面保存的是v2对象里面的副本**

另外一个例子

```cpp
  Blob<string> b1;
  {
    Blob<string> b2 = {1,2,3};
    b1 = b2
  }
```

这段代码，v2赋值给v1，**虽然v2的生命周期结束了，但是b2里面的元素不会被释放，b1里面的元素指向的底层元素和b2是一样的**

### 自定义strBlob体验共享内存

```cpp
class StrBlob {
  public:
    typedef vector<string>::size_type size_type;
    StrBlob(): data(make_shared<vector<string>>()) {}
    StrBlob(initializer_list<string> il): data(make_shared<vector<string>>(il)) {}
    size_type size() const {
      return data->size();
    }
    bool empty() const {
      return data->empty();
    }
    void push_back(const string &t) {
      data->push_back(t);
    }
    void pop_back() {
      check(0, "pushback on empty strblob");
      data->pop_back();
    };
    string &front() {
      check(0, "front on empty strblob");
      return data->front();
    };
    string &back() {
      check(0, "back on empty strblob");
      return data->back();
    };
  private:
    shared_ptr<vector<string>> data;
    void check(size_type i, const string &msg) const {
      if (i >= data->size()) {
        throw out_of_range(msg);
      }
    };
};
```

## 直接管理内存

c++语言定义了连个运算符来分配和释放动态内存：

- new分配
- delete释放

相对于智能指针，这两个运算符的管理十分容易出问题

### 使用new初始化对象

```cpp
 // p1指向一个动态分配的，未初始化的无名对象
 int *p1 = new int;
```

new表达式在自由空间构造一个int型对象。并返回指向该对象的指针

```cpp
string *ps1 = new string //默认初始化为空string
string *ps2 = new string() // 值初始化为空string
int  *p1 = new int; // 默认初始化，p1未定义
int  *p2 = new int(); //值初始化0
```

对于定义了自己的构造函数的类类型来说，要求值初始化是没有意义的；不管采用什么方式，对象都会通过默认构造函数初始化，就像上面的string。

对于内置类型就像int，默认初始化和值初始化就不一样了，默认初始化就是未定义，值初始化会有一个默认值赋给它

### auto

我们可以使用auto进行初始化，让编译器来推断我们初始化的类型

```
auto p1 = new auto(obj)// p指向一个与obj类型相同的对象
auto p2 = new auto(a,b,c) // 错误
```

如果obj是int，那么p1就是int，string就是string

### new const

使用new分配给一个const对象是合法的

`const int *p = new const int(1024);`

类似其他const对象，一个动态分配的const对象必须进行初始化，

## 内存耗尽

一旦一个程序用光了所有可用内存，new表达式就会失败

## 释放动态内存

通过delete释放内存

`delete p`

假如我们对一个非new的内存进行delete或者将相同的指针值释放多次，其行为都是未定义的

```cpp
int main (int argc, char *argv[])
{
  int i;
  int *p = &i;
  int *p1 = new int(1024);

  // 错误，i不是指针
  delete i;
  // 未定义，P1指向的变量不是new
  delete p;
  // 正确
  delete p1;
  // 未定义，p1已经被释放了
  delete p1;
}
```

const new也是可以被释放的

## 动态对象的生存期直到被释放时为止

对于使用share_ptr这些智能指针创建出来的对象，到生命周期结束之后它会被自动释放，但是对于new这种的就需要手动释放才会被释放

```cpp
Foo *factory() {
  return new Foo();
}

void useFoo (int argc, char *argv[])
{
  Foo *p = factory();
}
```

**对这个函数就是useFoo生命周期结束了，这个p也不会被回收**，所以js对于堆内存有自己的一套自动回收的算法

## 小结

尽量不要使用这个new，尽量使用智能指针

## shared_ptr和new结合使用

```cpp
shared_ptr<int> p2(new int(42));
```

