与万得配合使用
====================

.. toctree::
   :maxdepth: 4

RiskQuantLib提供了二次封装的数据接口，以供使用者更加方便地利用万得金融终端的 ``python API`` 进行数据分析。这样的接口是利用另一个独立的python库进行的，名字叫做 ``windget`` , ``windget`` 提供了以 ``get`` 作为关键字的函数族，并封装了万得API提供的绝大部分函数接口。

安装和导入windget
^^^^^^^^^^^^^^^^^^^^^^

windget是独立的python库，可以独立于RiskQuantLib运行，但在此之前，需要使用 ``pip`` 命令进行安装:
::

   pip install windget

**如果你是RiskQuantLib的用户，你无需进行这一步操作，RiskQuantLib在** ``0.0.45`` **版本之后已经集成了** ``windget`` 。 **在进行更新和安装的时候，RiskQuantLib会自动对** ``windget`` **进行安装和更新。**

如果你安装了RiskQuantLib或者windget，可以在源代码中使用以下命令进行导入：
::

   from windget import *

或者在RiskQuantLib的工程主文件中使用（在 ``0.2.0`` 之后的版本已经被废弃）：
::

   from RiskQuantLib.DataInputAPI.getDataFromWind import *

get函数族
^^^^^^^^^^^^^^^^^^^^^^

同模板类和模板列表类中的函数族一样， ``get`` 函数族指的是一类名称以 ``get`` 开头的函数，他们负责从万得的python API中获取数据。这类函数的名称是 ``get`` 加上获取数据的字段的英文关键字。比如想要获取收盘价，函数的名称应该为 ``getClose`` ，你可以通过以下命令来获取收盘价：
::

   getClose('000001.SZ',"tradeDate=20200606")

你可以将 ``Series`` 添加在任何字段的名称之后，来获取当前字段的时间序列。比如要获取股票的收盘价时间序列，可以使用：
::

   getCloseSeries('000001.SZ',"20200606","20200706")

在这种情况下， ``Series`` 被称为函数族尾缀，注意， ``Series`` 并不是唯一的函数族尾缀，其他可用的尾缀包括 ``Wsi`` ， ``Wst`` ， ``Wsee`` ， ``Wses`` ， ``Wsq`` 。他们对应的意义分别如下：
::

   getCloseWsi('000001.SZ',"2022-05-20 09:00:00", "2022-05-20 11:24:00") # 获取收盘价分钟序列
   getSecCloseTsWavGWsee('a001010100000000',"tradeDate=20220521;DynamicTime=1") # 获取收盘价(总股本加权平均)板块多维
   getSecCloseTsWavGWses('a001010100000000', "2022-04-22", "2022-05-21") # 获取收盘价(总股本加权平均)板块序列
   getRtLastWsq('000001.SZ') # 获取现价实时行情
   getLastWst('000001.SZ',"2022-05-20 09:00:00", "2022-05-20 11:24:00") # 获取最新价日内跳价

**注意：所有的字段之后都可以增加** ``Series`` **作为尾缀，但其他的尾缀不一定适用。**

**注意：以上例子中的股票字符串均可以使用列表或者pandas.Series进行替代，比如：**
::

   getClose(['000001.SZ','000004.SZ'],"tradeDate=20200606")
   getCloseSeries(['000001.SZ','000004.SZ'],"20200606","20200706")


模糊查找字段
^^^^^^^^^^^^^^^^^^^^^^^

如果你并不知道万得函数的字段名称，可以使用模糊查找功能进行匹配。在pycharm中，同时按下 ``control`` + ``shift`` + ``F`` 可以调出查询框，你可以输入中文的字段名称来查询 ``get`` 函数族中符合的函数。此功能的使用必须依赖于RiskQuantLib工程，且适用于 ``0.2.0`` 之前的版本。只有在RiskQuantLib工程中才可以用中文进行字段查询。

Pycharm插件
^^^^^^^^^^^^^^^^^^^^^^^

windget提供了一个pycharm插件，进行代码的自动补全，以此弥补用户无法准确获取万得函数字段的问题。在pycharm的 ``File-Setting-Plugin`` 中的插件市场中，搜索windget并安装，重启pycharm即可使用windget的pycharm插件。此插件不需要依赖于RiskQuantLib工程，可以在任何pycharm项目中使用。

