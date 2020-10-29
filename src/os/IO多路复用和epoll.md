这个概念真的很难理解透彻，建议先看一下
* [TCPIP网络编程](https://book.douban.com/subject/25911735/)
* [epoll原理详解及epoll反应堆模型](https://blog.csdn.net/daaikuaichuan/article/details/83862311)
* [如果这篇文章说不清epoll的本质，那就过来掐死我吧!](https://zhuanlan.zhihu.com/p/64746509)

## select

服务端需要管理多个客户端连接，而recv只能监视单个socket，这种矛盾下，人们开始寻找监视多个socket的方法。select就可以帮我们管理多个socket连接。


```Python
import socket
import select

sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sk.bind(('0.0.0.0', 8000))
sk.setblocking(False)
sk.listen()

read_lst = [sk]
while True:
    r_lst, w_lst, x_lst = select.select(read_lst,[],[])
    for i in r_lst:
        if i is sk:  # 只有sk有accept方法，其他socket只能接收和发送数据
            conn, addr = i.accept()
            read_lst.append(conn)
        else:
            ret = i.recv(1024)
            if ret == b'':  # 如果发送的是空数据，就断开连接
                i.close()
                read_lst.remove(i)
                continue
            print(ret)
            i.send(b'HTTP/1.1 200 OK \n\n hello')
            i.close()
```

应用程序调用select方法的时候，底层会调用系统调用函数select，我们需要把注册的所有socket作为参数提供给内核，内核会去轮询这些socket。如果有socket变成可读，就会中断程序并返回。

缺点:

* 操作系统需要不断轮询socket，效率低
* 每次循环都要调用系统函数，并把所有的socket从用户空间拷贝到内核空间，效率低




## 水平触发和边沿触发
水平触发LT（Level Trigger）和边缘触发ET（Edge Trigger）

epoll有EPOLLLT和EPOLLET两种触发模式，LT是默认的模式，ET是“高速”模式。LT模式下，只要这个fd还有数据可读，每次 epoll_wait都会返回它的事件，提醒用户程序去操作，而在ET（边缘触发）模式中，它只会提示一次，直到下次再有数据流入之前都不会再提示了，无 论fd中是否还有数据可读。所以在ET模式下，read一个fd的时候一定要把它的buffer读光，也就是说一直读到read的返回值小于请求值，或者 遇到EAGAIN错误。

进一步理解：

在linux的IO多路复用中有水平触发，边缘触发两种模式，这两种模式的区别如下:

水平触发:如果文件描述符已经就绪可以非阻塞的执行IO操作了，此时会触发通知。允许在任意时刻重复检测IO的状态。select，poll就属于水平触发。

边缘触发:如果文件描述符自上次状态改变后有新的IO活动到来，此时会触发通知。在收到一个IO事件通知后要尽可能多的执行IO操作，因为如果在一次通知中没有执行完IO那么就需要等到下一次新的IO活动到来才能获取到就绪的描述符。信号驱动式IO就属于边缘触发。

写过单片机的人可以从另一方理解水平触发和边缘触发的区别：

水平触发:就是只有高电平(1)或低电平(0)时才触发通知，只要在这两种状态就能得到通知。上面提到的只要有数据可读(描述符就绪)那么水平触发的epoll就立即返回。

边缘触发:只有电平发生变化(高电平到低电平，或者低电平到高电平)的时候才触发通知。上面提到即使有数据可读，但是io状态没有变化epoll也不会立即返回。

epoll既可以采用水平触发，也可以采用边缘触发。

大家可能还不能完全了解这两种模式的区别，我们可以举例说明:一个管道收到了1kb的数据，epoll会立即返回，此时读了512字节数据，然后再次调用epoll。这时如果是水平触发的，epoll会立即返回，因为有数据准备好了。如果是边缘触发的不会立即返回，因为此时虽然有数据可读但是已经触发了一次通知，在这次通知到现在还没有新的数据到来，直到有新的数据到来epoll才会返回，此时老的数据和新的数据都可以读取到(当然是需要这次你尽可能的多读取)。所以当我们写epoll网络模型时，如果我们用水平触发不用担心数据有没有读完因为下次epoll返回时，没有读完的socket依然会被返回，但是要注意这种模式下的写事件，因为是水平触发，每次socket可写时epoll都会返回，当我们写的数据包过大时，一次写不完，要多次才能写完或者每次socket写都写一个很小的数据包时，每次写都会被epoll检测到，因此长期关注socket写事件会无故cpu消耗过大甚至导致cpu跑满，所以在水平触发模式下我们一般不关注socket可写事件而是通过调用socket write或者send api函数来写socket，说到这我们可以看到这种模式在效率上是没有边缘触发高的，因为每个socket读或者写可能被返回两次甚至多次，所以有时候我们也会用到边缘触发但是这种模式下在读数据的时候一定要注意，因为如果一次可写事件我们没有把数据读完，如果没有读完，在socket没有新的数据可读时epoll就不回返回了，只有在新的数据到来时，我们才能读取到上次没有读完的数据。
