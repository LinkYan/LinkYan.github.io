---
layout: post
title: "谬论：C最高效"
tags: 
- C
- PL
- "翻译"
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---

作者：Mark Chu-Carroll (aka MarkCC)
翻译：ssword
原文：<a href="http://scienceblogs.com/goodmath/2006/11/the_c_is_efficient_language_fa.php">http://scienceblogs.com/goodmath/2006/11/the_c_is_efficient_language_fa.php</a>

<hr />

昨天偶然看到一篇关于程序设计语言的文章，直击我G点，忍不住前来吐槽。这篇文章来自greythumb.org，叫做《程序员的呼喊：C\C++该有什么》。

无非也就是“C\C++是追求高效的首选”之类的老生常谈。他们错了。它们不是。C\C++接近硬件是为了直接控制堆栈、修改寄存器等等，而非为了高效。在科研编程或者数值计算的应用上，它们差劲得很。

贴一段让我崩溃的文字：

<blockquote>首先，那些担忧纯属多余。C\C++永远不会消失。为什么？因为有无数的程序现在是、永远都是吃CPU的。对这些领域的编程而言，根本没有快过C\C++的语言。将来会不会出现这种语言？我非常怀疑。

我说的这些领域就是：科学计算、游戏/物理特效、光线跟踪、实时3D图像、音频处理、编译器、高速路由、进化计算(我的最爱:)，还有高级语言运行时---毫无疑问。再就是像操作系统、硬件驱动之类“接近底层”，需要很多交互、甚至内嵌汇编的程序。C就是简单版的汇编，这就是为何C总是作为此类程序的首选。

对这些领域而言，在语言及架构层面的过早优化是可以接受、有时甚至是必须的。我敢打赌，在五十年后，这些领域的一部分依然会是C\C++或者相似语言的天下。对于同样一个基于进化计算指令集的实现，C要整整快过java两倍。由此你可以看出C是多么的快。</blockquote>

问题在这里：C\C++在数值计算上的性能相当扯淡。它们不是最快的，而且绝非偶然。实际上，受底层实现的一些限制，使得C\C++根本不可能表现得很高效。这便是为什么到今天依然有Fortan应用于在高精度科研项目，而这些应用往往需要榨干机器的每一滴性能------如流体动力学模拟。[<a href="#q1">1</a>]

程序要高效离不开编译器优化，现代架构的编译器可以达到人类优化汇编代码的极限。有时交换两条无关指令的顺序就可以得到一个出人意料的性能提升，而机器所做的优化，很多都是人类难以企及的。[<a href="#q2">2</a>]

因此对于现代的开发而言，程序的高效绝非只凭程序员一人之力。程序员需要做的，是仔细选择合适的算法-----这活机器做不了；机器需要做的，是仔细地调整指令、约束流水线、内存延时等等。二者合作才会有高效的程序。二者的工作又是相互影响的：程序员应该用机器能够理解的代码来描述算法，以方便机器进行优化。

这就是C\C++失败的地方。C\C++在语义上过度依赖指针，导致不受约束的指针几乎无处不在。在C\C++中，并没有真正意义上的数组------它们只是指针，下标只是指针运算的简写形式（C\C++里的x[n]与*(x+n)是完全一样的）。

过度依赖指针就意味着，C\C++的编译器会很难辨认两个东西是否独立。由此产生的问题被称作“重名探测”（alias detection），也就是找出可能指向同一个位置的两个变量。若存在不受约束的指针，别名探测几乎就无法实现。举个例子：

<pre lang="c">
for (int i=0; i < 20000) {
   for (int j=0; j < 20000) {
      x[i][j] = y[i-2][j+1] * y[i+1][j-2];
   }
}
</pre>

看下这个循环。它可以并行化或向量化[<a href="#q3">3</a>]，但前提必须是x与y没有重合、完全无关，这在C\C++中这根没办法得到保证。Fortran-77就没问题，你可以轻而易举地检查它俩是否不同。若是Fortran-98，你还可以检查出它们是否为指针，若有需要，程序就可以弄清楚它们之间是否有重复。在C\C++下，这就做不到（Fortran也不是最好的------一门来自Lawrence Livermore labs 的实验语言Sisal，在特定代码上要强过Fortran 20%）。

这个例子是拿并行说事，但“重名”带来的问题可不只并行这么简单，说并行只是因为比较好解释。“重名”带来的问题直接影响了C\C++代码的编写。

再举个具体的例子吧，我就不吐槽了。六年前，我在一个项目里需要实现一个相当复杂的算法来计算两个数组的“最长公共串”（longest comon sequence， LCS）。计算LCS的标准算法被称作“动态规划”，是O*(n^3) 的时间，O(n^2) 的空间。搞计算生物学的人们也有个相似的算法，时间与它相同，不过空间平均起来只有O(n)。

我不知道该用哪门语言，就做了个实验。我用C，C++，ocaml，java和python分别实现了一遍LCS算法，来比对代码的复杂度
以及运行效率。2000个元素的数组来做实验数据，所得的测试结果如下：

<ul>
	<li>C: 0.8 seconds.</li>
	<li>C++: 2.3 seconds.</li>
	<li>OCaml: 0.6 seconds interpreted, 0.3 seconds fully compiled.</li>
	<li>Java: 1 minute 20 seconds.</li>
	<li>Python: over 5 minutes.</li>
</ul>

一年以后，我用新版本的JIT再次测试，java的时间减少到了0.7秒，外加1秒的JVM初始化。（C\C++以及ocaml的初始化时间几乎可以忽略不计）

Objective-Caml字节码解释器的执行效率比经过仔细优化的C程序还要高！为什么？因为Ocaml编译器可以辨别出两个数组的无关性------这一来就不必担心循环中的一个迭代会影响到另一个迭代中的值。C编译器可做的优化就要少得多了，因为它无法辨认这些优化是否会影响到程序的正常运行。

不只没有赋值的函数式语言，不那么高效的高级语言在一些方面的性能也要比C\C++出色。CMU Common Lisp在数值计算上就比C\C++强。几年前有个论文写了这个：在一台Sun Sparc工作站上，如果你带上类型声明，用vector（Lisp的数组）和赋值来实现的科学计算/数值计算Lisp代码，要比经过Solaris C或gcc最大优化的C算法实现更加高效。


<hr />

译者注：
<a name="q1"></a>[1] 从一开始Fortran就被设计成可以进行高度优化的语言…Fortran的设计者关注的是科学计算中代码的运行时性能…在这样想法的指导下，只有当Fortran编译器生成的代码性能是有经验的程序员手写的并经过性能调整的汇编代码性能的两倍时，Fortran语言才会被用户接受。 ------J.Backus《the history of Fortran I , II, III》
C最初被设计成有类型的汇编语言，重点针对可用性，而非优化。 ------《现代体系结构的优化编译器》 p147

<a name="q2"></a>[2] 为此就需要对指令重新排列，或成为调度，以使相互冲突或依赖的指令在时间上分离，这种需要也就是为现代处理器做编译器特别困难的主要原因之一。 -------《程序设计语言：实践之路》 p207

<a name="q3"></a>[3] 向量化就是将一个循环中的迭代并行执行。
例如：
For I  in  1 to 100
    A[i]=i*2
End
其中的a[1]和a[2]可以同时执行，前提就是循环中的每个迭代不会相互影响。古代有种机器貌似叫做向量计算机，里面的循环都是并行执行的，就是比较早的并行计算的大型机了。

<hr />

ps:
作者原文上的评论是相当的长，应了那句“凡语言贴，必火”。 毕竟没有银弹，要了解一个东西也应该了解它的弱点，不然就容易为你爱的东西所束缚。

不过作者貌似回避了一点，那就是所谓“科学计算、游戏/物理特效、光线跟踪、实时3D图像、音频处理、编译器、高速路由、进化计算，还有高级语言运行时”等等，在实际中还是C用的多。“C是接近硬件，而不是为了高效”，不过游戏、音频等方面还有个东西叫做硬件加速哇。对于C在优化上的限制，可以参考《现代体系结构的优化编译器》 418页，上面举的例子可能要更好，说明也更详细，也有一整章C编译器在优化上的解决方法。

pps: 很明显，第一段翻译的很不妥..求正解