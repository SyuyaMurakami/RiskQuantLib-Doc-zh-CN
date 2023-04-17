与Github配合使用
====================

.. toctree::
   :maxdepth: 4

在 **模板管理** 中，我们了解到RiskQuantLib可以将工程保存为模板，然后供下一次进行数据分析时初始化项目使用。这样的使用方式仅仅局限于一台电脑。如果你在团队中试图使用RiskQuantLib，你或许需要向团队成员分享代码和模板工程，RiskQuantLib通过Github来实现这一功能，通过预置的命令，RiskQuantLib可以获取Github上所有的公开项目，并将这些项目保存为本地计算机上的工程模板。

从Github获取项目
^^^^^^^^^^^^^^^^^^^^^^

在终端对话框中，使用以下命令来获取Github项目，以下我们将使用 `hub项目 <https://github.com/github/hub>`_ 作为例子:
::

   getRQL https://github.com/github/hub

运行成功后，会显示如下信息：
::

   Successfully Installed hub

你也可以通过终端命令中的 ``listRQL`` 来查看已经保存为模板工程的项目。

模糊查找项目
^^^^^^^^^^^^^^^^^^^^^^^

如果你并不知道Github项目的确切网址，你可以通过大致的关键字来模糊查找，在 ``getRQL`` 命令后输入关键字，可以触发模糊查询功能。
::

   getRQL hub

运行此命令会使得RiskQuantLib在Github中查询所有相关的项目，并按照相关性和项目标星数进行排列。你可以看到如下界面：
::

   Total repositories:  266905
   Show Top Github Repositories:
   ------------------------------
   0 -> ohmyzsh
   1 -> HelloGitHub
   2 -> GitHub-Chinese-Top-Charts
   3 -> rclone
   4 -> CodeHub
   5 -> hub
   6 -> hubot
   Choose One From Above:

输入对应项目前面的数字，可以将此项目下载到本地作为工程，在本例子中，我们选择 ``HelloGitHub`` ，因此应该输入1，之后会看到如下界面：
::

   Successfully Installed HelloGitHub

之后你可以使用已经保存的模板工程来初始化新的工程。