- [NCCL 学习笔记（一）：MPI 基本知识](#nccl-学习笔记一mpi-基本知识)
  - [MPI简介](#mpi简介)
  - [P2P 通信](#p2p-通信)
    - [MPI Send and Receive](#mpi-send-and-receive)
  - [集合通信](#集合通信)
    - [MPI\_Barrier](#mpi_barrier)
    - [MPI\_Bcast](#mpi_bcast)
    - [MPI\_Scatter](#mpi_scatter)
    - [MPI\_Gather](#mpi_gather)
    - [MPI\_Allgather](#mpi_allgather)
    - [MPI\_Reduce](#mpi_reduce)
    - [MPI\_Allreduce](#mpi_allreduce)

# NCCL 学习笔记（一）：MPI 基本知识

学习NCCL之间需要先了解一些通信的基本 MPI 原语，主要了解其中的概念即可。

## MPI简介

MPI是一个跨语言的通讯协议，支持高效方便的点对点、广播和组播。它提供了应用程序接口，包括协议和和语义说明，他们指明其如何在各种实现中发挥其特性。

MPI(Message Passing Interface) 基本概念：

- 通讯器（communicator）: 通讯器定义了一组能够互相发消息的进程。
- 秩（rank）: 在这组进程中，每个进程会被分配一个序号，称作 rank ，进程间显性地通过指定秩来进行通信。
- 标签（tag）: 通信的基础建立在不同进程间发送和接收操作。一个进程可以通过指定另一个进程的秩以及一个独一无二的消息来发送消息给另一个进程。接受者可以发送一个接收特定标签标记的消息的请求（或者也可以完全不管标签，接收任何消息），然后依次处理接收到的数据。
- P2P 通信：涉及一个发送者以及一个接受者的通信。
- 集合通信（collective communications）：多个进程间通信。MPI 有专门的接口来帮我们处理相关的通信。

## P2P 通信

### MPI Send and Receive

一个进程向另一个进程的点对点通信，主要是两个接口，Send 和 Receive，字面意思。

```C
MPI_Send(
    void* data,
    int count,
    MPI_Datatype datatype,
    int destination,
    int tag,
    MPI_Comm communicator)

MPI_Recv(
    void* data,
    int count,
    MPI_Datatype datatype,
    int source,
    int tag,
    MPI_Comm communicator,
    MPI_Status* status)
```

## 集合通信

### MPI_Barrier

同步，所有进程同步到同一时刻。

![image_0](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/45bab061e6d3da3bbbdd529361df6ac15818a8c1adb85e6504167591ed553cc6.png)  

### MPI_Bcast

广播 (broadcast) 是标准的集体通信技术之一。一个广播发生的时候，一个进程会把同样一份数据传递给一个 communicator 里的所有其他进程。广播的主要用途之一是把用户输入传递给一个分布式程序，或者把一些配置参数传递给所有的进程。

![image_1](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/8259d93b344c2cb4eaaab7f737ce439f3c5dbf0412886a519767fc5b90a3b212.png)  

```c
MPI_Bcast(
    void* data,
    int count,
    MPI_Datatype datatype,
    int root,
    MPI_Comm communicator)
```

### MPI_Scatter

MPI_Scatter 是一个跟 MPI_Bcast 类似的集体通信机制，MPI_Scatter 的操作会设计一个指定的根进程，根进程会将数据发送到 communicator 里面的所有进程。MPI_Bcast 和 MPI_Scatter 的主要区别很小但是很重要。MPI_Bcast 给每个进程发送的是同样的数据，然而 MPI_Scatter 给每个进程发送的是一个数组的一部分数据。下图进一步展示了这个区别。

![image_2](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/88bb4b88265886e7f249b7585c70aec5d75bfa435f991d51eccb33d8078045f1.png)  

```c
MPI_Scatter(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)
```

### MPI_Gather

MPI_Gather 跟 MPI_Scatter 是相反的。MPI_Gather 从好多进程里面收集数据到一个进程上面而不是从一个进程分发数据到多个进程。这个机制对很多平行算法很有用，比如并行的排序和搜索。

![image_4](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/1bc5dac372f0604aac3b12a31388c0636abf62584575f629e0946a41af06bdf4.png)  

```c
MPI_Gather(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)
```

### MPI_Allgather

前面的 OP 都是用来操作多对一或者一对多通信模式的MPI方法，也就是说多个进程要么向一个进程发送数据，要么从一个进程接收数据。很多时候发送多个元素到多个进程也很有用（也就是多对多通信模式）。MPI_Allgather 就是这个作用。

对于分发在所有进程上的一组数据来说，MPI_Allgather 会收集所有数据到所有进程上。从最基础的角度来看，MPI_Allgather相当于一个MPI_Gather 操作之后跟着一个 MPI_Bcast操作。下面的示意图显示了 MPI_Allgather 调用之后数据是如何分布的。

![image_5](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/31a7bb146cf14603a39a9ef87928cde7959f2ce4dd02970b57fc026c625b5c32.png)  

```c
MPI_Allgather(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    MPI_Comm communicator)
```

### MPI_Reduce

与 MPI_Gather 类似，MPI_Reduce 在每个进程上获取一个输入元素数组，并将输出元素数组返回给根进程。输出元素包含减少的结果。 简而言之就是一起做一个计算，MPI_Reduce 的原型如下所示：

![image_6](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/ba4a803876e395536fc275ef22ba5271c8b7f2e484dd6b04bda7bf5b4814ae72.png)  


```C
MPI_Reduce(
    void* send_data,
    void* recv_data,
    int count,
    MPI_Datatype datatype,
    MPI_Op op,
    int root,
    MPI_Comm communicator)
```

send_data 参数是每个进程都希望归约的 datatype 类型元素的数组。 recv_data 仅与具有 root 秩的进程相关。 recv_data 数组包含归约的结果，大小为 sizeof（datatype）* count。 op 参数是您希望应用于数据的操作。 MPI 包含一组可以使用的常见归约运算。 尽管可以定义自定义归约操作，但这超出了本教程的范围。 MPI 定义的归约操作包括：

- MPI_MAX - 返回最大元素。
- MPI_MIN - 返回最小元素。
- MPI_SUM - 对元素求和。
- MPI_PROD - 将所有元素相乘。
- MPI_LAND - 对元素执行逻辑与运算。
- MPI_LOR - 对元素执行逻辑或运算。
- MPI_BAND - 对元素的各个位按位与执行。
- MPI_BOR - 对元素的位执行按位或运算。
- MPI_MAXLOC - 返回最大值和所在的进程的秩。
- MPI_MINLOC - 返回最小值和所在的进程的秩。

### MPI_Allreduce

许多并行程序中，需要在所有进程而不是仅仅在根进程中访问归约的结果。 以与 MPI_Gather 相似的补充方式，MPI_Allreduce 将归约值并将结果分配给所有进程。 函数原型如下：

```c
MPI_Allreduce(
    void* send_data,
    void* recv_data,
    int count,
    MPI_Datatype datatype,
    MPI_Op op,
    MPI_Comm communicator)
```

![image_7](https://cdn.jsdelivr.net/gh/ZqtCtios/Image@master/sysml_master/ee08030329a7760826ad534f2983426e701aebb0c4392e529136d5a293df1ff0.png)


MPI_Allreduce 与 MPI_Reduce 相同，不同之处在于它不需要根进程 ID（因为结果分配给所有进程）。MPI_Allreduce 等效于先执行 MPI_Reduce，然后执行 MPI_Bcast。

参考文档：

- [https://mpitutorial.com/](https://mpitutorial.com/)
