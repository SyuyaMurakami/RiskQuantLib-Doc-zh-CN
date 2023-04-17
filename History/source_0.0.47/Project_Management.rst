Project Management
====================

.. toctree::
   :maxdepth: 4

One of the most important use of RiskQuantLib is project management. It's necessary to introduce a conception named template. Here in RiskQuantLib, you can define and build a project before you know your data. You can also code some data process logic in instrument class file. If we stop right in this step, close all files, and save this project, it is called template.

In short, *a template is a data process project which is suitable for more than one situations*, it could deal with data regardless of data paterns.

However, you can just save any project in case you would use it some day later.

Save Project As Template
^^^^^^^^^^^^^^^^^^^^^^^^

You can save a project as a RiskQuantLib template by using terminal command:
::

   saveRQL your_project_path

After run this command, a '.zip' file will be under your project path, and meanwhile, a template will be added into RiskQuantLib template library.

Show All Template
^^^^^^^^^^^^^^^^^

You can show all saved template in your library by using terminal command:
::

   listRQL

After run this command, you will see all templates in your library.

Unpack Template
^^^^^^^^^^^^^^^

You can unpack a project template by using terminal command:
::

   tplRQL template_name your_new_project_path

After run this command, you will unpack the contents of this template into ``your_new_project_path``, and you can use them as the fundation of another project.

Delete Template
^^^^^^^^^^^^^^^

If you want to delete some template, use terminal command:
::

   delRQL template_name

Or if you want to delete all template, you should use:
::

   clearRQL

**Notice: Use this command carefully, cause it can not be un-done.**
