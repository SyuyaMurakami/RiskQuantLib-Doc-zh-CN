图数据结构编程
====================

.. toctree::
   :maxdepth: 4

图数据结构基础
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一般我们说到数据库，大家的第一印象通常是SQL。SQL作为 **关系型** 数据库的通用查询语言在过去几十年得到了广泛的发展和应用。但正如它的名字所言，它仅仅是一种关系型数据库的查询方式，而其他类型的数据库，比如图数据库，则使用cypher作为查询语言。如果我们打开思路，我们就会发现数据的处理方式并非只有关系型。

在很多情况下，关系型数据确实是人们最容易理解和操作的数据，日常我们遇见的任何表格，excel文档等都可以看做是关系型数据。他们的典型特征是数据以行和列的形式存储，每一行的数据之间存在固定的关系，这些关系通常由列名来标识。

但随着数据科学的发展，在越来越多的情况下，数据之间的关系变得不再那么规范。你可能已经注意到，在任何时候都把数据保存为行列这样的矩阵形式并不是最方便的。很多时候你可能发现某些行有很多空值，或者上一行的长度不等于下一行（excel或许喜欢用合并单元格这样的方式来处理这个问题）。但如果我们仔细思考，我们会发现，所谓的空值往往来源于关系的缺失，即在那一行中其实并不存在这样的关系。而进一步思考会让我们发现，空值来源于我们强迫地使数据展现为行和列的方式，使得本就不应该存在的关系必须占用一个位置，这个位置的值自然只能为空。

**所以我们是时候开始承认，数据并不是总能表现为矩阵这样的关系形式，也不应该总表现为矩阵这样的关系形式。**

图数据结构是对关系型数据结构的一种很好的替代。它起源于集合论。图数据结构中，最基本的概念是 **节点** 以及 **链接**，节点是用来存储静态数据的单元，而链接通常表示节点与节点之间的关系。比如，我们有一个称为 ``human`` 的节点，同时有 ``father`` 以及 ``son`` 节点，那么我们可以在其间建立关系： ``human`` 包括了  ``father`` 以及 ``son``。

关于更多的图数据结构问题，你可以找到很多相关的资料，在这里我们就不一一赘述。比如 `W3cSchool <https://www.w3cschool.cn/neo4j/neo4j_graph_theory_basics.html>`_ 就在图数据库Neo4j的教程中提供了相应的介绍，这些简单易懂的图例会帮助我们理解什么是图数据结构。



进一步了解RiskQuantLib
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RiskQuantLib使用模板类和模板列表类来处理和分析数据，但如果我们只停留在这里，那么RiskQuantLib的功能将非常局限。要充分发掘RiskQuantLib的潜力，我们需要换一种方式来看待模板类和模板列表类：

    `模板类和模板列表类不仅仅是一个python的类文件，在RiskQuantLib中，它们也是图数据结构下的一个节点。`

这也是RiskQuantLib的核心设计，任何一类节点都有两层（你也可以把这两层合称为一个节点），即顶层的模板列表类和底层的模板类。顶层的模板列表类通常用于处理同一类节点内部的关系，而底层的模板类则作为具体的独立节点，负责与其他类型的节点进行交互，或者在自身的作用域下进行运算。

RiskQuantLib通过两个关键的函数来完成图数据结构下各个节点之间的链接： ``join``, ``connect`` ：

*Join*

``join`` 是用于在两个模板列表类的元素之间建立 **单向链接** 的函数。

如果 ``stockListA`` 是一个 ``stockList`` 的实例，同时 ``companyListB`` 是一个 ``companyList`` 的实例，而且我们还有一个字典 ``stockIPODetail``，其中保存了公司和股票代码之间的一一映射关系，比如A公司发行的股票代码为0123.HK。注意在这里，同一家公司可能在两个不同的交易所发行两个不同代码的股票，比如同时在纽约交易所和香港交易所上市就一度是一件非常流行的事情。

``join`` 函数是任何模板列表类的属性函数，你可以对任何模板列表类使用它。它的参数如下：
::

    riskQuantLibList.join(anotherList, targetAttrName, filterFunction)

当调用了这个函数后，对于 ``riskQuantLibList`` 的每一个元素, 程序都会从 ``anotherList`` 的元素中寻找哪些满足条件的元素，然后把这些元素汇总为一个模板列表，作为一个属性挂载在当前的元素下。这个属性的名称就是 ``targetAttrName`` 声明的名称，而筛选条件就是 ``filterFunction`` 声明的筛选条件。比如在上面的例子中：
::

    companyListB.join(stockListA, 'issuedStock', lambda company, stock: stockIPODetail[company.name]==stock.code)

当运行这一行之后， ``companyListB`` 的每一个元素， 也就是 ``company`` 对象下回多出一个属性，名为 ``issuedStock``， 这个属性的值是一个  ``stockList``，其中包含了所有该公司发行的股票。如果该公司没有发行股票，那么这个 ``stockList`` 的长度就为0，反之，发行了多只股票的情况下，其中就会包含所有的股票。

*Connect*

``connect`` 是用于在两个模板列表类的元素之间建立 **双向链接** 的函数。

同样的，``connect`` 函数也是任何模板列表类的属性函数，你可以对任何模板列表类使用它。它的参数如下：
::

    riskQuantLibList.connect(anotherList, targetAttrNameOnLeft, targetAttrNameOnRight, filterFunction)

当调用了这个函数后，对于 ``riskQuantLibList`` 的每一个元素, 程序都会从 ``anotherList`` 的元素中寻找哪些满足条件的元素，然后把这些元素汇总为一个模板列表，作为一个属性挂载在当前的元素下。这个属性的名称就是 ``targetAttrNameOnLeft`` 声明的名称，而筛选条件就是 ``filterFunction`` 声明的筛选条件。

然后，对于 ``anotherList`` 的每一个元素, 程序都会从 ``riskQuantLibList`` 的元素中寻找哪些满足条件的元素，然后把这些元素汇总为一个模板列表，作为一个属性挂载在当前的元素下。这个属性的名称就是 ``targetAttrNameOnRight`` 声明的名称，而筛选条件就是 ``filterFunction`` 声明的筛选条件。

比如在上面的例子中：
::

    companyListB.connect(stockListA, 'issuedStock', 'issuedBy', lambda company, stock: stockIPODetail[company.name]==stock.code)

当运行这一行之后， ``companyListB`` 的每一个元素， 也就是 ``company`` 对象下会多出一个属性，名为 ``issuedStock``， 这个属性的值是一个  ``stockList``，其中包含了所有该公司发行的股票。如果该公司没有发行股票，那么这个 ``stockList`` 的长度就为0，反之，发行了多只股票的情况下，其中就会包含所有的股票。

同时，``stockListA`` 的每一个元素，也就是 ``stock`` 对象下会多出一个属性，名为 ``issuedBy``， 这个属性的值是一个  ``companyList``，其中包含了所有发行该股票的公司（一般一只股票只会由一个公司发行）。如果字典中没有存储该股票的发行信息，那么这个 ``companyList`` 的长度就为0。

让我们一起来完成一个数据关系较为复杂的任务。这会让你了解到RQL如何使用图数据结构、双层节点、提线木偶（指针）等设计让问题变得简单。

一个文本计数项目
^^^^^^^^^^^^^^^^^^^^^^^

假设你是一位ESG报告研究员，你正在研究哪些名词的使用会使得企业的ESG评级更高。在分析之前，你严格的上司提出了一些令人头疼的要求。他认为只有一部分名词可以体现企业进行环境保护的行为，这些名词应当符合以下条件：
::

    1.该词在至少75%的企业的报告中出现至少1次。这意味着这个词或许和体现ESG行为有一定关联。
    2.该词不是某个行业的特殊用词。这意味着这个词在任何一个行业中的出现频率不应显著高于其他行业。


你现在收集到了一些日本上市企业的行业信息，保存在 ``JP.xlsx`` 中。你还收集了这些企业的环保责任报告（**E report**）的名词使用情况，储存在一些txt文件中。

+---------+-------------------+---------------------------------+
| Company |     Industry      |     	  E Using Words         |   
+=========+===================+=================================+
|  Asahi  |  Chemical Product | fuel, air, efficiency, coast,...|     
+---------+-------------------+---------------------------------+
|  Chubu  |  Electric Power   | carbon, nature, cloud,...       |
+---------+-------------------+---------------------------------+
|  ...    |       ...         |      ...                        |
+---------+-------------------+---------------------------------+

面对这样复杂的任务，你决定使用RiskQuantLib。你在终端中输入 ``newRQL path\myESG`` 来新建了一个RQL项目，并在项目根目录下新建了 ``Data`` 文件夹存入数据。随后你修改了两个xlsx文件并 `编译 <https://riskquantlib-doc.readthedocs.io/zh_CN/latest/Build_Project.html>`_ 生成了 ``company`` 和 ``industry`` 这两个类。

``Build_Instrument.xlsx`` 看起来像这样:

+--------------+------------------+-----------------------+------------+---------------------+
|InstrumentName|ParentRQLClassName|ParentQuantLibClassName|LibararyName|DefaultInstrumentType|
+==============+==================+=======================+============+=====================+
|   industry   |                  |                       |            |       industry      |
+--------------+------------------+-----------------------+------------+---------------------+
|   company    |                  |                       |            |       company       |
+--------------+------------------+-----------------------+------------+---------------------+

你为 ``company`` 类准备了 ``industry`` 属性和 ``usedWords`` 属性。在 ``build`` （编译）环节设置类的属性，可以使 ``build`` 后该类的RQLlist拥有便捷的 ``set`` 方法。

``Build_Attr.xlsx`` 看起来像这样:

+--------------+-------------------+----------------+
| SecurityType |      AttrName     |    AttrType    |
+==============+===================+================+
|   Company    |      industry     |     String     |
+--------------+-------------------+----------------+
|   Company    |      usedWords    |                |
+--------------+-------------------+----------------+
|     ...      |         ...       |       ...      |
+--------------+-------------------+----------------+

随后你执行了 ``build.py`` ，这生成了上述类，并自动生成他们各自的模板列表类。
随后你打开了 ``main.py`` 文件开始了数据分析之旅。此时你的 ``main.py`` 文件是这样的：

``main.py``
::

    import os
    import sys
    from RiskQuantLib.Module import *
    path = sys.path[0]

一个RQL项目的常用设计框架是数据导入，数据分析和结果输出。你尝试用这样的框架设计代码。

*Data Input*

让我们先导入数据，为了方便之后实例的值的批量设置，我们常常将属性导入为可迭代的对象，且他的顺序是和实例的顺序相一致的（也就是说公司A如果出现在公司列表的第3位，那么公司A的 ``usedWords`` 属性也应当出现在 ``usedWords`` 列表的第3位）。于是我们将企业的 ``usedWords`` 设计成一个如下的列表。

``main.py``
::

    import pandas as pd
    df = pd.read_excel(r".\Data\JP.xlsx")
    def input_text(dirpath): 
        words = []
        with open(dirpath + os.sep + "noun.txt", 'r') as f:
            for item in f:
                words.append(item[:-1])
        return words

    companies = df["Name"]
    industries = df["Industry"]
    companies_words = [input_text(r".\Data\{0}".format(i)) for i in companies]

实例化 ``company_list`` 和 ``industry_list`` ，并使用 ``add`` 函数为他们加入元素，这个过程中也为元素添加了用于识别的 ``code`` 属性。 ``set`` 函数为 ``company_list`` 的 ``company`` 元素添加了 ``industry`` 属性和 ``usedWords`` 属性。

``main.py``
::

    company_list = companyList()
    company_list.addCompanySeries(companies, companies)
    company_list.setIndustry(companies, industries)
    company_list.setUsedWords(companies, companies_words)

    industry_list = industryList()
    industry_list.addIndustrySeries(industries, industries)

你可以在debug的过程中使用 ``company_list[code]`` 或者 ``company_list[0]`` 来找到对应的 ``company`` 元素（返回一个 ``company`` 实例）。

``main.py``
::

    company_list["Asahi"]
    # or company_list[0]

在检查你的 ``company`` 属性的过程中，你发现有一些企业的 ``wordsList`` 的长度为0，这说明这个企业的数据存在缺失。你决定通过RQL list(模板列表类)的 ``filter`` 函数筛掉这些元素。当 ``filter`` 的参数 ``useObj`` 为 ``True`` 时，他的结果仍然是一个RQL list。于是，你在 ``main.py`` 中继续输入：

``main.py``
::

    company_list = company_list.filter(lambda company:len(company.usedWords) != 0, useObj=True)

*Data Analysis*

随后让我们进入（复杂的）分析过程。，你决定先统计每一个company的用词频率。你打开了 ``RiskQuantLib\Company\company.py``，在 ``Company`` 类中添加了一条 ``countUsedWords`` 方法，具体操作如下：

``RiskQuantLib.Company.company``
::

    class company(setCompany):
        ...
        def countUsedWordsDict(self):
            from collections import Counter
            self.usedWordsDict = Counter(self.usedWords)

你在 ``main.py`` 中执行了 ``company_list.exec("countUsedWords")`` ，使得对于列表中的每个元素调用它的 ``countUsedWords`` 方法。你的每个 ``Company`` 元素会因此获得额外的属性 ``usedWordsDict``。

``main.py``
::

    company_list.execFunc("countUsedWordsDict")

为了满足领导的要求1，你需要统计哪些词至少在75%的企业中出现过至少一次。你打开了 ``RiskQuantLib\CompanyList\companyList.py``，在 ``Company_list`` 中添加了一条方法如下，

``RiskQuantLib.CompanyList.companyList``
::

    class companyList(listBase,setCompanyList):
    ...
    def rule_one(self):
        threshold = len(self.all) * 0.75

        from collections import Counter
        word_dict = Counter()
        for company in self.all:
            word_dict.update({word:1 for word in company.usedWordsDict.keys()})

        self.rule_one = [word for word in word_dict.keys() if word_dict[word] > threshold]

在 ``main.py`` 中执行 ``company_list`` 的 ``rule_one`` 方法，使得 ``company_list`` 增加了一条属性 ``rule_one``。

``main.py``
::

    company_list.rule_one()


之后，你决定使用RQL的 ``connect`` 函数将 ``company_list`` 和 ``industry_list`` 中的某些有关联的元素进行链接，使得他们作为彼此的属性可以进行调用，这是RQL的核心理念之一（图结构数据存储）。你输入代码如下：

``main.py``
::

    company_list.connect(industry_list, 
                         targetAttrNameOnLeft="industryObj",
                         targetAttrNameOnRight="companiesObj",
                         filterFunction=lambda x,y:x.industry == y.name)

我们可以看见每个 ``company`` 元素有了新的属性 ``industryObj``，而 ``industry`` 元素有了新的属性 ``companiesObj`` ，他们都是RQL list。

你决定先统计每个行业的企业平均用词情况，你在 ``RiskQuantLib\Industry\industry.py``，在 ``industry`` 类中添加了一条方法如下，

``RiskQuantLib.Industry.industry``
::

    class industry(setIndustry):
        ...
        def countAvgWords(self):
            from collections import Counter
            countWords = Counter()
            [countWords.update(company.usedWords) for company in self.CompaniesObj]
            self.avgWords = {word:countWords[word]/len(self.CompaniesObj.all) for word in countWords.keys()}

你在 ``main.py`` 中执行了 ``industry_list.execFunc("avgCountWords")``，使得对于列表中的每个元素调用它的 ``countAvgWords`` 方法。你的每个 ``Industry`` 元素会因此获得新的属性 ``avgWords``。

``main.py``
::

    industry_list.execFunc("countAvgWords")

你希望可以满足领导的要求2，对于每一个 ``company_list.rule_one`` 中的词，需要去检查它是否显著的频繁出现于某一个行业。你决定统计每个单词在各行业的使用频率，你打开了 ``RiskQuantLib\IndustryList\industryList.py``，在 ``Industrylist`` 中添加了一条方法 ``removeBiasWords`` 如下：

``RiskQuantLib.IndustryList.industryList``
::

    class industryList(listBase,setIndustryList):
        ...
        def rule_two(self, rule_one, n_sigma=2):
            self.rule_two = []
            for word in rule_one:
                frequency_list = [industry.avgWords[word] if industry.avgWords.get(word, -1) != -1 else 0 for industry in self.all]
                if max(frequency_list) < np.mean(frequency_list) + n_sigma * np.std(frequency_list):
                    self.rule_two.append(word)

在执行 ``Industry_list.rule_two`` 后，``industry_list`` 增加了一条属性 ``rule_two``。

``main.py``
::

    industry_list.rule_two(rule_one = company_list.rule_one)

于是你可以使用这个满足领导要求的单词表去进行筛选了！你在 ``company`` 类中定义如下函数 ``findUsefulWords``：

``RiskQuantLib.Company.company``
::

    class company(setCompany):
        ...
        def findUsefulWordsDict(self, rule_two):
            self.usefulWordsDict = {word:self.usedWordsDict[word] for word in self.usedWordsDict.keys() if word in rule_two}

在执行 ``company_list.execFunc(“findUsefulWords”, industry.rule_two)`` 后，每个企业会得到属性 ``usefulWordsDict``。

``main.py``
::

    company_list.execFunc("findUsefulWords", industry.rule_two)

*Data Output*

这是满足领导意见的词表，我们可以用他做后续的研究了！

总结
^^^^^^^

让我们回顾一下项目的整个流程：

1.思考节点类的结构关系和属性，修改 ``instrument`` 和 ``attr`` 两个xlsx文件，运行 ``build.py``。
2.导入数据。实例化RQLlist（模板列表类）并加入元素，使用 ``set`` 函数来设置元素节点的属性。
3.开始分析。**往返** 于 ``main.py`` 和各个模板类（在这个项目中是 ``company`` 和 ``industry`` ）之间，为元素节点和list节点设计各种方法。并将调用方法的结果作为属性存在该节点上（或任何其他位置）。
4.重复3的过程，直到得到最终结果。
5.导出数据。

RQL的设计为使用者提供自定义的数据分析模式，使用怎样的设计取决于使用者面对的数据问题。

