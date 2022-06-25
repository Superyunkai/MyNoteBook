[TOC]

 #  GCC编译及MakeFile语法



 ## 1. 编译选项

 
   ```
        -fno-elide-constructiors    关闭编译器省略复制构造的优化
   ```
 ## 2. 编译器特性

 ### 2.1. 编译器delete用法
```C++
    class T
    {
        &T operator=(&T) = delete;
    }
```
在赋值构造函数和拷贝构造函数后，指明不允许编译器自动生成相关代码，防止在多线程或其它条件下，造成潜在的资源生命周期问题。例如在包含一个thread_t成员变量的类内部，如果使用拷贝构造函数或者赋值构造函数，thread_t被复制后，其生命周期由新产生的类变量决定，但其thread_t指向的线程可能在第一个类中已经结束。thread_t







 #  现代C++语义
 ## 1. 移动语义

+ 引入：C++ 11标准引入，是C++11的重大改动之一
+ 可拷贝和可移动的概念
  在面向对象编程中, 有的类是可拷贝的资源，例如一般的变量（string，ptr, int……), 而有些对象的或者其资源是独一无二的，所有权应该属于且仅属于一个对象。例如（IO、std::unique_ptr, thread_t)，他们不可以被复制，但是可以把资源的所有权交给新的对象，称为可移动的。
+ 作用：在一些构造时需要申请大量资源的对象，可以获取已有的资源而不需要申请新的资源再拷贝，这样的移动而非拷贝构造会大幅度的提升性能。例如构造函数传入的是一个即将消亡的右值，可以用移动语义接管它的资源。
+ 例子
  考虑如下一个类：
  ```C++
        #include <iostream>
        #include <cstring>
        using namespace std;
        class A{
            public:
            A():field(new int[5000]){
                cout<<"class A construct"<< endl;
            }
            A(const A &a):field(new int[5000]){
                memcpy(field,a.field, 5000 * sizeof(int));
                cout<<"class A copy!"<<endl;

            }
            ~A(){
                delete[] field;
                cout<<"clas A destruct"<<endl;
            }
            private:
                int * field;
        }

  ```   
  它在每次构造时需要重新申请空间，如果作为函数的返回值，在编译器关闭省略复制构造的优化选项时，会有一次构造和两次拷贝，在每次拷贝中都要重新申请内存，且拷贝后申请的对象又很快被释放，这无疑是很浪费的。
  **在类中增加移动构造函数，减少这种浪费**
  ```C++
  A(A &&a) noexcept
    :field(a.field)
    {
        a.field = nullptr;
        cout<<"Class A move"<<endl;

    }
    A &operator =(A &&rhs) noexcept{
      // check self assignment
      if(this != &rhs){
         delete []i;
         i = rhs.i;
         rhs.i = nullptr;
      }
      cout<< "class A move and assignment"<<std::endl;
      return *this;
   }
  ```
+ 总结
  1 移动构造语义，主要是把复制对象的资源的所有权给新的对象，实现一个类似浅拷贝的操作
  2 需要把原先右值引用的指针成员置空，及取消原有右值引用成员的所有权，防止资源被释放
  3 需要检查是否存在自赋值，然后才能先delet自己的成员再浅拷贝

 ### 1.1 std::move()
+ 作用： 把一个左值转换为右值，并承诺原值不再使用，即废弃
  
+ 实现：
```C++
    // FUNCTION TEMPLATE move
    template <class _Ty>
    _NODISCARD constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept { // forward _Arg as movable
        return static_cast<remove_reference_t<_Ty>&&>(_Arg);
    }
```
    
 ## 2. explict 关键字
   
+ explicit关键字指定该构造函数或转换函数为显式(C+++11)，即禁止隐式调用或复制初始化。
+ explicit指定符可以与常量表达式一同使用，当表达式为ture时才为显示(C++20)
  考虑以下代码：
```c++
    #include <iostream>
    using namespace std;

    class Point {
    public:
        int x, y;
        explicit Point(int x = 0, int y = 0)
            : x(x), y(y) {}
    };

    void displayPoint(const Point& p) 
    {
        cout << "(" << p.x << "," 
            << p.y << ")" << endl;
    }

    int main()
    {
        displayPoint(1);
        Point p = 1;  //can't compile
    }    
```
## 3. extern 关键字
extern 关键字可以置于变量和函数之前，以标示变量或者函数的定义在别的文件，提示编译器遇到此函数和变量时在其他模块中寻找。此外extern也可以用来进行链接指定。用以屏蔽编译器保持当前函数名称，不生成用以链接的中间函数名

## 4. constexpr 关键字

这个关键字在C++11中引入，并在C++14中增强了它的功能。它代表一个常量的表达式，与const相同，它可以应用于一个变量，当任何试图修改其值的行为发生时，编译器会报错。此外，constexpr还可以应用在函数和类的构造函数中，指明返回的为const类型，某些时候可以在编译时确认

## 5. optional 
The need for "Sometime-a-thing"
The need for "Yet-a-thing"
主要是对缺省值的优化，假设有一个函数make_any_int(int a), 它接受一个int类型参数的同时，也希望它无参数调用。通常的方法有声明函数时：
make_any_int(int a = -1),当a = -1 时则认为是一个无效参数。
但如果想要参数a覆盖所有整数的范围，一般有两种方式：
1. make_any_int(int a = -1, bool bvailed = false)
2. make_any_int(int * a = NULL)

上面两种都可以实现目标但都存在一定的缺陷，第一种需要额外输入一个新的参数，而且用户很容易忘记第二个参数，因为它没有实际意义，如果优化成pair的方式，则有可能用户在两个参数的顺序上出错。第二种方式则并不完全符合原有与语义，我们有时候只是想输入一个整型常量

另一种情况是我们的类持有的资源并不立刻初始化，而是延迟初始化。 对这个资源的访问与是否初始化有关，此时需要一个额外的成员变量来标识是否初始化。而是用optional可以规避这个问题。

对于第一种情况，std::nullopt 表示无效的opt，可以通过显示的调用has_value()方法来检查optional实例是否具有值。当你确定该optional中有值时，也可以直接用*获取。optional支持<,>,<=,>= 等操作符，std:nullopt小于所有非空实例。

```C++
void maybe_take_an_int(optional<int> potential_value = nullopt); 
     // or equivalently, "potential_value = {}"
opional<int> maybe_return_an_int();
optional<int> o = maybe_return_an_int();
if (o.has_value()) { /* ... */ }
if (o) { /* ... */ } // "if" converts its condition to bool
if (o) { cout << "The value is: " << *o << '\n'; }
```

对于第二种情况：
```C++
using T = /* some object type */;

struct S {
  optional<T> maybe_T;    

  void construct_the_T(int arg) {
    // We need not guard against repeat initialization;
    // optional's emplace member will destroy any 
    // contained object and make a fresh one.        
    maybe_T.emplace(arg);
  }

  T& get_the_T() { 
    assert(maybe_T);
    return *maybe_T;    
    // Or, if we prefer an exception when maybe_T is not initialized:
    // return maybe_T.value();
  }

  // ... No error-prone handwritten special member functions! ...
};
```

 #  C++标准库函数及系统函数

 ##  1. 扫描目录

``` C++
    #include <dirent.h>
    int scandir(const char *restrict dirp, struct dirent ***restrict namelist, int (*filter)(const struct dirent *), int (*compar)(const struct dirent**, const struct dirent **));
    int alphasort(const struct dirent **a, const struct dirent **b);
    int versionsort(const struct dirent **a, const struct dirent **b);
    /*  函数说明
    *scandir()函数扫描dirp目录下的文件列表，对于每个文件项，使用filter()函数过滤。对于每个目录项，filter()返回非0值时，将该目录项保存到 namelist 中。
    *如果filter为NULL，则代表选择所有的目录项。
    *目录项 namelist由 malloc分配内存，结束后需要记得手动 free。
    *对于得到的目录项会使用qsort()进行排序，排序函数由compar指定。
    *提供了两个比较函数：1.alphsort():类似于ls，使用strcoll()，对于jan1，jan2，jan3，jan10，jan12这样的文件列表排序不正确
    *                  2.versionsort():类似于ls -v ,使用strverscmp(),忽略文件列表间的相同前缀(glibc>=2.1)
    */

    // \return -1 for Error, >=0 for 文件数量
    
```
### std::move()
+ 作用： 把一个左值转换为右值，并承诺原值不再使用，即废弃
  
+ 实现：
```C++
    // FUNCTION TEMPLATE move
    template <class _Ty>
    _NODISCARD constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept { // forward _Arg as movable
        return static_cast<remove_reference_t<_Ty>&&>(_Arg);
    }
```
### std::find()

+ Defined in header <algorithm>
+ 函数原型:
```C++
Defined:
    template< class InputIt, class T >
    InputIt find( InputIt first, InputIt last, const T& value );

Parameters:
    first, last - the range of the elements to examine
          value - value to compare the elements to
        
```

##  2. 判断文件是否存在

头文件: <sys/types.h>
        <sys/stat.h>
        <unistd.h>

函数原型： 
```C++
int stat(const char *path, struct stat*buf)
int fstat(int fd, struct stat *buf)
int lstat(const char *path, struct stat *buf)
```

描述:以上函数返回文件的相关信息。对文件权限没有要求，但是当使用stat和lstat时，其文件所在目录需要有执行权限。三个函数的区别在于传入文件的类型：
1. stat():路径
2. lstat():链接文件, 显示该链接文件的信息，而不是链接指向的文件的信息
3. fstat(): 文件描述符

获得的内容:
```C++
struct stat {
    dev_t     st_dev;     /* ID of device containing file */
    ino_t     st_ino;     /* inode number */
    mode_t    st_mode;    /* protection */
    nlink_t   st_nlink;   /* number of hard links */
    uid_t     st_uid;     /* user ID of owner */
    gid_t     st_gid;     /* group ID of owner */
    dev_t     st_rdev;    /* device ID (if special file) */
    off_t     st_size;    /* total size, in bytes */
    blksize_t st_blksize; /* blocksize for file system I/O */
    blkcnt_t  st_blocks;  /* number of 512B blocks allocated */
    time_t    st_atime;   /* time of last access */
    time_t    st_mtime;   /* time of last modification */
    time_t    st_ctime;   /* time of last status change */
};
```
[更多内容见](https://linux.die.net/man/2/stat)

返回值：0-succ  -1-err  设置errno
常见errno:
EACCES      path的路径前缀目录之一的搜索权限被拒绝
EBADF       fd 错误
EFAULT      地址错误
ELOOP       过多符号链接
ENAMETOOLONG path过长
ENOENT      路径为空或不存在
ENOMEM      超出内存
ENOTDIR     path的前缀目录之一是普通文件
EOVERFLOW   文件大小超出范围

#  C++代码实现技巧

##  1. 柔性数组（flexible array)
C99标准中，允许结构体的最后一个元素是未知大小的数组，成为柔性数组成员。但是结构体中柔性数组成员前必须有一个其他成员。
柔性数组成员的结构允许结构体包含一个大小可变的数组。sizeof操作符返回的这种结构体的大小不包括改柔性数组的大小。
包含柔性数组成员函数的结构要用malloc函数进行内存的动态分配，并且分配的内存大小应该大于结构体的大小
```C++
    typdef struct st_type
    {
        int i;
        char a[];
    }type_a;
    sizeof(type_a) = 4
    type_a *p = (type_a*)malloc(sizeof(type_a) + 100 * sizeof(char))
```

## 2. 指针
+ 函数指针: typedef void (*pf)(int, int)
+ restrict 关键词是一个限定词，可以被用在指针上，它向编译器保证，在这个指针的生命周期内，通过这个指针访问的内存，都只能被这个指针修改。

## 3. Varint 表示法
简介：Varint是一种紧凑型数字表示法，使用一个或多个字节的内存来表示整数。比如对于uint32_t类型的数字，通常是需要4byte来表示，较小的数字甚至可以用1byte来表示，而较大的数使用5个byte来表示。从统计的角度来说，较小的数字出现的频率较高，项目中也不可能都是较大的数字，所以使用这中方法可以节省空间
实现：每个byte的最高位有特殊含义，如果为1，则表示后续的byte也为这个数字的一部分，为0则表示结束。也就是说128以下的数字只需要一个byte， 而大于128的需要两个byte。如果特别大的数字需要5个字节， 使用Varint表示法时所需空间：n, 表示的数字大小为：2^n*7

编码实现：
```C++
char* EncodeVarint32(char* dst, uint32_t v) {
  // Operate on characters as unsigneds
  unsigned char* ptr = reinterpret_cast<unsigned char*>(dst);
  static const int B = 128;
  // 0000 0101 0010 1000
  // 0000 0000 1000 0000
  if (v < (1<<7)) {
    *(ptr++) = v;
  } else if (v < (1<<14)) {
    // 0000 01010 1010 1000
    *(ptr++) = v | B;
    *(ptr++) = v>>7;
  } else if (v < (1<<21)) {
    *(ptr++) = v | B;
    *(ptr++) = (v>>7) | B;
    *(ptr++) = v>>14;
  } else if (v < (1<<28)) {
    *(ptr++) = v | B;
    *(ptr++) = (v>>7) | B;
    *(ptr++) = (v>>14) | B;
    *(ptr++) = v>>21;
  } else {
    *(ptr++) = v | B;
    *(ptr++) = (v>>7) | B;
    *(ptr++) = (v>>14) | B;
    *(ptr++) = (v>>21) | B;
    *(ptr++) = v>>28;
  }
  return reinterpret_cast<char*>(ptr);
}
```
## 4. 可变参数与可变宏
### 4.1可变函数参数
可变参数的函数最常见的就是printf()和scanf()。它的实现主要依赖以下几个宏
+ val_list   
+ val_start
+ val_end
这几个宏有C库函数<stdarg.h>提供
```C++
bool  func(char *params, ...)
{
    //创建变参列表
    val_list paramlist;
    // 初始化，获取最后一个实参的下一个参数的指针
    val_start(paramlist, params);
    while(paramlist != NULL)
    {
        // 以指定类型获取参数，并将paramlist指向下一个参数
        char * key = va_arg(paramlist, char *);
        if (strcmp(key, "arg_end"))
            break;
    }
    val_end(paramlist);
}
```
以上只能使用在真正的函数上

### 4.2 可变参数宏
在C99编译器中提供了新的宏**__VA_ARGS__**，实现宏的多参数。可变参数宏不被ANSI/ISO C++ 所正式支持。因此，你应当检查你的编译器，看它是否支持这项技术。
其实现思想就是编译器用__VA_ARGS__去替代宏定义中...号的部分，其使用如下

```C++
//在GUNC中实现如下
#define pr_debug(fmt,arg...) \ 
printf(KERN_DEBUG fmt, ##arg)


//C99,目前似乎只有gcc支持
#define debug(...) printf(__VA_ARGS__)
```



#  多线程

## 1 Introduce
``` C++
    #include<thread>
    #include<stdio.h>
    void f()
    {
        cout<<"Hello world"<<endl;
    }
    int main()
    {
        thread t = new thread(f);
        t.jion();
    }
```

 ### 1.1 子线程的生命周期(detach和jion的区别)


detach 后线程彻底脱离主线程的控制，需要谨慎使用，适合用于完全分离的两个任务，例如编辑器的两个窗口。jion将子线程和主线程汇合，也就是主线程必须等待子线程结束才能退出，可以保证资源的正确释放

 ## 2 线程的管控
 ### 2.1 线程的基本管控
+ 线程构建
```C++
  void f(){
    //do some thing
  }
  std::thread t(f);
  void f(t1,t2,t3){}
  std::thread t(t1,t2,t3);
  void classA::f(){};
  std::thread t(classA::f, classA* a);
```
这里要注意防范所谓的“C++最麻烦的解释” ，即在传入临时变量而非具名变量时，那么调用构造函数的声明可能与函数声明相同，这时需要用列表初始化语法，或者为临时对象命名，即为临对象多加一对圆括号。

+ 线程的启动与结束
  一旦启动了线程，我们就必须要明确指定是否要等待其运行结束（与之汇合jion)，还是任由它独自运行，如果在thread对象销毁时还没有决定好，thread析构函数会调用treminate()终止进程。
  如果选择了分离，且分离时新线程还没有结束，那么新线程将继续运行，即使thread对象销毁后仍然会继续执行。这种情况下，如果新线程持有指向主线程局部变量的引用或者指针，那么就会造成指针或者引用的空悬。
+ 等待线程的完成
  调用函数jionable()或者jion()实现。jion()会阻塞主线程的运行，直到新线程退出，任何thread对象仅能汇合一次，汇合过后，jionable将会返回false。jion()需要选择合适的位置进行，避免由于前面程序抛出异常而未进行jion()，产生意外的资源问题。
  一个更好的实现是采用RAII方法实现：
  ```c++
    class thread_guard
    {
        std::thread& t;
    public:
        explicit thread_guard(std::thread& t_):t(t_){}
        ~thread_guard()
        {
            if(t.jionable())
            {
                j.jion();
            }
        }
        thread_guard(thread_guard const&)=delete;
        thread_guard& operator=(thread_guard const&)=delete;
    }
  ```

 ### 2.2 向线程传递参数

C++ 中线程的参数传递有一个微妙的地方，新线程具有独立的储存空间，参数会按照默认方式先拷贝到这里，这样新的线程才能访问它，这些拷贝的副本会被当做右值，即使函数的相关参数按设想应该是引用，上述行为仍会发生。这会产生一些影响：
```c++
    void f(int i, std::string const&s);
    void oops(int some_param)
    {
        char buffer[1024];
        sprintf(buffer, sizeof(buffer), "%i", some_param);
        std::thread t(f,3,buffer);
        t.detach();
    }
```
在上面的程序中，向新线程传递了一个指向buffer数组的指针，本来我们设想buffer会在新线程内转换为string类型，但在此完成之前，oops()函数极可能已经退出，造成buffer被销毁而引发未知行为。问题的根源在于我们期望buffer被转换成string对象后再传递给线程，而事实并非如此。解决方法是在调用thread构造函数之前就将buffer转为string对象
```c++
    void f(int i, std::string const&s);
    void oops(int some_param)
    {
        char buffer[1024];
        sprintf(buffer, sizeof(buffer), "%i", some_param);
        std::thread t(f,3,std::string(buffer));
        t.detach();
    }
```
另一种情况是我们期望的是传入一个非const引用或者指针，这时需要使用std::ref()
```c++
    std::thread t(update_data_for_widgest,w, std::ref(data))
```
若传入的是一个对象的成员函数，则还应该传入该对象的指针。
若转入的是一个可移动不可拷贝的对象，则应该使用C++的移动语义。即std::move()

 ### 2.3 线程所属权的转移

thread类支持移动语义的原因在于，有时候希望编写一个函数，作用是创建函数并置于后台，这个函数本身并不等待线程完结，而是将其归属权上交给函数调用者，或者相反的创建一个thread对象，并将它传入一个函数，由它负责等待新线程结束。
```c++
std::thread f()
{
    void some_function();
    return std::thread(some_function);
}
std::thread g()
{
    void some_function(int);
    std::thread t(some_function, 42);
    return t;
}
void f(std::thread t);
void g()
{
    void some_funtion();
    f(std::thread(some_function));
    std::thread t(some_function);
    f(std::move(t));
}
```

 ### 2.4 在运行中确定线程数量

C++标准库std::thread::hardware_concurrency()函数，它返回一个指标，表明程序在各次运行中可真正并发的的线程数量。在多核操作系统上，通常就是系统的核心数，若信息无法获取则会返回0。


 ### 2.5 识别线程
线程ID所属型别是std::thread::id, 它有两种获取方式，一种是在与线程关联的std::thread对象上调用成员函数get_id(), 如果该对象没有关联线程，则会返回一个由默认构造方式生成的thread::id,表示线程不存在。其次，当前线程的id可以用std::this_thread::get_id()获取。


 ## 3 线程间数据共享

 ### 3.1 线程间数据共享的问题
归根结底，这类问题大多是由于数据改动引起，如果所有的共享数据都是读操作，则不会有问题。

#### 3.1.1 条件竞争


## 4.C++多线程内存模型

### 4.1 3种内存模型
考虑如下情况
![](https://pic3.zhimg.com/80/24e4b3d7c8f5d141695ae08dd62a9509_720w.jpg?source%3D1940ef5c)

上面的图描述了一个问题，A进程读X, B进程读X，A进程写X。
对此反映了内存系统两方面的行为：
1. Coherence (连贯性)
2. Consitency (一致性)

第一个方面需要满足以下三个条件：
+ 系统提供我们在单处理器下熟知的程序次序(program order)
+ 写结果最终对所有处理器可见（合适可见未作描述）
+ 系统提供写串行性

对于第二个方面，也称为**内存一致性模型（Sequential Consitency model）**，本质上是限制了读操作的返回值。
直观上来说，读操作的返回值应该是最后一次写操作的返回值
+ 在单处理器中，最后一次写操作由program order决定
+ 在多处理器系统中，称为顺序连贯（sequental Consitency,SC)

SC有两点要求：
1. 在每个处理器内部维护次序
2. 对于所有的处理器来说，维护单一表征的修改顺序。即不能出现两个处理器看到的修改顺序不同。

为了在特定的硬件优化下实现SC模型，程序会做很多特殊处理，包括不限于缓存一致性协议。这些都会消耗性能，同时SC也限制了编译器优化。

为获得更好的性能，引入了放松内存一致性模型(relaxed memory consistency models),这种模型要求更加简单

### 4.2 C++ 的几种内存序
C++内存序实际上是约束同一个线程内的内存访问排序方式，虽然编译器保证在一个线程内的内存序重排不会影响最终的结果，但是在多线程的情况下可能会导致不同的结果。
1. 指令乱序
   
   现在的CPU都采用的是多核、多线程技术用以提升计算能力；采用乱序执行、流水线、分支预测以及多级缓存等方法来提升程序性能。多核技术在提升程序性能的同时，也带来了执行序列乱序和内存序列访问的乱序问题。与此同时，编译器也会基于自己的规则对代码进行优化，这些优化动作也会导致一些代码的顺序被重排。
   ```c++
    int A = 0;
    int B = 0;

    void fun() {
        A = B + 1; // L5
        B = 1; // L6
    }

    int main() {
        fun();
        return 0;
    }
    
    //g++ test.cpp 生成的汇编指令如下
    movl    B(%rip), %eax
    addl    $1, %eax
    movl    %eax, A(%rip)
    movl    $1, B(%rip)

    // 使用O2优化
    movl    B(%rip), %eax
    movl    $1, B(%rip)
    addl    $1, %eax
    movl    %eax, A(%rip)
   ```

2. 几种关系术语
   + evalueation(求值): 分为两个部分：value computations 和 side effects(读写)
   + sequeced-before: 在同一线程内的求值数顺序关系，A sequenced before B, 代表A的求值先完成再对B进行求值, A not sequeced before B , B not sequenced before A,则表示两者都有可能先执行或者同时执行
   + happens-before：sequence-before的扩展，当A happens-before B , A 操作先完成，且A操作的结果对B可见
   + synchronizes-with: 不同线程间的同步关系， 保证A的写操作结果在B中可见

3. C++中的六种内存约束
   ```C++
   typedef enum memory_order {
        memory_order_relaxed,
        memory_order_consume,
        memory_order_acquire,
        memory_order_release,
        memory_order_acq_rel,
        memory_order_seq_cst
    } memory_order;
   ```
   从读写角度划分为以下三种：
   + 读操作（memory_order_acquire, memory_order_consume)
   + 写操作 (memory_order_release)
   + 读写修改操作(memory_order_acq_rel memory_order_seq_cst)
  
   从访问控制的角度分类
   + Sequential consistency模型(memory_order_seq_cst)
   + Relax模型(memory_order_relaxed)
   + Acquire-Release模型(memory_order_consume memory_order_acquire memory_order_release memory_order_acq_rel)

 从访问控制的强弱排序，Sequential consistency模型最强，Acquire-Release模型次之，Relax模型最弱。

4. Sequential consistency 模型
   又称顺序一致性模型，是控制粒度最严格的内存模型，在这个模型下，程序的执行顺序和代码顺序严格一致，也就是说不存在指令乱序
   对应的内存约束符号是memory_order_seq_cst，类似于互斥锁的模式
5. relax 模型
   对应的是memory_order_relax, 其对内存序的限制最小，仅保证当前的数据访问是原子的，但是对内存访问顺序没有任何约束，也就是说可能存在读写的重排。通常用于统计数据。
6. Acquire-Release 模型
   控制力度介于Relax和Sequential consistency模型之间。其包括两面：
   + Acquire: 如果操作X带有acquire语义，则在操作X后面的所有读写指令都不会被重排到X之前。有memory_order_acquire, memory_order_acq_rel
   + Release: 如果操作X带有release语义，则在操作X前的所有读写指令都不会重排到X之前。有memory_order_release, memory_order_acq_rel

+ memory_order_release
  假设有一个原子变量A,对其进行写操作X的时候施加了memory_order_release约束，则在当前线程T1中，操作X之前的任何读写操作指令不能放在操作X之后。当另外一个线程T2对原子变量A进行读操作的时候施加了memory_order_acquire约束符，则当前线程T1中写操作之前的任何读写操作都对线程T2可见；当另外一个线程T2对原子变量A进行读操作的时候，如果施加了memory_order_consume约束符，则当前线程T1中所有原子变量A所依赖的读写操作都对T2线程可见(没有依赖关系的内存操作就不能保证顺序)。
  需要注意的是，对于施加了memory_order_release约束符的写操作，其写之前所有读写指令操作都不会被重排序写操作之后的前提是：其他线程对这个原子变量执行了读操作，且施加了memory_order_acquire或者 memory_order_consume约束符。

+ memory_order_acquire
  一个对原子变量的load操作时，使用memory_order_acquire约束符：在当前线程中，该load之后读和写操作都不能被重排到当前指令前。如果其他线程使用memory_order_release约束符，则对此原子变量进行store操作，在当前线程中是可见的。
  假设有一个原子变量A，如果A的读操作X施加了memory_order_acquire标记，则在当前线程T1中，在操作X之后的所有读写指令都不能重排到操作X之前；当其它线程如果对A进行施加了memory_order_release约束符的写操作Y，则这个写操作Y之前所有的读写指令对当前线程T1是可见的(这里的可见请结合 happens-before 原则理解，即那些内存读写操作会确保完成，不会被重新排序)。也就是说从线程T2的角度来看，在原子变量A写操作之前发生的所有内存写入在线程T1中都会产生作用。也就是说，一旦原子读取完成，线程T1就可以保证看到线程 A 写入内存的所有内容。
+ memory_order_acq_rel
  memory_order_acq_rel，它既可以约束读，也可以约束写。
  对于使用memory_order_acq_rel约束符的原子操作，对当前线程的影响就是：当前线程T1中此操作之前或者之后的内存读写都不能被重新排序（假设此操作之前的操作为操作A，此操作为操作B，此操作之后的操作为B，那么执行顺序总是ABC，这块可以理解为同一线程内的sequenced-before关系）；对其它线程T2的影响是，如果T2线程使用了memory_order_release约束符的写操作，那么T2线程中写操作之前的所有操作均对T1线程可见；如果T2线程使用了memory_order_acquire约束符的读操作，则T1线程的写操作对T2线程可见。
  理解起来可能比较绕，这个标记相当于对读操作使用了memory_order_acquire约束符，对写操作使用了memory_order_release约束符。当前线程中这个操作之前的内存读写不能被重排到这个操作之后，这个操作之后的内存读写也不能被重排到这个操作之前。

+ memory_order_consume
  有一个原子变量A，在线程T1中对原子变量的写操作施加了memory_order_release标记符，同时线程T2对原子变量A的读操作被标记为memory_order_consume，则从线程T1的角度来看，在原子变量写之前发生的所有读写操作，只有与该变量有依赖关系的内存读写才会保证不会重排到这个写操作之后，也就是说，当线程T2使用了带memory_order_consume标记的读操作时，线程T1中只有与这个原子变量有依赖关系的读写操作才不会被重排到写操作之后。而如果读操作施加了memory_order_acquire标记，则线程T1中所有写操作之前的读写操作都不会重排到写之后(此处需要注意的是，一个是有依赖关系的不重排，一个是全部不重排)。

7. 总结
C++11提供的6种内存访问约束符中：

  + memory_order_release：在当前线程T1中，该操作X之前的任何读写操作指令都不能放在操作X之后。如果其它线程对同一变量使用了memory_order_acquire或者memory_order_consume约束符，则当前线程写操作之前的任何读写操作都对其它线程可见(注意consume的话是依赖关系可见)

  + memory_order_acquire：在当前线程中，load操作之后的读和写操作都不能被重排到当前指令前。如果有其他线程使用memory_order_release内存模型对此原子变量进行store操作，在当前线程中是可见的。

  + memory_order_relaxed：没有同步或顺序制约，仅对此操作要求原子性

  + memory_order_consume：在当前线程中，load操作之后的依赖于此原子变量的读和写操作都不能被重排到当前指令前。如果有其他线程使用memory_order_release内存模型对此原子变量进行store操作，在当前线程中是可见的。

  + memory_order_acq_rel：等同于对原子变量同时使用memory_order_release和memory_order_acquire约束符

  + memory_order_seq_cst：从宏观角度看，线程的执行顺序与代码顺序严格一致

  C++的内存模型则是依赖上面六种内存约束符来实现的：

  + Relax模型：对应的是memory_order中的memory_order_relaxed。从其字面意思就能看出，其对于内存序的限制最小，也就是说这种方式只能保证当前的数据访问是原子操作(不会被其他线程的操作打断)，但是对内存访问顺序没有任何约束，也就是说对不同的数据的读写可能会被重新排序

  + Acquire-Release模型：对应的memory_order_consume memory_order_acquire memory_order_release memory_order_acq_rel约束符(需要互相配合使用)；对于一个原子变量A，对A的写操作(Release)和读操作(Acquire)之间进行同步，并建立排序约束关系，即对于写操作(release)X，在写操作X之前的所有读写指令都不能放到写操作X之后；对于读操作(acquire)Y，在读操作Y之后的所有读写指令都不能放到读操作Y之前。

  + Sequential consistency模型：对应的memory_order_seq_cst约束符；程序的执行顺序与代码顺序严格一致，也就是说，在顺序一致性模型中，不存在指令乱序。
  ![alt](../img/C%2B%2B_memory_order.png)




 #  函数对象

 ## 1. 定义
所有能像函数一样使用的对象统称为函数对 ~~x象~~ 111
 ## 2. 分类
1 函数类，即重载了（）操作符的类。
   
   它的优势在于可以使用成员变量保存自己的状态


 #  对象（CLASS)

# 网络编程
## 1. 套接字编程
### 1.1 connect函数在阻塞和非阻塞下的行为
阻塞行为下connect函数会等待一个明确的结果并返回，这通常需要一段时间，在实际编程中我们通常使用更加高效的异步的方式去创建socket
非阻塞变成的流程如下：
1. 创建socket，并设置为非阻塞
2. 调用connect函数，此时connect函数立即返回，如果返回-1不一定代表出错
3. 接着调用select或者poll函数在一定时间内判断改socket是否可写，如果不可写则说明连接失败
代码实现如下：
```C++
    /*
    异步实现connect
    */  
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <arpa/inet.h>
    #include <unistd.h>
    #define SERVER_ADDR "127.0.0.1"
    #define SERVER_PORT 3000
    #define SEND_DATA "...."
    int main()
    {
        if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
        {
            return -1;
        }
        // 设置非阻塞
        if (fcntl(fd, F_SETFL, O_NONBLOCK) == -1)
        {
            return -1;
        }
        
        //连接服务器
        struct sockaddr_in serveraddr;
        serveraddr.sin_family = AF_INET;
        serveraddr.sin_addr.s_addr = inet_addr(SERVER_ADDR);
        serveraddr.sin_port = htons(SERVER_PORT);
        int ret = connect(fd, (struct sockaddr *)&serveraddr, sizeof serveraddr);
        fd_set writeset;
        FD_ZERO(&writeset);
        FD_SET(clientfd, &writeset);
        //可以利用tv_sec和tv_usec做更小精度的超时控制
        struct timeval tv;
        tv.tv_sec = 3;  
        tv.tv_usec = 0;
        //linux 下不仅select可写，而且socket错误码不能为0
        if (select(fd+1, NULL, &writeset, NULL , &tv) != 1)
            return -1;
        if (::getsocketopt(fd, SOL_SOCKET, SOERROR,&err, %len) <0)
            return -1;
        


    }

```
### 1.2 bind函数重难点
bind函数基本使用如下
```C++
struct sockaddr_in bindaddr;
bindaddr.sin_family = AF_INET;
bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
bindaddr.sin_port = htons(3000);
if (bind(listenfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) == -1)
{
    std::cout << "bind listen socket error." << std::endl;
    return -1;
}
```
其中INADDR_ANY是一个宏，其意义是当套接字不关心具体的ip地址时，使用该宏，协议底层会自动选择ip，这样对于多网卡机器有利。
**bind函数的端口问题**
上述例子的服务端口为3000，当端口不重要时，可以填0，会自动分配端口。值的注意的是，通常bind函数由服务方调用，客户端的端口会由操作系统自动分配。而在特殊情况下要求客户端以特定的端口去连接服务器，此时可以在客户端中调用bind函数。示例如下，在客户端代码中
```C++
struct sockaddr_in bindaddr;
bindaddr.sin_family = AF_INET;
bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
//将socket绑定到20000号端口上去
bindaddr.sin_port = htons(20000);
if (bind(clientfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) == -1)
{
    std::cout << "bind socket error." << std::endl;
    return -1;
}
```
启动客户端后，使用lsof -i -Pn 命令查看，可以看到客户端确实以20000端口号连接服务器。此时，启动第二个客户端就会因为端口占用而失败。

