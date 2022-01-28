[TOC]

#### 扫描目录
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