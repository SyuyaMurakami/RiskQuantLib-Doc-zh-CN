使用组件文件夹
====================

.. toctree::
   :maxdepth: 4

本章节主要涉及 ``Src`` 文件夹，它可以像任意一个文件夹一样，你可以把所有的python源代码放在其中。然后从主函数导入运行，或者直接运行 ``Src`` 文件夹中的代码。你可以在这些代码中写注释，就像一如既往地进行python编程一样。

但 ``Src`` 文件夹中的源代码文件还有一些非常特殊的地方，即 **这些代码的注释语句拥有控制代码生成的能力** 。RiskQuantLib作为脚手架，为了更方便地进行一些大型工程的开发，会使用代码来生成代码，这是RiskQuantLib的核心概念之一。在前面教程中频繁出现的模板类，模板列表类，Set函数族，get函数族，都可以看做是批量生成代码的特例。我想你已经猜到了，RiskQuantLib实际上可以以任意的规则，按照任意的逻辑来生成代码，只要你按照规范声明生成代码的方式。

如果你坚持使用RiskQuantLib来构建每一个数据分析工程，你的数据处理逻辑就会变成一些积木，越来越多的工程经验将使得RiskQuantLib拥有越来越多的积木。当你需要再次开始一个新的工程的时候，你可以通过注释来告诉RiskQuantLib如何通过已经有的积木来快速搭建一个房子，然后对于那些你需要，但是在已有的程序积木中找不到的部分，再进行编写。当然，编写的部分会成为新的积木。

**这里，我们正式地把Src文件夹成为组件文件夹，其中的单个文件成为组件。**

为什么需要Src组件文件夹
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你只想了解如何使用组件文件夹，你可以跳过这部分，进入 :ref:`how_to_use_src`。

最主要的原因
-------------------

Src文件夹可以帮助你把逻辑相关的代码放在同一个文件中，使得它成为一个可以被重复使用的积木。

类编程最主要的问题之一，是完成一个业务功能所需要的代码被分别放置在不同的类中。这确实使得工程变得更加易于维护。然而数据工程分析是一项非常不同的任务，它不像一些生产类应用，具有稳定的程序运行环境。对于数据分析工程来说，通常需要进行探索性的分析，也就是频繁对代码逻辑进行修改。这使得用户要不停地在类文件之间穿梭，来修改他们的代码。而且如果经过了一段时间，你需要重新修改代码来进行新的类似的项目，你会难以找到实现业务逻辑的代码被分别放在什么位置。

当然，写注释可以在一定程度上解决这个问题，但对于大部分程序员来说，写注释是非常痛苦的。RiskQuantLib给出的解决方案是将业务逻辑相关的代码全部放进同一个.py文件中，然后通过非常简单的注释告诉RiskQuantLib应该把他们放进哪个类文件中。你已经知道了RiskQuantLib的核心概念，就是 **用代码来生成代码** ，现在，你应该已经意识到了RiskQuantLib的另一个核心概念，即 **用代码控制代码的位置** 。

次要的原因
--------------------

你可能依然会有一些疑问，为什么我会需要一个组件文件夹呢？python或者任何语言的函数和库导入机制已经提供了将程序封装为组件的能力，用户可以方便地使用它们。比如，像我们一开始说的，你可以把所有的python源代码放在Src其中，然后从主函数导入运行。

你是对的。如果你的工程不够复杂，或者你平时不会频繁地开启一个新的数据分析项目，你应该不需要用到组件这样的功能。但数据分析和其他的代码工程非常不同，简单地说，数据分析工程不够 **稳定** 。一次又一次，数据分析师创建一个数据工程，分析，得到结论，然后把工程删除或者归档。然后重复这个过程。任何一个数据分析过程都有自己的独特之处，迫使数据分析师从过去的代码中粘贴一部分，然后做微小的修改，然后才能继续运行。

**然后你会开始思考一个问题，为什么数据分析领域的代码重用率如此低？**

答案当然是很多的，很多人归咎于python的数据类型机制，不进行类型检查，使得代码的泛用性增强的同时bug也多了很多，你不得不因为一些小的bug对代码进行反复地修改。还有人认为这个是数据分析的本质，毕竟洗数据这个过程就是不断地对数据进行微调，使得它符合之前的代码范式。

RiskQuantLib给出的答案是，**重用率低是因为代码的封装单位不够小** 。想想看，python和其他计算机语言中，几乎可重用的最小代码单位是一个函数（你可以说是一个变量，但变量没法执行一个整体的逻辑，对于函数式语言来说，变量可以是函数，从而可以执行一个完整的逻辑，所以总的而言，函数是大部分语言的代码可重复最小单位）。而一个函数则被现代数学做出了若干定义，比如输入，输出，映射的概念。（变量的限制就更多，考虑一下数据类型，它是对变量最大的限制。）

代码的本质是字母，字母的重复构成了一个完整的功能。可以说，字母是最细的代码构建颗粒。之后就到了函数与变量。这是目前的语言中第二细的代码构建颗粒。在这样的体系下，代码重用率低的问题反复出现，这使得我们开始思考，是否在这两者之间存在一个地带，代码的构建颗粒大于字母，却小于函数？

答案当然是肯定的，这就是RiskQuantLib给出的方案。这种可以在函数内部反复出现的代码，我们不妨叫它 ``chunk`` ，中文名 ``代码块`` 。这个概念被Haskell等编程语言发挥到了极致，Haskell的惰性机制很大程度上是使用 ``chunk`` 来实现的。在RiskQuantLib的Src文件夹中，你可以编写的是各种 ``代码块`` ，而不仅仅是函数。

我们总结一下，为什么我们可能需要Src组件文件夹：

**代码块是一种可以重复出现的代码语句集合，块内的代码可以不完整，无法单独执行任何功能。**

**代码块可大可小，从包含的内容多少来说，通常一个代码块大于一个单词，小于一个函数，一个函数应该由多个代码块堆叠构成。**

**代码的最小可重复逻辑单元从函数变成了更小的：代码块。**

**最后，代码块可以重复使用，这种重复使用可以被程序自动化，从而提升代码的重用率。**

.. _how_to_use_src:

如何使用Src组件文件夹
^^^^^^^^^^^^^^^^^^^^^^^^^^

注释控制语句
------------------

*分发*
>>>>>>>>>>>>

在之前的关于如何构建你的工程的教程中，你应该已经接触如何使用Src文件夹的注释控制语句。我们来回顾这个例子：
::

   #->stock
   def sayHello(self):
       self.greeting = self.name + "_hello"

这些代码位于 ``Src/test.py`` 中，它是顶格编写的，不存在于任何类内部。实际上它是 ``Src/test.py`` 文件的全部内容。

它的含义是定义一个 ``sayHello`` 函数，把它绑定给 ``stock`` 模板类。你应该注意到了函数定义的上方一行的注释，以正常的 ``#`` 开头，但紧跟着一个 ``->`` 符号。这是注释控制语句的一种，名为 **分发** 。当你需要向一个模板类中增加一个类函数的时候，你可以使用注释来告诉RiskQuantLib，到底应该把这个函数放在哪里。

**注意：分发语句的生效范围是它之后的若干行。分发语句必须位于行首，直到碰见下一个分发语句或者文件结束，分发语句会把这之间的所有代码全部放入目标位置。**

*标签*
>>>>>>>>>>>>

再来看另一个例子：
::

   #->bond@import
   import os

它的含义是把 ``import os`` 语句分发到 ``bond`` 模板类的 ``import`` 标签下。这里的 ``@`` 分割了想要分发的目的地文件名和目的标签的名称。这里我们遇见了所谓的标签，它并不难以理解，因为一个目标.py文件中有很多个可以插入代码的位置，我们要如何知道应该把这一行代码插入到目标文件的第三行还是第五行呢？标签的作用就是帮助我们确定位置。

你可以打开目标文件，它的位置是 ``RiskQuantLib/Instrument/Security/Bond/bond.py`` ，你可以看见标签的样式，它是这样的：
::

   #<import>
   #</import>

标签非常类似于html中的元素。标签的开始是 ``#<tagName>`` ，标签的结束是 ``#</tagName>`` 。代码可以被自动插入到标签的开始和结束之间。每当你对工程进行一次编译，这些标签之间的内容就会被重新构建。

**注意：被分发的代码只能被插入到标签中间，标签中间的代码是程序自动生成的，用户的自定义代码不应该写在标签之间。**

**注意：你可以定义自己的标签，自定义的标签也可以在注释控制语句中使用。** 

比如，你在 ``RiskQuantLib/Instrument/Security/Bond/bond.py`` 中新增了另一个标签，名为 ``ifItIsConvertibleBond`` ，之后你的文件会看起来像这样：
::

   #!/usr/bin/python
   #coding = utf-8
   import numpy as np
   import pandas as pd
   from RiskQuantLib.Instrument.Security.security import security
   from RiskQuantLib.Auto.Instrument.Security.Bond.bond import bondAuto
   from QuantLib import Bond
   #<import>
   #</import>

   class bond(security,Bond,bondAuto):
       """
       bond is an instrument class, used as nodes of data graph.
       """
       #<init>
       def __init__(self,codeString,nameString,instrumentTypeString = 'Bond'):
           security.__init__(self,codeString,nameString,instrumentTypeString)
       #</init>

       #<initQuantLib>
       def iniPricingModule(self,*args):
           Bond.__init__(self,*args)
       #</initQuantLib>

       #<bond>
       #</bond>

       #<ifItIsConvertibleBond>
       #</ifItIsConvertibleBond>

然后你就可以在 ``Src/test.py`` 中编辑代码：
::

   #->bond@ifItIsConvertibleBond
   def ifItIsConvertibleBond(self):
       if self.bondType == 'convertibleBond':
           return True
       else:
           return False

编译之后，这个函数就会出现在对应的标签之间。

**注意：你可以在任何位置，包括函数内部定义标签。生成的代码将保持标签的缩进水平。**

让我们看看在函数内部定义标签的例子：
::

   class bond(security,Bond,bondAuto):
       """
       bond is an instrument class, used as nodes of data graph.
       """
       #<init>
       def __init__(self,codeString,nameString,instrumentTypeString = 'Bond'):
           security.__init__(self,codeString,nameString,instrumentTypeString)
       #</init>

       #<initQuantLib>
       def iniPricingModule(self,*args):
           Bond.__init__(self,*args)
       #</initQuantLib>

       #<bond>
       #</bond>

       def ifItIsConvertibleBond(self):
           if self.bondType == 'convertibleBond':
               #<ifItIsConvertibleBond>
               #</ifItIsConvertibleBond>
           else:
               return False

然后你就可以在 ``Src/test.py`` 中编辑代码：
::

   #->bond@ifItIsConvertibleBond
   print("This is a convertible bond!")
   return True

编译之后，这个 ``代码块`` 就会出现在对应的标签之间。

*多个目标*
>>>>>>>>>>>>

**目的地有多个目标时，可以使用逗号来分割它们。**

简单地，比如你需要在Src文件夹下的一个单独的文件 ``import.py`` 中控制整个项目的导入库，你可以这样做：
::

   #->stock@import,bond@import,future@import,option
   import numpy as np
   import pandas as pd

你可以随时在控制注释后面新增一些目的地，来告诉RiskQuantLib需要将这个 ``代码块`` 放进另一个目标位置。当然，这些目标的标签可以互不相同，也可以不带标签。

多逻辑项目
------------------

通常一个数据分析师在分析同一个问题的时候，会开启多个数据工程。简单地，如果你想要知道一种股票交易策略是否有效，并且将它运用在交易中。那么很可能，你会先开启一个数据工程，分析过去的数据，得到是否有效的结论。如果有效，你会再开启一个生产工程，每天向你推送那些筛选出来的交易标的。

问题是，这样的过程为何总是在重复？

有了Src文件夹提供的组件功能，我们可以很好地解决这个问题。RiskQuantLib将数据结构和数据处理逻辑分离，我们所有的处理逻辑都被放进了Src文件夹，而数据结构则由用户自行定义，工程结构则由 ``config.py`` 来保存。当我们需要由回测转向生产时，我们仅仅需要更换Src文件夹下面的内容，同时保持其他部分不变。

Src文件夹下方唯一有效的源代码文件是以 ``.py`` 结尾或者 ``.pyt`` 结尾的文件，如果我们需要注释掉整个代码逻辑，我们可以更改文件的文件名后缀，编译工程之后，整个处理逻辑就会从工程中消失。

你甚至可以通过更改文件夹名称的方式来实现这一点。假设你的回测框架的代码都位于Src文件夹中，更改文件夹的名字为 ``Back_Test`` ，然后创建一个新的空白的Src文件夹，重新编译，会一次性将工程换为一个生产环境，同时保留数据结构和工程结构。

当然，RiskQuantLib提供了方便的方式在两套处理逻辑之间切换，如果你在生产逻辑下想要重新回到回测逻辑，则运行下方的命令：
::

   python build.py -r Back_Test

``-r`` 表示从一个特定的文件夹来进行编译。