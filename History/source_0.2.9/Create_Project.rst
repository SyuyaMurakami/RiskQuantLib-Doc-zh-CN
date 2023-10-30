Create A New Project
====================

.. toctree::
   :maxdepth: 4

RiskQuantLib is based on project. For any data analysis mission, you should create at least one project to do it.

To create a new RiskQuantLib project, you can use ``newRQL`` command in terminal as follows:
::

   newRQL your_project_path

**Notice: If there is already a project in the target dictionary, all old files will be deleted, and replaced with brand new project files.**

After this command, your dictionary should looks like:
::

   --your_project_path
     --RiskQuantLib
     --Src
     --build.py
     --main.py
     --Build_Attr.xlsx
     --Build_Instrument.xlsx

Then you can build it to suit your mission, we will explain it in next chapter.