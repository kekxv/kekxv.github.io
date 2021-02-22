---
layout: post title: "java PipedInputStream 管道吐槽"
tagline: "PipedInputStream 管道吐槽"
categories: java author: "kekxv"
---

最近业务上需要使用到管道，做数据缓存和通讯，使用了`PipedInputStream`、`PipedOutputStream`。记录一下一个隐藏的坑！

在项目初期，所有调用均为正常，数据也能正常写入读取管道，遂提交代码，并且交由他人使用。

然，增加功能以及业务调用之后，无法读取到数据？？？？并且在进行**管道写**的时候报错`Java io ioexception read end dead`？？？？？？？？？？

什么情况？在查看源代码并且在源码中搜索关键字`read end dead`后找到：

```java
package java.io;

public class PipedInputStream extends InputStream {
    // 其他代码 ...
    private void checkStateForReceive() throws IOException {
        if (!connected) {
            throw new IOException("Pipe not connected");
        } else if (closedByWriter || closedByReader) {
            throw new IOException("Pipe closed");
        } else if (readSide != null && !readSide.isAlive()) {
            throw new IOException("Read end dead");
        }
    }
    // 其他代码 ...
}
```

`readSide`定义为`Thread readSide;`？？？

这干嘛的？

```java
package java.io;

public class PipedInputStream extends InputStream {
    // 其他代码 ...
    public synchronized int read()  throws IOException {
        if (!connected) {
            throw new IOException("Pipe not connected");
        } else if (closedByReader) {
            throw new IOException("Pipe closed");
        } else if (writeSide != null && !writeSide.isAlive()
                   && !closedByWriter && (in < 0)) {
            throw new IOException("Write end dead");
        }

        readSide = Thread.currentThread();
        // 其他代码 ...
    }
}
```

`readSide = Thread.currentThread();`？？？每次进行读的时候获取当前读线程的线程？？有什么用？

![readSide use](/assets/imgs/20210221-readSide_use.png)

开起来似乎，没有任何用处啊？！

这内部为什么获取读的线程之后除了判断线程死活之外没有任何限制？

**而且问题是，读的线程死活关你管道写什么事？** 为什么写入管道数据的时候，需要判断读的线程是否存活？

如果要保证读写线程为一对的，但为什么每次读的时候又去更新`readSide = Thread.currentThread();`？难道不应该是：

```java
if(readSide==null)readSide=Thread.currentThread();
assert(readSide==Thread.currentThread();)
```

每次调用读的时候去更新的话，那还有什么意义？

而且，最为关键的是，只有在调用 `read` 读的时候才会更新，并且没有办法使用其他方式更新(哦，好像可以用反射)；这也就导致，我没有办法在写之前告诉他，我`读`还活着！！！

最后，为了解决这个问题，只能引入一个线程池的方式，使用线程池统一去读取；类似以下例子：

```java
class main {
    // 创建固定大小的线程池:
    private static ExecutorService executor = Executors.newFixedThreadPool(1);
    private static PipedInputStream pipedInputStream = new PipedInputStream(10 * 1024);

    public int read() {
        return executor.submit(() -> pipedInputStream.read()).get();
    }
}
```

备注：省略了其他的多余代码。

