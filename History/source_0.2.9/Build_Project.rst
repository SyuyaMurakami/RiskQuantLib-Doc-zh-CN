Build Your Project
====================

.. toctree::
   :maxdepth: 4

After create a project, you should build it to suit for your mission. By ``Build``, RiskQuantLib will generate python source code automatically. It is the core conception of RiskQuantLib. Before moving on, let specify some definition in RiskQuantLib:

``Build`` means generate source code automatically.

``Instrument`` means any class you will use in your mission. For financial analysis, ``Instrument`` refers to stock, bond or other security type, or like interest rate or company, etc.

Then we start to build our first project. The critical thing is to tell RiskQuantLib how to build it. This is done by specify key word in ``Build_Instrument.xlsx`` and ``Build_Attr.xlsx``, which you can find in your project root dictionary once you create a project. The content of building will be written into ``RiskQuantLib.Auto``. 

In last chapter we create a project, it looks like:
::

   --your_project_path
     --RiskQuantLib
     --Src
     --build.py
     --main.py
     --Build_Attr.xlsx
     --Build_Instrument.xlsx

We will explain the function of each term as follows:

RiskQuantLib
^^^^^^^^^^^^

``RiskQuantLib`` is a dictionary holding all source files of RiskQuantLib, it looks like:
::

   --RiskQuantLib
     --Auto
     --Build
     --Instrument
     --InstrumentList
     --Model
     --Operation
     --Property
     --Tool
     --__init__.py
     --module.py

Src
^^^^
``Src`` is a dictionary holding all source files that is needed in your project. You can leave ``Src`` alone and never use it. It won't influence your project at all. Or you can put any python source code in it, run or import them as you always know in python, RiskQuantLib is also cool with that. However, if you want to try some new things, ``Src`` will give you a great way to manage your code. 

**In short, any code in** ``Src`` **can be parsed by RiskQuantLib according to the comment in source code, the control comment syntax will help to generate python code as you wish.**

The most simple way to control your code is to tell RiskQuantLib where they should be insert. You can insert any code into any position of any file as long as this file is under ``RiskQuantLib`` directory. If you want to do this, use ``#->`` control comment.

For example, you have a instrument class named as ``stock``, and you want to add a class method to append a string ``_hello`` after the ``name`` attribute, and mark the value as a new attribute named as ``greeting``. You can go to ``RiskQuantLib/Instrument/Security/Stock.stock.py`` to add this function:
::

   def sayHello(self):
       self.greeting = self.name + "_hello"

This is usually what we do. But with ``Src``, you can do it another way. First, create a file named ``greeting.py`` under ``Src``, then open it and write:
::

   #->stock
   def sayHello(self):
       self.greeting = self.name + "_hello"

Then close this file and open your terminal, change working directory into current project, run:
::

   python build.py

And this function will be inserted into ``RiskQuantLib/Instrument/Security/Stock.stock.py`` automatically. Cool, isn't it?

**For the automatically generated instrument by RiskQuantLib, use instrument name will be enough to specify a target destination, like** ``#->stock``. **But if you create the source file by yourself, you should use the absolute import path to tell RiskQuantLib where this file is, like:** ``#->RiskQuantLib.Instrument.Security.Stock.stock``

Build_Instrument.xlsx
^^^^^^^^^^^^^^^^^^^^^

``Build_Instrument.xlsx`` is the file to tell RiskQuantLib how to generate python source code of instrument class. After calling ``build.py``, Any instrument specified will be created, the source file will be added into ``RiskQuantLib`` dictionary besides its first RiskQuantLib parent class. ``Build_Instrument.xlsx`` looks like:

+--------------+------------------+-----------------------+------------+---------------------+
|InstrumentName|ParentRQLClassName|ParentQuantLibClassName|LibararyName|DefaultInstrumentType|
+==============+==================+=======================+============+=====================+
| treasureBond |       Bond       |                       |    numpy   |   Treasure Bond     |
+--------------+------------------+-----------------------+------------+---------------------+
|      ....    |        ...       |           ...         |      ...   |           ...       |
+--------------+------------------+-----------------------+------------+---------------------+

The *InstrumentName* column can be defined by your will, depending on your data. RiskQuantLib will create instrument class source file ``treasureBond.py`` under ``RiskQuantLib.Instrument.Security.Bond.TreasureBond``, and create list class source file ``treasureBondList.py`` under  ``RiskQuantLib.InstrumentList.SecurityList.BondList.TreasureBondList`` after your run ``python build.py``.

The *ParentRQLClassName* means this new instrument will inherit from this RiskQuantLib class. It can accept key word like: ``Any``, ``Fund``, ``Stock``, ``Bond``, ``Repo``, etc. However,

**Notice: You can use the instrumentName you specified in last row to fill ParentRQLClassName column, like:**

+--------------+------------------+-----------------------+------------+---------------------+
|InstrumentName|ParentRQLClassName|ParentQuantLibClassName|LibararyName|DefaultInstrumentType|
+==============+==================+=======================+============+=====================+
| treasureBond |       bond       |                       |    numpy   |   Treasure Bond     |
+--------------+------------------+-----------------------+------------+---------------------+
|chinaTreaBond |   treasureBond   |                       |            |     Panda Bond      |
+--------------+------------------+-----------------------+------------+---------------------+
|      ....    |        ...       |           ...         |      ...   |           ...       |
+--------------+------------------+-----------------------+------------+---------------------+

The *ParentQuantLibClassName* means this new instrument will inherit from this QuantLib class. It can accept key word like: ``Instrument``, ``Bond``, etc. You can refer to QuantLib document to find what class QuantLib has.

The *LibraryName* is other library that you will use in instrument class source file, like numpy and pandas.

The *DefaultInstrumentType* is a string to mark your new instrument class. It can be any string or blank.

Build_Attr.xlsx
^^^^^^^^^^^^^^^

``Build_Attr.xlsx`` is the file used to tell RiskQuantLib what kind of attributes you need, when analysising your data, and which class these attributes belong to. After calling ``build.py``, any attributes specified here will be registered and can be used with ``set`` function. This file looks like:

+--------------+-------------------+----------------+
| SecurityType |    AttrName       |    AttrType    |
+==============+===================+================+
|     fund     | yourAttribute     |     number     |
+--------------+-------------------+----------------+
|     stock    | anotherAttribute  |     string     |
+--------------+-------------------+----------------+
|     ...      |         ...       |       ...      |
+--------------+-------------------+----------------+

The *SecurityType* column can accept key word like: ``Any``, ``Fund``, ``Stock``, ``Bond``, ``Repo``, etc. However,

Any instrument you specify in *Build_Instrument.xlsx* can also be used here. If you already create a class named ``treasureBond`` in *Build_Instrument.xlsx*, then here you can fill ``treasureBond`` in *SecurityType* column.

The *AttrType* column can accept key word of ``Number``, ``String``, ``Any``, ``Series``, or any other string that you think it could be a type. It tells RiskQuantLib what kind of data will be stored by this attribute. If you add 'sellPrice' to ``Stock``, this should be a ``Number`` attribute, while an attribute like 'issuerName' is a ``String`` kind. ``Series`` is used if it's an attribute like 'sellPriceOfPastSixMonth'. If you specify a kind that is never used before, it will be created as a type class, located in ``RiskQuantLib.Property``.

build.py
^^^^^^^^

``build.py`` is used to generate python source code automatically. After you specify what kind of class you want to create, how it inherit from other class in *Build_Instrument.xlsx*, and what attributes these class should have in *Build_Attr.xlsx*, you can call *build.py* by terminal:
::

   python build.py

**Notice: If you do not use control comment syntax in Src, this build.py will only need to be excuted once, at the begin of your project. Do not build your project every time you run main.py, it is not necessary. But if you use control comment in Src, you can use the following command so that the build action will be triggered every time you make change to Src directory:**
::

   python build.py -a

If your project is under development, it will be useful to use ``debug`` mode. With this mode, the python source code in ``Src`` will not be directly inserted into target file, it will be bound dynamically into target file. By this way, the break point in file under ``Src`` will start to effect, you can debug it directly. Surely, the ``auto build`` mode can be run at the same time, it will automatically build the whole project every time you make a change. To build project in automatically debug mode, run:
::

   python build.py -a -d

main.py
^^^^^^^

``main.py`` is entrance of your project. You can start your coding here, by:
::

   from RiskQuantLib.module import *

Then you can use the class directly by:
::

   bondA = treasureBond('firstBond','nameOfFirstBond')

You can also set attributes directly by:
::

   stockA = stock('firstStock','nameOfFirstStock')
   stockA.setAnotherAttribute('valueOfAnotherAttribute')

For more information about the ``Instrument``, we will introduce it in next chapter.