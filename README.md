# Project4：Parallel Computing in eBPF and WASM

Taichi 是一门开源的、嵌入在 Python 中的并行编程语言，它语法简单，上手容易，运行高效，可以大大简化高性能图形学、数值计算、人工智能等应用的开发，并且也支持脱离 Python 运行，将高性能计算代码导出并部署到 PC、移动端、浏览器等不同设备上。

Taichi 可以应用于多种场景，例如Python 加速，数值仿真、3d渲染、机器人视觉等。

Taichi 还可以被用来进行非结构网格的并行，上面我们看到的所有网格都是结构化的（Grid），而在很多图形学程序中，我们也需要使用非结构的网格，通常我们叫它 Mesh。非结构网格会定义诸如节点-节点，边-节点，面-节点的连接关系，通过一种显式的方式去构造网格，而 Taichi 也可以在这样的网格上进行并行加速。

网格生成作为数值仿真的起点，是CAE领域根技术/基础核心技术之一，也是计算可信性的前提保障，但却是CAX智能制造链条中最薄弱环节之一。全自动、多尺度、大规模以及多场耦合对网格生成算法的研究提出了更高的要求，高可靠、高效率、高质量的网格生成是CAE领域永恒主题。我国缺少工业级前处理网格生成产品，而国外对我有功能规模限制。这很大程度制约了结构、电磁、气动、流固与多物理场耦合、直至EDA等研究领域的发展。

Mesh Taichi 这篇文章，就使用了 Taichi 来完成类似的网格仿真计算，不过仅限于单机执行。

Taichi 社区的用户也常有类似的需求：不少用户在研究中需要处理非常大规模的网格（如流体和粒子仿真），大到数据量远远超过一张显卡的显存。有没有一种简单的方法，可以让 Taichi 程序分布式地跑在多张 GPU 卡甚至多台 GPU 机器上，从而高效地解算大规模问题呢？

Taichi 有过一些同时实现高性能并行+分布式计算的尝试，但是是使用 MPI 完成的，MPI（MPI是一个标准，有不同的具体实现，比如MPICH等）是多主机联网协作进行并行计算的工具，MPI 的性能很不错，但是它的编程模型复杂，用户必须通过显式地发送和接收消息来实现处理机间的数据交换：

需要分析及划分应用程序问题，并将问题映射到分布式进程集合；
需要解决通信延迟大和负载不平衡两个主要问题；
调试MPI程序麻烦；
MPI程序可靠性差，一个进程出问题，整个程序将错误；

另外 Taichi 的后端也可以生成 WASM 代码。因此有没有可能能尝试把 Taichi 的 WASM 编译后端，放在 WAMR 这样的服务器端的无服务器 WASM 运行时里面，进行调度和分布式并行加速？主要有以下几个目标：

1. 在浏览器外的 WASM 运行时中运行 Taichi 的 JIT 编译产物；
2. 尝试利用基于 WASM 或者 eBPF 的 FaaS 进行分布式并行，以加速 Taichi 程序的运行；
3. 比较 Taichi 在多种不同应用场景或工作负载下 ，使用 WASM 或其他方式并行加速所带来的可能收益效果；

参考资料：

<https://docs.taichi-lang.org/docs/compilation> 介绍了taichi 的编译流程
Taichi x MPI4Py <https://zhuanlan.zhihu.com/p/581896682>
<https://github.com/AmesingFlank/taichi.js> 介绍了 taichi 在浏览器中，借助 Js 和 wasm 运行的相关工作；
