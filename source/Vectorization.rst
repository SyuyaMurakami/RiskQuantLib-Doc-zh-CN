使用矢量化
====================

.. toctree::
   :maxdepth: 4


本章节主要涉及 ``矢量化编程`` 。这个词对数据分析师来说并不陌生，简单来说，它即是 ``尽可能使用矩阵进行运算`` 的代指。因为现代CPU基本都是多位运算的，矢量化的数据结构可以充分利用这一特点，达到成数倍地提升运算速度的效果。

使用矢量化的注意事项
^^^^^^^^^^^^^^^^^^^^^^^^^^^

**函数的纯数学性**：
---------------------------

矢量化变成和多进程编程一样，是一把双刃剑。几乎所有程序员都会认同，能够矢量化运算的函数仅仅是函数大家族中非常小的一部分。因为要使用底层的并行指令，矢量化技术对于数据类型进行了严格的限制，确保只有正确的数据类型才能被处理。

**体现在用户端，不准确地说，能够被矢量化运算的函数必须是纯数学函数。**

什么是纯数学函数呢。这里有必要对计算机上面的函数和数学意义上的函数做区分。计算机上的函数是通过关键字定义的，实现某个特殊功能的代码块。其中往往涉及逻辑判断，循环，输入输出等计算机操作。而数学函数是变量对变量的映射，在给定相同输入的情况下，必须得到相同的输出。

**计算机函数往往是状态依赖的，比如读取文件的函数** ``pandas.read_csv`` **，它接收一个文件路径，返回文件的内容。如果你更改了文件的内容，返回的内容也会改变，这是理所当然的。但需要注意的是，这个函数的接收的变量值没有任何改变，依然是文件的路径。之所以在变量值不变的情况下会返回不同的值，是因为它依赖于文件的状态。**

**数学函数是纯的，无状态依赖的。比如** ``numpy.exp`` **这个函数，在接收自变量为** ``1`` **时，一定会返回一个自然对数** ``e`` **的值，无论什么时候，在什么场合运行这个函数都是这样的。**

因此，在使用这个功能时，需要额外注意确保你的函数不依赖于状态，是一个纯的数学函数。另外，在矢量化过程中需要涉及数据的包装和解包，这会消耗一定的计算资源。当然，对于可以被矢量化运算的函数来说，这些额外的资源消耗和矢量化提升的速度相比往往是可以忽略的。

基于纯数学函数的要求， **python的很多逻辑关键字是不可以被使用的** 。能够被矢量化的函数中不可出现 ``if`` , ``for`` , ``is`` , ``in`` 等关键字。如果要进行逻辑运算，你必须熟悉如何使用逻辑矩阵，我们会在下面进行演示。

如何使用矢量化
^^^^^^^^^^^^^^^^^^^^^^^^^^^

RiskQuantLib中使用多进程非常简单，和使用 ``execFunc`` 几乎一模一样。如果你还不熟悉 ``execFunc`` 的使用，你可以参考 :doc:`Instrument_List` 中的 **遍历** 小节。

简单来说，通过 ``vecFunc`` 可以对各个模板类的实例进行并行运算。使用：
::

   instrumentListA.vecFunc('someFunc',someParameter)

几乎完全相同于：
::

   instrumentListA.execFunc('someFunc',someParameter)

他们的效果基本相当于：
::

   for instrumentA in instrumentListA:
       instrumentA.someFunc(someParameter)

区别在于， ``execFunc`` 是顺序运算的，排在前面的列表元素会先执行 ``someFunc`` ，而排在后面的函数会后执行。 ``vecFunc`` 是矢量并行执行的，非常不准确地说，所有元素几乎在同一时间被调用 ``someFunc`` ，这使得你无法让下一个元素的运算依赖于上一个元素运算的结果，就像 ``fold`` 函数那样。

幸运的是， ``vecFunc`` 中调用的函数可以在内部使用几乎所有的 ``numpy`` 函数。因为 ``numpy`` 函数几乎都是纯数学的。实际上， ``RiskQuantLib`` 的底层就是使用 ``numpy`` 来进行矢量化的。

另外， ``vecFunc`` 调用的函数可以对数据图进行修改。赋值操作依然可以正常运行。但是赋值操作是不能被矢量化的，它只是不会造成矢量化的异常，从而兼容矢量化。赋值操作往往会拖累矢量化的速度。

**你并不需要对函数进行特别的优化来使用** ``vecFunc`` ，**函数内部依然可以对属性进行修改。 你只需要确保被调用的函数是纯数学的，没有逻辑判断关键字。**

一个良好的实例是：
::

   instrumentListA.vecFunc('someFuncX',someParameterX)
   instrumentListA.vecFunc('someFuncY',someParameterY)
   instrumentListA.vecFunc('someFuncZ',someParameterZ)

这相当于：
::

   instrumentListA.execFunc('someFuncX',someParameterX)
   instrumentListA.execFunc('someFuncY',someParameterY)
   instrumentListA.execFunc('someFuncZ',someParameterZ)

如果你这样定义上面例子里的三个函数：
::

   def someFuncX(self):
       self.codeX = self.code + 1
       return self.codeX
   def someFuncY(self):
       self.codeY = self.codeX + 2
       return self.codeY
   def someFuncZ(self):
       self.codeZ = self.codeY + 3
       return self.codeZ

矢量化版本和非多进程版本都是可以正常运行的。矢量化版本的速度往往是非矢量化版本的3-5倍。注意到依次运行的矢量化函数，后一个运行的函数是可以正常访问前一个运行的函数生成的数据图节点变量的。

另外，注意到我们这里的三个函数都是纯数学的。而类似下面的函数是不可以被矢量化的：
::

   def wrongFuncX(self):
       self.codeX = self.code if self.code >= 1 else 3
       return self.codeX

如果要使用逻辑判断，你必须使用逻辑矩阵。要实现上面函数的功能，同时使用矢量化运算，你应该将函数改成：
::

   def rightFuncX(self):
       logicMatrix = (self.code >= 1) 
       self.codeX = self.code * logicMatrix + 3 * (1 - logicMatrix)
       return self.codeX

**可以被矢量化的函数，其中涉及的变量值也可以是高维度的。** 比如上面的代码中的 ``self.code`` 属性，我们默认它是个数值型的变量。但实际上，它还可以是更高维度的变量，比如数值型的矩阵，这并不影响矢量化。

**高维度矢量化**：
---------------------------

**可以使用常数作为被矢量化函数的传入参数，但是传入常数矢量或者常数序列时，你需要确保传入的矢量是已经经过矢量化的，因为只有节点的属性会被自动矢量化，常数不会被自动矢量化。** 我们举例说明这一点：
::

   def someFuncWithPara(self, para):
       self.codeX = self.code + para
       return self.codeX

对于这个函数，如果想要将其矢量化运行，默认 ``self.code`` , ``para`` , ``self.codeX`` 都是数值型变量，比如 ``float`` 类型的变量。但实际上， ``self.code`` , ``self.codeX`` 都可以是数值型的矢量或者矩阵，比如元素是 ``float`` 型的 ``numpy.ndarray`` 型变量。因为所有数据图节点的属性都会被 ``RiskQuantLib`` 自动矢量化。

但问题出在 ``para`` 变量上。如果 ``para`` 是一个 ``float`` 类型的变量，那么当 ``self.code`` 是矢量的时候，运算不会出现问题，因为常数会被自动广播到矢量的每个元素上。如果 ``para`` 也是一个元素是 ``float`` 型的 ``numpy.ndarray`` ，会发生什么？

由于 ``RiskQuantLib`` 无法判断传入的参数到底是否是已经经过矢量化处理的，所以 ``RiskQuantLib`` 会使用 ``para`` 传入时的变量类型直接与已经矢量化后的属性变量进行运算。也就是说， ``para`` 不会被进行额外的处理。所以如果 ``para`` 也是一个元素是 ``float`` 型的 ``numpy.ndarray`` ，那么很大可能会发生报错。

如果你想要程序正常运行，那么你需要确保在矢量化调用 ``someFuncWithPara`` 前，先手动对 ``para`` 常数进行矢量化，即 ``para`` 应该是一个可迭代对象，且长度应该等于当前 ``RiskQuantLib`` 的模板列表的长度。 ``para`` 的第一个元素应该是传给 ``RiskQuantLib`` 列表的第一个元素的参数， ``para`` 的第二个元素应该是传给 ``RiskQuantLib`` 列表的第二个元素的参数，以此类推。

**好在这种情况非常少见，如果你需要在每个元素上使用不同的参数，为什么不将其设定为一个属性，而是需要从外部传入变量呢？考虑更改数据图的链接结构，使用**  ``match`` ， ``link`` ， ``join`` ， ``connect`` 来更改数据图，才是更合理的做法。

如何结合多进程和矢量化
^^^^^^^^^^^^^^^^^^^^^^^^^^^

作为能够提升速度的两种方式，多进程和矢量化给予了 ``RiskQuantLib`` 非常灵活和强大的数据处理能力。通过仔细的调试和设计，使用 ``RiskQuantLib`` 进行编写的程序的速度往往可以达到使用 ``pandas`` 和 ``numpy`` 进行编写的矩阵话程序的速度的50%。在涉及超高数据吞吐，网络交互等的情况下， ``RiskQuantLib`` 的速度甚至可以超过矩阵化程序的速度。（当然，很少有程序员用 ``pandas`` 等数值库来处理高吞吐的数据。）如果考虑到程序的反复修改，比如数据探索等领域， ``RiskQuantLib`` 所带来的代码编写的简易化，后期维护的低成本，很高的代码重用率等优点会变得更加突出。

通常， ``numpy`` 会自动进行CPU的调度，但如果你需要，你可以同时使用多进程和矢量化：
::

   result = instrumentListA.groupByFunc(lambda x:int(numpy.random.uniform(0,4))).paraFunc('vecFunc','someFuncX',someParameterX).paraRun()

上面的代码含义为：将 ``instrumentListA`` 随机分成四组，每组新开一个进程，每个进程进行矢量化运算，调用 ``someFuncX`` 函数，然后收集所有的结果，存储在 ``result`` 变量中。注意到因为使用了多进程，多进程是管道化的，对数据图的修改不会被保留，所以 ``someFuncX`` 必须有返回值，而且必须通过一个变量接收它，以便于后续的处理。

上面的代码展现了 ``RiskQuantLib`` 令人惊叹的抽象能力和灵活度。在不需要对数据处理逻辑函数以及数据图进行任何修改的情况下，你可以仅仅通过一行代码来更改程序的运行方式。其中采用的链式调用让程序变得更加可读。在 ``paraFunc`` 中我们使用 ``隧穿调用`` ，这在前面的章节中我们提到过，对于多层数据图，我们可以在 ``paraFunc`` 或者 ``execFunc`` 或者 ``vecFunc`` 中嵌套任意数量（不超过图的深度）的函数调用，就像是通过隧道，经过上层的图节点，直达下一层的图节点，然后调用下一层的函数。最后我们使用 ``paraRun`` 来触发多进程。

**遗憾的是，如果** ``someFuncX`` **的定义是我们在上文中示例的那样，那么上面的代码在大部分计算机上的运行速度应该不如直接进行矢量化：**
::

   result = instrumentListA.vecFunc('someFuncX',someParameterX)

同时使用多进程和矢量化的速度甚至不如直接遍历：
::

   result = instrumentListA.execFunc('someFuncX',someParameterX)

原因很简单，我们在上文定义的 ``someFuncX`` 太过于简单，使用多进程的开销已经远远大于收益。所以在这种情况下，使用多进程不是个好主意。通常，  ``someFuncX`` 函数中的运算次数应该超过十万次，才能确保多进程的使用是会带来效率提升的。这就是我们说的，使用多进程和矢量化前，需要先思考程序的运行环境，确保这样的开销是值得的。