---
title: dispatch_once死锁
date: 2016-05-31 16:00:30
tags:
---
在iOS开发中，我们经常会使用到单例，现在Objective-C中写单例的标配是使用dispatch_once。相信这个函数的意义大家都非常清楚了，就是希望dispatch_once参数中的block在全局只执行一次。

不过，有时稍微不留意便会出现问题，比如下面这段代码？

{% codeblock lang:objc %}

@implementation Test

+(Test *)shareInstance
{
    static Test *test= nil;
    static dispatch_once_t token;

    dispatch_once(&token, ^{

        test = [[Test alloc]init];
    });

    return test;
}

-(instancetype)init
{
    self = [super init];

    if (self) {

        [Test shareInstance];
    }

    return self;
}


@end
{% endcodeblock %}
当我们创建Test([[Test alloc] init])对象时,会出现什么现象呢？死锁。是的，死锁，线程直接卡住了。为什么呢？
现在暂停程序，查看调用栈的状态如下图所示：
![avatar](https://raw.githubusercontent.com/glhfstar/glhfstar.github.io/hexo/source/images/dispatch_once.png)

发现程序是卡在dispatch_once_f中。研究一下dispatch_once_f的实现吧，如下代码所示，会发现一些有意思的东西。

{% codeblock lang:objc %}
void
dispatch_once_f(dispatch_once_t *val, void *ctxt, dispatch_function_t func)
{
	struct _dispatch_once_waiter_s * volatile *vval =
			(struct _dispatch_once_waiter_s**)val;
	struct _dispatch_once_waiter_s dow = { NULL, 0 };
	struct _dispatch_once_waiter_s *tail, *tmp;
	_dispatch_thread_semaphore_t sema;
	if (dispatch_atomic_cmpxchg(vval, NULL, &dow)) {
		dispatch_atomic_acquire_barrier();
		_dispatch_client_callout(ctxt, func);
		dispatch_atomic_maximally_synchronizing_barrier();
		//dispatch_atomic_release_barrier(); // assumed contained in above
		tmp = dispatch_atomic_xchg(vval, DISPATCH_ONCE_DONE);
		tail = &dow;
		while (tail != tmp) {
			while (!tmp->dow_next) {
				_dispatch_hardware_pause();
			}
			sema = tmp->dow_sema;
			tmp = (struct _dispatch_once_waiter_s*)tmp->dow_next;
			_dispatch_thread_semaphore_signal(sema);
		}
	} else {
		dow.dow_sema = _dispatch_get_thread_semaphore();
		for (;;) {
			tmp = *vval;
			if (tmp == DISPATCH_ONCE_DONE) {
				break;
			}
			dispatch_atomic_store_barrier();
			if (dispatch_atomic_cmpxchg(vval, tmp, &dow)) {
				dow.dow_next = tmp;
				_dispatch_thread_semaphore_wait(dow.dow_sema);
			}
		}
		_dispatch_put_thread_semaphore(dow.dow_sema);
	}
}
{% endcodeblock %}

简单描述一下吧。onceToken在第一次执行block之前，其值将由NULL变为指向第一个调用者的指针(&dow)。如果在block完成之前，有其它的调用者进来，则会把这些调用者放到一个waiter链表中(走else分支)，直到block执行完成。waiter链中的每个调用者都会等待一个信号量(dow.dow_sema)。在block执行完成后，除了将onceToken置为DISPATCH_ONCE_DONE外，还会去遍历waiter链中的所有waiter，抛出相应的信号量，以告知waiter们调用结束。

因此上面的死锁问题就好理解了。递归调用[[Test alloc] init]时，第二次调用作为一个waiter，在等待block完成，而block的完成依赖于[[Test alloc] init]的执行完成，这就成了一个死锁。

所以应该避免在dispatch_once做递归调用，不管是直接的还是间接的。


