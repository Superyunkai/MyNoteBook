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

 ## 4. std::move()
+ 作用： 把一个左值转换为右值，并承诺原值不再使用，即废弃
  
+ 实现：
```C++
    // FUNCTION TEMPLATE move
    template <class _Ty>
    _NODISCARD constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept { // forward _Arg as movable
        return static_cast<remove_reference_t<_Ty>&&>(_Arg);
    }
```
    
 ## 5. explict 关键字
   
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

 # 5 多线程

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
    条件

 #  函数对象

 ## 1. 定义
所有能像函数一样使用的对象统称为函数对 ~~x象~~ 111
 ## 2. 分类
1 函数类，即重载了（）操作符的类。
   
   它的优势在于可以使用成员变量保存自己的状态


 #  对象（CLASS)

