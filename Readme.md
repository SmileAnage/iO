IO并发
===
### 常见模型分类
1. 循环服务器模型：循环接收客户端请求，处理请求．同一时刻只能处理一个请求，处理完毕后在处理下一个.
>优点：实现简单，占用资源少
>缺点：无法同时处理多个客户端请求
>适用情况：处理的任务可以很快完成，客户端无需长期占用服务端程序，处理完毕后在处理下一个

2. IO并发模型：处理IO多路复用，异步IO等技术，同时处理多个客户端IO请求
>优点：资源消耗少，能同时高效处理多个IO行为
>缺点：只能处理并发产生的IO事件，无法处理cpu计算
>适用情况：HTTP请求，网络传输等都是IO行为

3. 多进程/线程网络并发模型，每当一个客户端连接服务器，就创建一个新的进程/线程为客户端服务，客户端退出时再销毁该进程/线程
>优点：能同时满足多个客户端长期占有服务器需求，可以处理多种请求
>缺点：资源消耗较大
>适用情况：客户端同时连接量较少，需要处理行为较复杂情况

### IO分类
* IO分类：阻塞IO，非阻塞IO，IO多路复用，异步IO等
#### 阻塞IO
1. 定义：在执行IO操作时如果执行条件不满足则阻塞．阻塞IO是IO的默认形态
2. 效率：阻塞IO是效率很低的一种IO，但是由于逻辑简单所以是默认IO行为
3. 阻塞情况：因为某种执行条件没有满足造成的函数阻塞
* 处理IO的时间较长产生的阻塞状态(e.g. 网络传输，大文件读写)
#### 非阻塞IO
1. 定义：通过修改IO属性行为，使原本阻塞的IO变为非阻塞的状态
2. 设置套接字为非阻塞IO
>sockfd.setblocking(bool)
>功能：设置套接字为非阻塞IO
>参数：默认为True，表示套接字IO阻塞；设置为False则套接字IO变为非阻塞
3. 超时检测：设置一个最长阻塞时间，超过该时间后则不再阻塞等待
>sockfd.settimeout(sec)
>功能：设置套接字的超时时间
>参数：设置的时间

#### IO多路复用
1. 定义：同时监控多个IO事件，当那个IO事件准备就绪就执行哪个IO事件．以此形成可以同时处理多个IO的行为，避免一个IO阻塞造成其他IO均无法执行，提高了IO执行效率
2. 具体方案
>select方法：windows linux unix
>poll方法：linux unix
>epoll方法：linux

##### select方法
```python
rs, ws, xs = select(rlist, wlist, xlist[, timeout])
功能：监控IO事件，阻塞等待IO发生
参数：rlist 列表 存放关注的等待发生的IO事件
 			 wlist 列表 存放关注的要主动处理的IO事件
 			 xlist 列表 存放关注的出现异常要处理的IO
 			 timeout 超时时间
 返回值：rs 列表 rlist中准备就绪的IO
 				  ws 列表 wlist中准备就绪的IO
 				  xs 列表 xlist中准备就绪的IO
```
***参考代码select_server.py***
##### poll方法
```python
1. 创建套接字  p = select.poll()
2. 将套接字register
3. 创建查找字典，并维护
4. 循环监控IO发生
5. 处理发生的IO
events = p.poll()
功能：阻塞等待监控的IO事件发生
返回值：返回发生的IO
```
***参考代码poll_server.py***
##### epoll方法
1. 使用方法：基本与poll相同
>生成对象改为epoll()
>将所有事件类型改为EPOLL类型
2. epoll特点
3. epoll效率比select poll 要高
4. epoll监控IO数量比select要多
5. epoll的触发方式比poll要多(EPOLLET边缘触发)

### 协程技术
#### 基础概念
1. 定义：纤程，微线程．是允许在不同入口点不同位置暂停或开始的计算机程序，简单来说，协程就是可以暂停执行的函数
2. 协程原理：记录一个函数的上下文，协程调度切换时会将记录的上下文保存，在切换回来时进行调取，恢复原有的执行内容，以便从上次执行位置继续执行
3. 协会特点：
>优点
>>- 协程完成多任务占用计算资源很少
>>- 由于协程的多任务切换在应用层完成，因此切换开销少
>>- 协程未单线程程序，无需进行共享资源同步互斥处理

>缺点
>
>>- 协程的本质就是一个单线程，无法利用计算机多核资源

#### 第三方协程摸
1. greenlet模块
***参考代码greenlet.py***
2. gevent模块
***参考代码gevent.py***
3. monkey脚本
>- 作用：在gevent协程中，协程只能遇到gevent指定类型的阻塞才能跳转到其他协程，因此，我们希望将普通的IO阻塞行为转化为可以触发gevent协程跳转的阻塞，以提高执行效率
>- 转换方式：gevent提供了一个脚本程序monkey，可以修改底层解释IO阻塞的行为，将很多普通阻塞转换为gevent阻塞
>使用方法：
	```python
	from gevent import monkey  # 导入monkey
	monkey.patch_socket()  # 运行响应的脚本，例如转换socket中所有的阻塞
	monkey.patch_all()  # 如果将所有可转换的IO阻塞全部转换则运行all
	```

***参考代码***