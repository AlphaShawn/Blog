---
layout: post
title: cout的诡异之处
comments: true
category: C++
---

写小程序的意外发现的cout输出顺序的问题，之前也碰到过，时间久了就容易忘记。在这里记一个。

来看下面代码

{% highlight c++ %}
int f(int &x, int &y) {
  x++;
  y++;
  return x+y;
}

int main()
{
	int x = 3, y = 4;
  cout<<f(x,y)<<' '<<x<<' '<<y<<endl;
}
{% endhighlight %}

代码很简单，但是结果有点神奇。

第一眼一看就会觉得是`7 4 5`，但是运行输出是`7 3 4`。可以看到输出的`x`, `y`的值是调用`f()`之前的值，问题在哪？

查了网上的资料，原因在于对于`cout`,待输出的值**从右往左计算，然后从左往右输出**.

如果不知道这个点，然后被坑到了，确实还是要折腾一段时间才能知道问题在哪。

比如下面一个队列的代码，就很容易被坑

{% highlight c++ %}
MyQueue q;
q.push(3);
q.push(5);
cout<<q.pop()<<" "<<q.pop();
{% endhighlight %}

进队以后，队列`q:3 5`,则出队时候先5再3.

但是经过`cout`以后，反而输出是`5 3`.

