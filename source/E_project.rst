文本计数项目
====================

.. toctree::
   :maxdepth: 4

Become closer to RiskQuantLib
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

让我们一起来完成一个数据关系较为复杂的任务。这会让你了解到RQL如何使用图数据结构、双层节点、提线木偶（指针）等设计让问题变得简单。

A text counting problem
^^^^^^^^^^^^^^^^^^^^^^^

假设你是一位ESG报告研究员，你正在研究哪些名词的使用会使得企业的ESG评级更高。在分析之前，你严格的上司提出了一些令人头疼的要求。他认为只有一部分名词可以体现企业进行环境保护的行为，这些名词应当符合以下条件：

1.  该词在至少75%的企业的报告中出现至少1次。这意味着这个词或许和体现ESG行为有一定关联。
2.  该词不是某个行业的特殊用词。这意味着这个词在任何一个行业中的出现频率不应显著高于其他行业。


你现在收集到了一些日本上市企业的行业信息，保存在JP.xlsx中。你还收集了这些企业的环保责任报告（E report）的名词使用情况，储存在一些txt文件中。
+------ --+-----------------------------------------------------+
| Company |     Industry      |     	  E Using Words         |   
+===== ===+=====================================================+
|  Asahi  |  Chemical Product | fuel, air, efficiency, coast,...|     
+------ --+-----------------------------------------------------+
|  Chubu  |  Electric Power   | carbon, nature, cloud,...       |
+----- ---+-----------------------------------------------------+
|  ...    |                   |                                 |
+---------+-----------------------------------------------------+

面对这样复杂的任务，你决定使用RiskQuantLib。你在command中输入newRQL path\myESG来新建了一个RQL项目。并在项目根目录下新建了Data文件夹存入数据。随后你修改了两个xlsx文件并:ref:`build`了company和industry这两个类。

``Build_Instrument.xlsx`` looks like:

+--------------+------------------+-----------------------+------------+---------------------+
|InstrumentName|ParentRQLClassName|ParentQuantLibClassName|LibararyName|DefaultInstrumentType|
+==============+==================+=======================+============+=====================+
|   industry   |                  |                       |            |       industry      |
+--------------+------------------+-----------------------+------------+---------------------+
|   company    |                  |                       |            |       company       |
+--------------+------------------+-----------------------+------------+---------------------+

你为company类准备了industry属性和usedWords属性。在build环节设置类的属性，可以使build后该类的RQLlist拥有便捷的set方法。
``Build_Attr.xlsx`` looks like:

+--------------+-------------------+----------------+
| SecurityType |      AttrName     |    AttrType    |
+==============+===================+================+
|   Company    |      industry     |     String     |
+--------------+-------------------+----------------+
|   Company    |      usedWords    |                |
+--------------+-------------------+----------------+
|     ...      |         ...       |       ...      |
+--------------+-------------------+----------------+

随后你执行了build.py，这生成了上述类，并自动生成他们各自的RQLlist类。
随后你打开了main.py文件开始了数据分析之旅。此时你的main.py文件是这样的：

``main.py``
::

    import os
    import sys
    from RiskQuantLib.Module import *
    path = sys.path[0]

一个RQL项目的常用设计框架是数据导入，数据分析和结果输出。你尝试用这样的框架设计代码。

*Data Input*

让我们先导入数据，为了方便之后实例的值的批量设置，我们常常将属性导入为可迭代的对象，且他的顺序是和实例的顺序相一致的（也就是说公司A如果出现在公司列表的第3位，那么公司A的usedWords属性也应当出现在usedWords列表的第3位）。于是我们将企业的usedWords设计成一个如下的列表。

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

实例化company_list和industry_list，并使用add函数为他们加入元素，这个过程中也为元素添加了用于识别的code属性。set函数为company_list的company元素添加了industry属性和usedWords属性。

``main.py``
::

    company_list = companyList()
    company_list.addCompanySeries(companies, companies)
    company_list.setIndustry(companies, industries)
    company_list.setUsedWords(companies, companies_words)

    industry_list = industryList()
    industry_list.addIndustrySeries(industries, industries)

你可以在debug的过程中使用company_list[code]或者company_list[0]来找到对应的company元素（返回一个company实例）。

``main.py``
::

    company_list["Asahi"]
    # or company_list[0]

在检查你的company属性的过程中，你发现有一些企业的wordsList的长度为0——这说明这个企业的数据存在缺失。你决定通过RQL list的filter函数筛掉这些元素。当filter的参数useObj为True时，他的结果仍然是一个RQL list。于是，你在main.py中继续输入

``main.py``
::

    company_list = company_list.filter(lambda company:len(company.usedWords) != 0, useObj=True)

*Data Analysis*

随后让我们进入（复杂的）分析过程。，你决定先统计每一个company的用词频率。你打开了.\RiskQuantLib\Company\company.py，在Company类中添加了一条countUsedWords方法如下

``RiskQuantLib.Company.company``
::

    class company(setCompany):
        ...
        def countUsedWordsDict(self):
            from collections import Counter
            self.usedWordsDict = Counter(self.usedWords)

你在main.py中执行了company_list.exec("countUsedWords")，使得对于列表中的每个元素调用它的countWords方法。你的每个Company元素会因此获得额外的属性usedWordsDict。

``main.py``
::

    company_list.execFunc("countUsedWordsDict")

为了满足领导的要求1，你需要统计哪些词至少在75%的企业中出现过至少一次。你打开了.\RiskQuantLib\CompanyList\companyList.py，在Company_list中添加了一条方法如下，

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

在main.py中执行company_list的rule_one方法，使得company_list增加了一条属性rule_one。

``main.py``
::

    company_list.rule_one()


之后，你决定使用RQL的connect函数将company_list和industry_list中的某些有关联的元素进行链接，使得他们作为彼此的属性可以进行调用，这是RQL的核心理念之一（图结构数据存储）。你输入代码如下，

``main.py``
::

    company_list.connect(industry_list, 
                         targetAttrNameOnLeft="industryObj",
                         targetAttrNameOnRight="companiesObj",
                         filterFunction=lambda x,y:x.industry == y.name)

我们可以看见每个company元素有了新的属性industryObj，而industry元素有了新的属性companiesObj——他们都是RQL list。

你决定先统计每个行业的企业平均用词情况，你在.\RiskQuantLib\Industry\industry.py，在industry类中添加了一条方法如下，

``RiskQuantLib.Industry.industry``
::

    class industry(setIndustry):
        ...
        def countAvgWords(self):
            from collections import Counter
            countWords = Counter()
            [countWords.update(company.usedWords) for company in self.CompaniesObj]
            self.avgWords = {word:countWords[word]/len(self.CompaniesObj.all) for word in countWords.keys()}

你在main.py中执行了industry_list.execFunc("avgCountWords")，使得对于列表中的每个元素调用它的countAvgWords方法。你的每个Industry元素会因此获得新的属性avgWords。

``main.py``
::

    industry_list.execFunc("countAvgWords")

你希望可以满足领导的要求2，对于每一个company_list.rule_one中的词，需要去检查它是否显著的频繁出现于某一个行业。你决定统计每个单词在各行业的使用频率，你打开了.\RiskQuantLib\IndustryList\industryList.py，在Industrylist中添加了一条方法removeBiasWords如下，.

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

在执行Industry_list.rule_two后，industry_list增加了一条属性rule_two。

``main.py``
::

    industry_list.rule_two(rule_one = company_list.rule_one)

于是你可以使用这个满足领导要求的单词表去进行筛选了！你在company类中定义如下函数findUsefulWords

``RiskQuantLib.Company.company``
::

    class company(setCompany):
        ...
        def findUsefulWordsDict(self, rule_two):
            self.usefulWordsDict = {word:self.usedWordsDict[word] for word in self.usedWordsDict.keys() if word in rule_two}

在执行company_list.execFunc(“findUsefulWords”, industry.rule_two)后，每个企业会得到属性usefulWordsDict。

``main.py``
::

    company_list.execFunc("findUsefulWords", industry.rule_two)

*Data Output*

这是满足领导意见的词表，我们可以用他做后续的研究了！

Summary
^^^^^^^

让我们回顾一下项目的整个流程：

1.思考节点类的结构关系和属性，修改instrument和attr两个xlsx文件，运行build.py。
2.导入数据。实例化RQLlist并加入元素，set元素节点的属性。
3.开始分析。往返于main.py和类，为元素节点和list节点设计各种方法。并将调用方法的结果作为属性存在该节点上（或任何其他位置）。
4.重复3的过程，直到得到最终结果。
5.导出数据。

RQL的设计为使用者提供自定义的数据分析模式，更多的设计请点击:ref:` `。

