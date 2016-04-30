---
layout: post
title: 标准输出的缓冲（行缓冲、全缓冲...）
comments: true
category: C
---

写ICS作业碰到的一些问题，题目是关于文件读写的，题目涉及到的标准IO缓冲有点奇妙，查了点资料在这里做一下整理。


题目是要求写出下面程序的输出（哪些输出到文件，哪些输出的屏幕）

{% highlight c %}
int main()
{
    int fd1, fd2, fd3, fd4, fd5;
    int status;
    pid_t pid;
    char c;
    /* foo.txt 123456 */
    fd1 = open("foo.txt", O_RDONLY, 0);
    fd2 = open("foo.txt", O_RDONLY, 0);
    /* bar.txt abcdef */
    fd3 = open("bar.txt", O_RDWR, 0);
    fd4 = open("alice.txt", O_WRONLY | O_CREAT | O_APPEND, S_IRUSR | S_IWUSR);
    fd5 = dup(STDOUT_FILENO);
    dup2(fd4, STDOUT_FILENO);
    if ( (pid = fork()) == 0 ) {
        dup2(fd2, fd3);
        read(fd3, &c, 1); printf("%c", c);
        write(fd4, &c, 1);
        read(fd1, &c, 1); printf("%c", c);
        read(fd2, &c, 1); printf("%c", c);
        exit(0);
    }
    wait(NULL);
    read(fd1, &c, 1); printf("%c", c);
    read(fd3, &c, 1); printf("%c", c);
    dup2(fd5, STDOUT_FILENO);
    printf("done.\n");
    return 0;
}
{% endhighlight %}

好吧其实上面程序大部分和缓冲没关系，而且上面代码有点长看着有点烦，作个简化:

{% highlight c %}
int main()
{
    int fd1, fd4, fd5;
    char c;
    /* foo.txt 123456 */
    fd1 = open("foo.txt", O_RDONLY, 0);
    fd4 = open("alice.txt", O_WRONLY | O_CREAT | O_APPEND, S_IRUSR | S_IWUSR);
    fd5 = dup(STDOUT_FILENO);
    dup2(fd4, STDOUT_FILENO);
    read(fd1, &c, 1); printf("%c", c);
    read(fd1, &c, 1); printf("%c", c);
    dup2(fd5, STDOUT_FILENO);
    printf("done.\n");
    return 0;
}
{% endhighlight %}

上面这个程序输出是

```
Terminal:
12done.

(alice.txt文件为空，没有输出)
```

上面代码主要做了下面几件事情：

1. 打开foo.txt（只读），打开alice.txt（写）
3. 复制了一个标准输出，得到文件描述符fd5
4. 把标准输出重定向fd4，也就是定向到alice.txt
5. 从foo.txt读字符，用`printf`输出
6. 把标准输出重定向回来（从alice.txt定向到标准输出）
7. `printf`输出done,程序结束

按道理上面的程序应该是是把12输出到alice.txt里，然后终端显示done. 但是实际运行却不是这样，我就想是不是缓存的原因。我就把两个输出改成了下面这样：

{% highlight c %}
read(fd1, &c, 1); printf("%c\n", c);
read(fd1, &c, 1); printf("%c\n", c);
{% endhighlight %}

以前debug深受`printf\cout`缓存的折磨，后来懂事了会用回车来刷缓存。

但是!这里用了回车，结果还是一样，alice.txt里面是空的，全部输出到了屏幕。

当时我就惊呆了。。。难道`printf`这东西还会自己保留一个文件描述符不成，然后开始去找它源码。。。虽然没在源码里面找到什么东西。。。但是找到了两个缓存的概念，**行缓存、全缓存**。然后就能顺利解释了上面的结果了。

* *全缓存*：当标准IO缓冲区满了以后（或者程序结束）才刷出缓存；可以使用`fflush()`强行刷出缓存。**对于磁盘上文件的IO操作通常用全缓存**
* *行缓存*：当遇到换行符时，标准IO刷出缓存。**当流涉及到一个终端时，通常使用行缓存**。

所以上面的程序，当我们把标准输出重定位到alice.txt（文件），`printf`用的是**全缓存**，即使用了`\n`也不会刷出缓存。所以两次输出都存在缓存里，等到程序运行结束，标准输出流已经定位回来了，不再对准alice.txt，刷出到终端。

所以可以想见，如果我把程序改成这样

{% highlight c %}
//fflush(NULL);刷出所有输出流缓存
read(fd1, &c, 1); printf("%c\n", c);fflush(NULL);
read(fd1, &c, 1); printf("%c\n", c);fflush(NULL);
{% endhighlight %}

结果就是

```
alice.txt:
12   
Terminal:
done.
```

最后贴一个迷之代码，来自[lxmky](http://blog.csdn.net/lxmky/article/details/5457324)

{% highlight c %}
//try.c
#include<stdio.h>
#include<unistd.h>
int glob=6;
char buf[]="a write ro stdout/n";
int main()
{
    int var;
    pid_t pid;
    printf("a write to stdout/n");
    if((pid=fork())<0)
    	printf("fork error");
    else
    {
        if(pid==0)
        {
            glob++;
            var++;
        }
        else
            sleep(2);
    }
    printf("pid=%d,glob=%d,var=%d/n", getpid(), glob, var);
    exit(0);
}
{% endhighlight %}

用不同方式运行上面的代码，运行会不一样

```
>./try > a.out
a.out:
a write to stdout
pid=6591,glob=7,var=134514042
a write to stdout
pid=6590,glob=6,var=134514041

>./try
a write to stdout
pid=19147,glob=1,var=4195985
pid=19146,glob=0,var=4195984
```

原因就是文件用的是全缓冲，终端用的是行缓存。

