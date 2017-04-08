---
title: block之变量的存取
date: 2016-03-11 20:14:33
tags:
---
** Block作为C语言的扩展，并不是高新技术，和其他语言的闭包或lambda表达式是一回事。Block被Obj-C看成是对象，它封装了一段代码，这段代码可以在任何时候执行。Block可以作为函数参数或者函数的返回值，而其本身又可以带输入参数或返回值。它和传统的函数指针很类似，但是有区别：Block是inline的，并且它对局部变量是只读的。**

** 接下来对Block对变量的访问和修改进行分析。**

#### 1.对局部变量的存取

下面先看代码

{% codeblock lang:objc %}
{    
    int outA = 8;    
    int (^myPtr)(int) = ^(int a){ return outA + a;};    
    //block里面可以读取同一类型的outA的值    
    int result = myPtr(3);  // result is 11    
    NSLog(@"result=%d", result);    
}  
{% endcodeblock %}
当我在调用"int result = myPtr(3);" 之前将outA的值修改为其它值时，result的值是否会改变呢?

{% codeblock lang:objc %}
{    
    int outA = 8;    
    int (^myPtr)(int) = ^(int a){ return outA + a;}; //block里面可以读取同一类型的outA的值    
        
    outA = 5;  //在调用myPtr之前改变outA的值    
    int result = myPtr(3);  // result的值仍然是11，并不是8    
    NSLog(@"result=%d", result);    
}  
{% endcodeblock %}
为什么result 的值仍然是11？而不是8呢？事实上，myPtr在其主体中用到的outA这个变量值的时候做了一个copy的动作，把outA的值copy下来。所以，之后outA即使换成了新的值，对于myPtr里面copy的值是没有影响的。
需要注意的是，这里copy的值是变量的值，如果它是一个记忆体的位置（地址），换句话说，就是这个变量是个指针的话，

它的值是可以在block里被改变的。如下例子：

{% codeblock lang:objc %}
{    
    NSMutableArray *mutableArray = [NSMutableArray arrayWithObjects:@"one", @"two", @"three", nil nil];    
    int result = ^(int a){[mutableArray removeLastObject]; return a*a;}(5);    
    NSLog(@"test array :%@", mutableArray);    
}  
{% endcodeblock %}
原本mutableArray的值是{@"one",@"two",@"three"}，在block里面被更改mutableArray后，就变成{@"one", @"two"}了。

#### 2.对static类型修饰的变量的存取
{% codeblock lang:objc %}
{    
    static int outA = 8;    
    int (^myPtr)(int) = ^(int a){return outA + a;};    
    outA = 5;    
    int result = myPtr(3);  //result的值是8，因为outA是static类型的变量    
    NSLog(@"result=%d", result);      
} 
{% endcodeblock %}
甚至可以直接在block里面修改outA的值，例如下面的写法：

{% codeblock lang:objc %}
{    
    static int outA = 8;    
    int (^myPtr)(int) = ^(int a){ outA = 5; return outA + a;};    
    int result = myPtr(3);  //result的值是8，因为outA是static类型的变量    
    NSLog(@"result=%d", result);    
}  

{% endcodeblock %}
#### 3. 对__block类型修饰的变量的存取
在某个变量前面如果加上修饰字“__block”的话（注意，block前面有两个下划线），那么在block里面就可以任意修改此变量的值，如下代码：

{% codeblock lang:objc %}
{    
    __block int num = 5;    
    int (^myPtr)(int) = ^(int a){return num++;};    
    int (^myPtr2)(int) = ^(int a){return num++;};    
    int result = myPtr(0);   //result的值为5，num的值为6    
    result = myPtr2(0);      //result的值为6，num的值为7    
    NSLog(@"result=%d", result);     
}  
{% endcodeblock %}


