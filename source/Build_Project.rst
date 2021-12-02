Build Your Project
====================

.. toctree::
   :maxdepth: 4

After create a project, you should build it to suit for your mission. By ``Build``, RiskQuantLib will generate python source code automatically. It is the core conception of RiskQuantLib. Before moving on, let specify some definition in RiskQuantLib:

``Build`` means generate source code automatically.

``Instrument`` means any class you will use in your mission. For financial analysis, ``Instrument`` refers to stock, bond or other security type, or like interest rate or company, etc.

Then we start to build our first project. The critical thing is to tell RiskQuantLib how to build it. This is done by specify key word in ``Build_Instrument.xlsx`` and ``Build_Attr.xlsx``, which you can find in your project root dictionary once you create a project. The content of building will be written into ``RiskQuantLib.Set``. 

In last chapter we create a project, it looks like:
::

   --your_project_path
     --RiskQuantLib
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
     --Build
     --Company
     --CompanyList
     --DataInputAPI
     --Index
     --IndexList
     --InterestRate
     --Model
     --Operation
     --Property
     --Security
     --SecurityList
     --Set
     --Tool
     --__init__.py
     --Module.py

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

The *InstrumentName* column can be defined by your will, depending on your data. RiskQuantLib will create instrument class source file ``treasureBond.py`` under ``RiskQuantLib.Security.Bond.TreasureBond``, and create list class source file ``treasureBondList.py`` under  ``RiskQuantLib.SecurityList.BondList.TreasureBondList`` after your run ``python build.py``.

The *ParentRQLClassName* means this new instrument will inherit from this RiskQuantLib class. It can accept key word like: ``Any``, ``Fund``, ``Stock``, ``Bond``, ``Repo``, etc. However,

**Notice: The first letter must be capitalized in ParentRQLClassName column.**

**Notice: You can use the instrumentName you specified in last row to fill ParentRQLClassName column, like:**

+--------------+------------------+-----------------------+------------+---------------------+
|InstrumentName|ParentRQLClassName|ParentQuantLibClassName|LibararyName|DefaultInstrumentType|
+==============+==================+=======================+============+=====================+
| treasureBond |       Bond       |                       |    numpy   |   Treasure Bond     |
+--------------+------------------+-----------------------+------------+---------------------+
|chinaTreaBond |   TreasureBond   |                       |            |     Panda Bond      |
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
|     Fund     | yourAttribute     |     Number     |
+--------------+-------------------+----------------+
|     Stock    | anotherAttribute  |     String     |
+--------------+-------------------+----------------+
|     ...      |         ...       |       ...      |
+--------------+-------------------+----------------+

The *SecurityType* column can accept key word like: ``Any``, ``Fund``, ``Stock``, ``Bond``, ``Repo``, etc. However,

**Notice: The first letter must be capitalized in SecurityType column.**

Any instrument you specify in *Build_Instrument.xlsx* can also be used here. If you already create a class named ``treasureBond`` in *Build_Instrument.xlsx*, then here you can fill ``TreasureBond`` in *SecurityType* column.

The *AttrType* column can accept key word of ``Number``, ``String``, ``Any``, ``Series``, or any other string that you think it could be a type. It tells RiskQuantLib what kind of data will be stored by this attribute. If you add 'sellPrice' to ``Stock``, this should be a ``Number`` attribute, while an attribute like 'issuerName' is a ``String`` kind. ``Series`` is used if it's an attribute like 'sellPriceOfPastSixMonth'. If you specify a kind that is never used before, it will be created as a type class, located in ``RiskQuantLib.Property``.

build.py
^^^^^^^^

``build.py`` is used to generate python source code automatically. After you specify what kind of class you want to create, how it inherit from other class in *Build_Instrument.xlsx*, and what attributes these class should have in *Build_Attr.xlsx*, you can call *build.py* by terminal:
::

   python build.py

**Notice: Usually this build.py will only be excuted once, at the begin of your project. Do not build your project every time you run main.py, it is not necessary.**

main.py
^^^^^^^

``main.py`` is entrance of your project. You can start your coding here, by:
::

   from RiskQuantLib.Module import *

Then you can use the class directly by:
::

   bondA = treasureBond('firstBond','nameOfFirstBond')

You can also set attributes directly by:
::

   stockA = stock('firstStock','nameOfFirstStock')
   stockA.setAnotherAttribute('valueOfAnotherAttribute')

For more information about the ``Instrument``, we will introduce it in next chapter.