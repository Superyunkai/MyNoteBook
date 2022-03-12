[TOC]

# C++


## GCC编译及MakeFile语法



### 编译选项

 
   ```
        -fno-elide-constructiors    关闭编译器省略复制构造的优化
   ```
### 编译器特性

#### 编译器delete用法
```C++
    class T
    {
        &T operator=(&T) = delete;
    }
```
在赋值构造函数和拷贝构造函数后，指明不允许编译器自动生成相关代码，防止在多线程或其它条件下，造成潜在的资源生命周期问题。例如在包含一个thread_t成员变量的类内部，如果使用拷贝构造函数或者赋值构造函数，thread_t被复制后，其生命周期由新产生的类变量决定，但其thread_t指向的线程可能在第一个类中已经结束。thread_t







## 现代C++语义
### 移动语义

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
  1. 移动构造语义，主要是把复制对象的资源的所有权给新的对象，实现一个类似浅拷贝的操作
  2. 需要把原先右值引用的指针成员置空，及取消原有右值引用成员的所有权，防止资源被释放
  3. 需要检查是否存在自赋值，然后才能先delet自己的成员再浅拷贝

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
    

## C++标准库函数及系统函数

### 扫描目录

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

## C++代码实现技巧

### 柔性数组（flexible array)
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

### 指针
+ 函数指针: typedef void (*pf)(int, int)
+ restrict 关键词是一个限定词，可以被用在指针上，它向编译器保证，在这个指针的生命周期内，通过这个指针访问的内存，都只能被这个指针修改。

## 多线程

### Introduce
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

### 子线程的生命周期(detach和jion的区别)
    detach 后线程彻底脱离主线程的控制，需要谨慎使用，适合用于完全分离的两个任务，例如编辑器的两个窗口
    jion将子线程和主线程汇合，也就是主线程必须等待子线程结束才能退出，可以保证资源的正确释放

### 线程的管控
#### 线程的基本管控
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



## 函数对象

### 定义
所有能像函数一样使用的对象统称为函数对象

### 分类
1. 函数类，即重载了（）操作符的类。
   
   它的优势在于可以使用成员变量保存自己的状态


## 对象（CLASS)

