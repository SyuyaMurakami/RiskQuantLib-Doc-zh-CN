使用多进程
====================

.. toctree::
   :maxdepth: 4


本章节主要涉及 ``多进程编程`` 。对于数据分析师而言，大家似乎已经接受了python是一种很慢的语言。它的优势在于丰富的应用生态，而速度则是一个巨大的缺点。RiskQuantLib提供了一种简单的方式来进行初级的多进程并行运行。对于那些计算密集型的任务来说，多进程运行可以将速度提升大概一个量级。

使用多进程的注意事项
^^^^^^^^^^^^^^^^^^^^^^^^^^^

多进程编程是一把双刃剑，几乎程序员们都会认同，没有一种简单的方式可以非常通用地对所有类型的程序进行多进程编程。良好的多进程程序往往需要经过细心的调试和优化，并且每个项目都会略有区别。另外，开启多进程的开销必须大于多进程带来的收益，这也是非常关键的一点。

对于RiskQuantLib这样主要面向数据处理的脚手架来说，多进程编程的要求被降低了，图数据存储使得数据结构也有章可循。但是比较棘手的是面向对象的编程风格并不非常适用于多进程，将整个数据图都序列化并且传输到另一个进程是非常消耗资源的。

所以非常有必要提醒需要用到此功能的用户，在开启多进程之前，需要考虑一下你要调用的函数是否足够的计算密集，需要考虑一下内存的限制。

如何使用多进程
^^^^^^^^^^^^^^^^^^^^^^^^^^^

RiskQuantLib中使用多进程非常简单，和使用 ``execFunc`` 几乎一模一样。如果你还不熟悉 ``execFunc`` 的使用，你可以参考 :doc:`Instrument_List` 中的 **遍历** 小节。

简单来说，通过 ``paraFunc`` 可以对各个模板类的实例进行并行运算。使用：
::

   instrumentListA.paraFunc('someFunc',someParameter)

几乎完全相同于：
::

   instrumentListA.execFunc('someFunc',someParameter)

他们的效果基本相当于：
::

   for instrumentA in instrumentListA:
       instrumentA.someFunc(someParameter)

区别在于， ``execFunc`` 是立即执行的，而且 ``execFunc`` 调用的函数可以对数据图进行直接修改。而 ``paraFunc`` 是延迟执行的，你必须在若干次使用 ``paraFunc`` 之后再使用 ``paraRun`` 来触发多进程。否则多进程程序并不会被执行。

另外， ``paraFunc`` 调用的函数不可以对数据图进行直接的修改，如果在这些函数内部对类属性进行修改，这些修改不会被保留，（简单说的话，实际上任何对原数据图的影响都不会被保留，因为这些影响发生在另一个进程上面）。这使得 ``paraFunc`` 调用的函数应该有返回值，只有返回值可以携带数据并返回给主数据图。

**你并不需要对函数进行特别的优化来使用** ``paraFunc`` **，函数内部依然可以对属性进行修改，你可以把每个进程想象为一个管道，这些修改将沿着管道传递，管道内部先后调用的函数是可以访问到这些被修改的属性的，只不过不同的管道之间不能传递属性，你必须在管道的终点接住输出的数据，然后亲自把它们送回主进程。**

一个良好的实例是：
::

   instrumentListA.paraFunc('someFuncX',someParameterX)
   instrumentListA.paraFunc('someFuncY',someParameterY)
   instrumentListA.paraFunc('someFuncZ',someParameterZ)
   result = instrumentListA.paraRun()

这相当于：
::

   instrumentListA.execFunc('someFuncX',someParameterX)
   instrumentListA.execFunc('someFuncY',someParameterY)
   instrumentListA.execFunc('someFuncZ',someParameterZ)

注意到这里使用 ``result`` 来接受返回值， ``result`` 是一个RiskQuantLib列表，其中的每个元素是一个三元组，（因为你并行调用了三个函数，每个函数的返回值都会被打包返回。）元组的第一个元素是第一个函数 ``someFuncX`` 的返回值，第二个元素是第二个函数 ``someFuncY`` 的返回值，以此类推。

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

多进程版本和非多进程版本都是可以正常运行的。非多进程版本运行后，你的 ``instrumentListA`` 的每个元素会多出 ``codeX`` ， ``codeY`` ， ``codeZ`` 三个属性，但多进程版本运行后则不会。多进程中的 ``result`` 变量会是一个列表，每个元素是 ``(self.codeX,self.codeY,self.codeZ)`` 的三元组。

正如刚才我们说的，多进程版本中 ``someFuncY`` 访问 ``someFuncX`` 中生成的 ``codeX`` 是没有问题的，只不过 ``codeX`` 不能在 ``instrumentListA`` 中访问。