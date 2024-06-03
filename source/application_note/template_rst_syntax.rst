=======
RST语法
=======

.. contents::
   :local:
   :depth: 2

一级标题x
==========

二级标题
--------

三级标题
~~~~~~~~

四级标题
^^^^^^^^

五级标题
********

.. note:: 
   - 章节头部由标题名称和符号下线组合创建，推荐统一使用上述样式。
   - 各级标题的符号线段依次使用 ``= - ~ ^ *`` 表示，长度应不小于标题长度。
     
     - 一级标题 ``========``
     - 二级标题 ``--------``
     - 三级标题 ``~~~~~~~~``
     - 四级标题 ``^^^^^^^^``
     - 五级标题 ``********``
  
   - 每份rst文档内部文档名使用上下线组合的标题样式。
   - HTML只支持最多6级标题，一般已足够使用。
   - 标题后续可以直接接正文，不用单独空一行。

RST架构
=======
目录
----
使用 ``.. contents::`` 指令添加目录。

参数释义
   :local: 只显示当前文件而非整个文档的目录。
   :depth: 指定目录的深度，默认情况下，除文档名外的所有标题级别都被包括在内。

::

   .. contents::
      :local:
      :depth: 2

.. note:: 
   - 目录的位置建议放到文档顶部，置于文档名之后，这样可以生成整篇文档的架构。
   - 目录样式已设置悬浮在网页右侧侧边栏显示。

目录树
------
使用 ``.. toctree::`` 指令定义目录树。

参数释义
   :maxdepth: 定义目录深度
   :hidden: 目录仍会存在于Sphinx的文档结构中，但是不会在当前指令位置显示
   :numbered: 在HTML中为章节编号
   :titlesonly: 目录里仅出现文档的标题，不出现文中其他同等级的标题行

::

   .. toctree::
      :maxdepth: 1
      :hidden:  

      template_rst_syntax
      RST syntax template <template_rst_syntax>
      Glossary <../glossary/glossary>
      Platform <../platform/text_cn/README>

.. admonition:: 上述示例中toctree添加文件的几种方式说明
   
   - ``rst文件名`` 直接引用当前路径下文件，无需添加格式后缀，显示为文件内部文档名，而非文件名。
   - ``自定义标题 <rst文件名>`` 添加自定义显示名称。
   - ``自定义标题 <文件路径>`` 添加绝对或者相对路径，引用不同文件。
   - ``自定义标题 <其他toctree>`` 添加toctree嵌套，同样生效。
   - 原则上所有文件都需出现在某个toctree指令里，否则Sphinx会报出警告，因为该文件没有通过标准导航。

RST基础
=======
段落
----
段落是rst文件的基本模块，正文的段落用空行分隔，一律顶格书写，行首不得有空格。

如果没有空行。
下一行的段落将紧跟上文，被视为连续文本。

缩进
----
缩进的段落
   被认为是前面文本的引文。
   
   保持相同的缩进为同一层级。
缩进取消，这段文字为新的段落。（不用空行也可以被识别为新的段落）

.. note:: 
   - 在rst中，缩进非常重要，通过对缩进和空行的把控，可以调整文档的布局和格式。
   - rst对具体缩进量的要求比较宽松，只有一个最小缩进要求，为2个空格。
     
     - 本文中为了与指令字符的对齐，使用了3个空格缩进，每个层级缩进3个空格。
     - 每当缩进量改变时，段落内容就进入了新的缩进层级。
     - 请保持（相邻的）同级内容使用同样的缩进量。

块/block
----------
以缩进为级别，整个文档可以看作不同级别的缩进结构内容的排列与嵌套，这些不同的结构可以是做一个块（block）。

.. _文字块:
文字块
~~~~~~
使用 ``::`` 开头，保持同一缩进即可。

::

   这是一个文字块
   文字块中的内容与实际显示效果一样

   保持相同缩进即可

这是新的一行（可以有空行，也可以不用空行，因为顶格没有缩进，该段被视为新的段落）。

文字块示例
^^^^^^^^^^^

.. highlight:: rst

SDK目录

::

   └── SDK                       SDK开发包
       ├── bin                   应用程序链接的二进制文件
       ├── bsp                   板级和硬件相关的文件
       ├── doc                   文档
       ├── download_images       快速启动的下载images
       ├── include               提供接口定义的头文件
       ├── samples               配置好可直接使用的Keil实例工程
       ├── subsys                上层和硬件无关的软件协议
       └── tools                 工具集

ble_peripheral示例工程

::

   └── Project: peripheral
       └── secure_only_app             
           └── Device                   存放启动代码
               ├── startup_rtl.c                
               └── system_rtl.c      
           ├── CMSIS                    存放启动代码
           ├── CMSIS                    存放CMSIS文件
           ├── contribute               如何参与项目，贡献与投稿的说明
           ├── CMSE Library             Non-secure callable lib
           ├── Lib                      应用程序使用的所有二进制文件
           ├── Peripheral               应用使用的所有外设驱动和模块代码
           ├── Profile                  应用使用的BLE profiles或者服务
           └── APP                      ble peripheral应用的实现
               ├── sncs.c
               ├── app_task.c        
               ├── main.c  
               └── peripheral_app.c

.. highlight:: none

| 文件路径统一使用以下形式：
| ``BEE4-SDK-Vx.x.x\tools\BeeMPTool_vx.x.x.x\BeeMPTool\tools\flashmap.ini``

.. note:: 文字块中的中文标点以及如上示例中一些特殊符号，可能在生成在线文档时会出现报错，此时可以用 ``.. highlight:: rst`` ``.. highlight:: none`` 两条指令将文字块包含，用来设定代码块的默认语法高亮方式为rst，即可消除告警。 

注释
----
使用 ``..`` 开头，后续添加注释内容。

.. 
   注释内容在网页上不会显示
   可以分行标注

   保持相同缩进即可
注释文本要求与\ :ref:`文字块`\ 相同，缩进取消即为新的段落。

.. note:: 注释文本紧跟符号 ``..`` ，文字块文本与 ``::`` 有一个空白行。

行块
~~~~~~
| 行块使用 ``|`` 开头，后续文本与之保持一个空格。
| 这样可以保证输出的正文和原文件的排列一致。
| 与正文段落作区分，减少段落之间的空行。

测试块
~~~~~~
测试块是交互式的Python会话，以 ``>>>`` 开始，一个空行结束。

>>> print "This is a doctest block."
This is a doctest block.

.. _分隔线:
分隔线
------
分隔线使用四个 ``----``，效果如下。

----

.. note:: 
   - 分隔线与上下内容之间需要有空行区分。
   - 分隔线不能紧贴在大纲标题之前或之后，也不能放在文档最开头。
   - 两个分隔线不能紧贴，必须有除了空行之外的内容。

行内标记
========
rst通过行内标记对文本定义不同的效果，以下具体说明。

符号标记
--------
最基础的符号标记有斜体、加粗、code这三种。

| *斜体*   ``*text*`` 
| **加粗**   ``**text**`` 
| ``code``   ````text```` 
   
.. note:: 
   - 行内标记的 :red-text:`文本` 不可以用空格开始或结尾，如 ``*text *`` ``* text*`` 都无法实现效果。
   - 行内标记的 :red-text:`符号` 前后都必须有空格，有以下两个方式实现：
   
     - 方式一：直接在符号指令前后文本添加一个空格，此方式较简单，但是在文本中空格会显示出来，特别是中文文档的显示上比较明显。
     - 方式二：使用转义字符反斜杠\ ``\``\ ，它可以用于取消一些特殊字符的特殊含义，使其按照普通字符处理。如果需要在连续文本中使用指令，可以用\ ``反斜杠+空格``\ 的方式实现转义，如此处说明文本不显示空格。
  
CSS标记
-------
基础的行内标记不能嵌套，无法简单地实现“倾斜且加粗”的功能，具体可以借助HTML/CSS功能，当前在线文档已设置了如下两个简单的自定义标记。

| :red-text:`红色`    ``:red-text:`` 
| :bolditalic:`粗斜体`    ``:bolditalic:`` 

.. note:: 该指令已通过CSS自定义，后续如果需要实现不同的效果，可自行查阅CSS设置并添加。

指令标记
---------
还有一些指令可以实现不同的行内标记效果。

.. centered:: 创建居中加粗文本行

居中加粗指令 ``.. centered::`` 

.. rubric:: 文档标题

文档标题指令 ``.. rubric::`` 

.. note:: ``.. rubric::`` 指令用来创建文档标题，但是该标题不出现在文档的目录结构，常用作\ :ref:`脚注`\ 说明。

列表
====
有序列表
--------

1. 这是一个有序列表
2. 这是父列表
  
   1. 一级嵌套
   2. 列表子项
  
      #. 二级嵌套
      #. 列表子项
  
#. 父列表继续

无序列表
--------

* 这是一个无序列表
* 这是父列表
  
  * 一级嵌套
  * 列表子项
  
    * 二级嵌套
    * 列表子项
  
* 父列表继续
  
.. note:: 
   - 列表使用\ :menuselection:`数字.`\ /\ :menuselection:`* - +`\ 后续跟一个\ :menuselection:`空格`\ 表示，也可以使用符号\ :menuselection:`#.`\ 自动加序号。
   - 列表可以嵌套，但是需跟父列表使用\ :red-text:`空行`\ 分隔，同时注意保持\ :red-text:`缩进`\ 。
   - :menuselection:`空行`：先检查一级嵌套与父列表存在空行，为两个段落。
   - :menuselection:`缩进`：再检查有无缩进，此处一级嵌套编号与父列表文本对齐没有缩进，在网页上整个列表呈现为一个整体，多级列表之间没有间隔。

常见问题
~~~~~~~~
在列表的呈现上，会遇到一些常见的问题，几乎都与空行和缩进有关，在此作说明。

问题一
^^^^^^^

* 这是一个
* 父列表
  * 一级嵌套
  * 列表子项
    * 二级嵌套
    * 列表子项
 
.. note::
   - :menuselection:`空行`：先检查一级嵌套与父列表没有空行，是连续的段落。
   - :menuselection:`缩进`：再检查有无缩进，此处一级嵌套编号与父列表文本没有缩进，对齐的段落被解析成连续的文本,\ :menuselection:`*`\ 标记紧跟父列表文本，编号定义没有生效，所以在网页上直接显示符号。
   - 二级嵌套编号与父列表存在缩进，被解析为父列表的一级嵌套。

问题二
^^^^^^^

* 这是一个
* 父列表
   * 一级嵌套
   * 列表子项
      * 二级嵌套
      * 列表子项

.. note::
   - :menuselection:`空行`：先检查一级嵌套与父列表没有空行，是连续的段落。
   - :menuselection:`缩进`：再检查有无缩进，此处一级嵌套编号与父列表文本存在缩进，缩进的段落被解析为上文的引文而单独一行（编号前面没有字符，定义生效），所以在网页上一级嵌套生效并被解读为父列表的引文，同理，二级嵌套被解析为列表子项的引文。

问题三
^^^^^^^

* 这是一个
* 父列表
  
   * 一级嵌套
   * 列表子项
  
      * 二级嵌套
      * 列表子项
  
.. note::
   - :menuselection:`空行`：先检查一级嵌套与父列表存在空行，为两个段落。
   - :menuselection:`缩进`：再检查有无缩进，此处一级嵌套编号与父列表文本存在缩进，所以在网页上嵌套列表与父列表呈现两个段落，中间有间隔。

定义列表
--------
定义列表可以用于名词解释。

功能
   这是一个功能描述。
 
注意事项
   这是注意事项描述。

字段列表
--------
字段列表可以用于函数说明。

:Function Name: ADC_DeInit
:Function Prototype: void ADC_DeInit(void)
:Function Description: Function Description
:Input Parameter: ADCx: Base Address of ADCx pointing to the ADC module selected
:Output Parameter: None  
:Return Value: None
:Prerequisite: None
:Functions Called: RCC_PeriphClockCmd Function

参数列表
-------- 
参数列表可以用于参数说明。

-V           简单参数
-h, --help   以逗号分隔的参数
-c docname   带空格的参数
--file=flag  带等号的参数
--an-longer-arg  有点长的参数
--an-arg-that-is-very-long
             过长的参数的说明文字，可以从第二行开始，注意对齐
/S           斜线开头的参数

表格
====
网格表格
--------
网格表格使用画图的方式绘制表格，做以下说明。

* 网格表中使用的符号如下：  

  * 用 :menuselection:`-` 来分隔行。
  * 用 :menuselection:`=` 来分隔表头和表体行。
  * 用 :menuselection:`|` 来分隔列。
  * 用 :menuselection:`+` 来表示行和列相交的节点。
    
* 单元格内部支持段落、强调、链接等语法。
* 表格的引用通过\ ``:ref:``\ 指令实现，这是\ :ref:`表格-标准网格表`\ ，也可以\ :ref:`自定义表格名称<表格-标准网格表>` 。

参数释义
   :table: 表格标题
   :widths: 表格相对宽度
   :align: 表格对齐方式（非单元格内容对齐方式）
   :name: 表格标签，用于表格跳转，格式：表格-表格标题

.. table:: 标准网格表
   :widths: 25 25 50
   :align: center
   :name: 表格-标准网格表

   +------------+-----------+-------------------------------+
   | Heading    | Heading   | Heading                       |
   +============+===========+===============================+
   | *Contents* | Contents  | **Contents**                  |
   +------------+-----------+-------------------------------+
   | *Contents* | Contents  | Contents (http://example.com) |
   +------------+-----------+-------------------------------+
  
.. table:: 合并单元格
   :widths: 25 25 50
   :align: center 
   :name: 表格-合并单元格

   +----------------+----------------+----------------+ 
   | Heading        | Heading        | Heading        | 
   +----------------+----------------+----------------+ 
   | Heading        | Heading                         | 
   +================+================+================+ 
   | Contents       | Contents       | Contents       | 
   +----------------+----------------+----------------+ 
   |                | Cells may span columns.         |
   | Contents       |                                 | 
   |                | Cells may span columns.         | 
   +----------------+----------------+----------------+ 
   | Contents       | Cells may      | - Contents     | 
   +----------------+ span           | - Contents     | 
   | Contents       | columns.       | - Contents     | 
   +----------------+----------------+----------------+

.. note:: 
   - 网格表的绘制必须保持每个单元格长度（主要是字符）一致，否则无法解析。
   - rst中涉及到单元格合并的表格建议使用网格表绘制。
   - 当前文档表格单元格内容均设置左对齐，目前暂时无法对某一列单独做调整，需要自定义CSS功能。

AsciiTable工具
~~~~~~~~~~~~~~~
由于中英文字符的不同，中文表格的绘制比较复杂，可以使用表格生成工具\ `AsciiTable <https://www.tablesgenerator.com/text_tables>`_\ ，直接将表格转换成rst能识别的语法。

以下是\ :ref:`图片-AsciiTable工具说明`\ 以及\ :ref:`表格-AsciiTable转换示例`\ 。

.. figure:: ../figures/AsciiTable.*
   :align: center
   :name: 图片-AsciiTable工具说明

   AsciiTable工具说明

.. table:: AsciiTable转换示例
   :widths: 50 50 50 50
   :align: center
   :name: 表格-AsciiTable转换示例

   +----------+----------+----------+----------+
   |          |          |          |          |
   | 标题1    | 标题2    | 标题3    | 标题4    |
   +==========+==========+==========+==========+
   |          |                     |          |
   | 内容     | 内容                | 内容     |
   +----------+----------+----------+----------+
   |          |          |          |          |
   | 内容     | 内容     | 内容     | 内容     |
   +----------+----------+          |          |
   |          |          |          |          |
   | 内容     | 内容     |          |          |
   +----------+----------+----------+----------+
  
CSV表格
----------
参数释义
   :csv-table: 表格标题
   :header: 表格标题行
   :stub-columns: 指定首列做标题
   :widths: 表格相对宽度
   :align: 表格对齐方式
   :name: 表格标签

.. csv-table:: CSV表格举例
   :header: 标题1, 标题2, 标题3
   :stub-columns: 1
   :widths: 20 80 80
   :align: center
   :name: 表格-CSV表格举例
  
   说明1, Xxx, Xxx
   说明2, Xxx, Xxx
   说明3, Xxx, Xxx
   说明4, "Hello, world!", """Hello, world!"""
  
.. note:: 
   -  标题行和数据行用英文逗号 ``,`` 进行分格，若单元格内容本身包含逗号，可用引号包含，如上示例 ``"Hello, world!"`` 
   - 如果数据中本身包含引号，那么需要使用\ :red-text:`两个`\ 连续的引引号来表示一个，如上示例 ``"""Hello, world!"""`` 
   - CSV表格也可以添加 ``:file:`` 参数引 ``.csv`` 文件直接插入表格。

Listtable表格
--------------
参数释义
   :list-table: 表标题
   :header-rows: 标题行数量（这边设置2级标题）
   :stub-columns: 指定首列做标题
   :widths: 表格相对宽度
   :align: 表格对齐方式
   :name: 表格标签

.. list-table:: Listtable表格举例
   :widths: 20 80 80
   :header-rows: 2
   :stub-columns: 1
   :align: center
   :name: 表格-Listtable表格举例

   * - 标题1
     - 标题2
     - 标题3
   * - 次标题1
     - 次标题2
     - 次标题3
   * - Xxx
     - Xxx
     - Xxx
   * - Xxx
     - Hello, world!
     - "Hello, world!"

.. note:: 
   线性表格内容无需用引号包含。

水平列表
----------
使用 ``.. hlist::`` 生成， ``:columns:`` 参数设置列数。

.. hlist::
   :columns: 4

   * 说明1
   * 说明2
   * 说明3
   * 说明4
   * 说明5
   * 说明6
   * 说明7
   * 说明8
  
.. note:: 
   该指令通常用于创建紧凑的列表，如一节结束时放置相关主题列表或者显示一系列小项列表以便紧凑、易读。

图片
====
| 使用 ``.. figure::`` 或者 ``.. image::`` 指令添加图片。
| 他们的主要区别在于：使用figure对图片进行更多控制和格式化选项，而image则更简单直接，以下参数选项均可选。

参数释义
   :figure: 图片路径，支持使用 ``.*`` 扩展名自动选择文件： ``.pgn`` ``.jpg`` ``.pdf`` 
   :height: 设置图片的高度，可以为具体像素px或者百分比
   :width: 设置图片的宽度，可以为具体像素px或者百分比
   :scale: 设置图片缩放比例，可以为数值或者百分比
   :align: 图片对齐方式：center/left/right
   :alt: 图片加载失败时的替代文本
   :name: 图片标签，用于图片跳转，注意与图片标题做区分，使用figure指令，图片标题在下方添加
   :target: 点击图片时跳转的链接，可以是绝对地址、相对地址，或者rst的超链接。

图片的引用通过 ``:ref:`` 指令实现， 这是\ :ref:`图片-绿灯`\ ，也可以\ :ref:`自定义图片名称<图片-绿灯>` 。

.. figure:: ../figures/red.*
   :align: center
   :scale: 100%
   :alt: 这里应该是红灯图片
   :name: 图片-红灯

   在这里添加图片标题

.. figure:: ../figures/green.*
   :align: center
   :scale: 100%
   :name: 图片-绿灯

   绿灯

.. image:: ../figures/yellow.*

.. note:: 
   - image指令无法添加图片标题，其余参数和figure一致。
   - 每份文档的图片均在对应文档的 :file:`../figures` 文件夹下，Visio原图保存在 :file:`../vsd` 文件夹下，中英文文档共用一份。

词汇表
======
使用 ``.. glossary::`` 指令生成词汇表，请参阅 :doc:`Glossary <../../glossary/glossary>` 以了解所有专业术语。

::

   .. glossary::
      :sorted:
      
      BLE
         Bluetooth Low Energyn

.. note::
   - 词汇表的引用： :term:`BLE` 
   - ``:sorted:`` 参数选项用于词汇表自动排序。

代码
====
| 使用 ``.. code-block::`` 指令生成代码块。

参数释义
   :code-block: code语言
   :emphasize-lines: 高亮特定行
   :linenos: 显示行号，默认显示在左侧单独一列
   :caption: 添加代码块标题
   :name: 代码引用标签

.. code-block:: c
   :emphasize-lines: 3,5
   :linenos:
   :caption: 高亮代码块示例
   :name: 代码-高亮代码块示例
  
   /* This is a sample code */
   #include<stdio.h>
   int main()
   {
      printf("%s\n","aaaa");
      return 0;
   }

.. note:: 
   - 文本代码块示例中所有参数选项都不是必须的。
   - 代码块的引用需name，caption搭配，使用 ``:name:`` 为代码块添加引用标签，使用 ``:caption:`` 添加代码块标题，同时在引用时显示链接为 :ref:`代码-高亮代码块示例`

.. _内联标记:
内联标记
========

角色
----
| “角色”（role）是内联标记的一种形式，用来定义特殊的文本格式或者链接，以下是一些\ :ref:`常用角色说明`\ 。
| 角色的语法为 ``:role:`content``` 

.. csv-table:: 常用角色
   :header: 角色,说明,效果
   :widths: 20 80 80
   :align: center
   :name: 常用角色说明
  
   ``:math:``, 用于数学表达式, :math:`(a + b)^2 = a^2 + 2ab + b^2`
   ``:file:``, 用于标识文件, :file:`../../cn/conf.py`
   ``:doc:``, 用于文件跳转, :doc:`Glossary <../../glossary/glossary>`
   ``:term:``, 用于链接术语表, :term:`ADC`
   ``:guilabel:``, 用于标识图形界面元素, :guilabel:`OK`
   ``:menuselection:``, 用于标识菜单条目, :menuselection:`File --> New --> Project`
   ``:kbd:``, 用于键盘输入指令, :kbd:`Ctrl+C`
   ``:sup:``, 用于上标, Bluetooth\ :sup:`®`
   ``:sub:``, 用于下标, H\ :sub:`2`\ O
   ``:mod:``, 用于引用模块, :mod:`numpy`
   ``:func:``, 用于引用函数, :func:`my_function`
   ``:class:``, 用于引用类, :class:`list`

交叉引用
---------
| rst中广泛的交叉引用可以通过 ``.. _标签:`` 与 ``:ref:`` 的配合实现。
| 使用 ``.. _标签:`` 在目标地址处添加标签，使用 ``:ref:`标签``` 在需要引用时添加跳转。

这是 :ref:`内联标记`，也可以 :ref:`自定义名称 <内联标记>`。

.. note:: 
   - 整份在线文档中的标签名必须要唯一，否则可能会引发错误或者警告，可以采用 ``.. _文档名-标签:`` 以作区分。
   - ``:ref:`` 适用于广泛的跳转，rst中为一些特定的跳转提供了更为简单的指令。
  
     - rst文档之间的跳转，可以使用文档指令 ``:Doc:`文档名/路径``` 直接跳转。
     - 图片、表格、代码等跳转，可以在 ``.. table::`` ``.. figure::`` ``.. code-block::`` 指令处添加 ``:name:`` 选项参数说明，即可直接使用 ``:ref:`` 跳转。

文本引用
--------
使用 ``.. include::`` 指令引用内容，以下是测试文本。

.. include_开始标签

1. 这是第1行
2. **这是第2行**
3. 这是第3行
4. 这是第4行
   
.. include_结束标签 

.. note:: 
   - ``.. include::`` 指令直接引用文件的\ :red-text:`内容`，而非指向该文件的链接。
   - 该指令会对内容共进行解析，如果导入文件中包含rst的语法，都会被正确的解析和渲染。
   - 用于避免重复内容，维护大型文档，团队协作等情况非常有帮助。

行数引用
~~~~~~~~~~~
参数释义
   :include: 引用文件，可以是当前文件，也可以添加文件路径，必须注明文件类型
   :start-line: 行引用开始，不包含当前行内容
   :end-line: 行引用结束，包含当前行内容

以下是行引用的内容：

.. include:: template_rst_syntax.rst
   :start-line: 681
   :end-line: 684

.. note:: 行数引用有一定的使用限制，会由于内容的修改引起行数变化，需要及时更新行数。

节点引用
~~~~~~~~~~~
参数释义
   :include: 引用文件，可以是当前文件，也可以添加文件路径，必须注明文件类型
   :start-after: 行引用开始，添加开始标签
   :end-before: 行引用结束，添加结束标签

以下是节点引用的内容：（引用index中的免责声明）

.. include:: template_rst_syntax.rst
   :start-after: include_开始标签
   :end-before: include_结束标签

.. note:: 
   - ``:start-after:`` 和 ``:end-before:`` 后的标签名唯一。
   - 使用语法 ``.. 标签`` 在引用处添加。

块引用
~~~~~~~~~~~
使用 ``.. literalinclude::`` 指令，把导入的文件内容当作“字面量”来对待，对导入的所有文本显示在文字块中，不进行再次解析。适用于引用源代码或者其它不希望被处理的文本。

参数释义
   :literalinclude: 引用文件，可以是当前文件，也可以添加文件路径，必须注明文件类型
   :linenos: 对代码编号
   :lines: 引用第1行，第6行往后
   :diff: 比较两份文档不同

引用一份文件：

.. literalinclude:: test1.py
   :encoding: utf-8
   :language: python
   :emphasize-lines: 1,5-6
   :linenos:
   :lines: 1,6-
  
对比两份文件：

.. literalinclude:: test1.py
   :diff: test2.py

.. note:: 
   - 可以以文本的形式直接导入内容。
   - 也可以选择高亮以及提取哪些行。

内联替换
---------
文字替换
~~~~~~~~
使用 ``|text|`` 产生替换，并统一使用 ``.. |text| replace::`` 声明替换内容。

|文本|，在需要使用的时候可以多处引用。

.. |文本| replace:: 这是一个替换文本

.. note:: 
   - 在文档中涉及到频繁重复的长文本或者插入常用的内容，可以使用替换标记。
   - 替换声明在文档中不显示。
   - rst中有一个默认的替换字段 ``|release|``，对应于 :file:`beeSDK/doc/conf_common.py` 配置文件中的 ``release`` 变量，可直接使用。如本文档版本号为： |release|

图像替换
~~~~~~~~
使用 ``|text|`` 产生替换，并统一使用 ``.. |text| image:: 图片路径`` 声明替换图片。

|red| 红灯停， |green| 绿灯行。

|yellow| 这个黄灯的图片是文字替换过来的。

.. |red| image:: ../figures/red.*
   :scale: 10%

.. |green| image:: ../figures/green.*
   :scale: 10%

.. |yellow| image:: ../figures/yellow.*

.. note:: 
   - 图像的替换与文字替换指令相同，适用于在文本中需要添加图片或者操作按钮图标的情况。
   - 可以添加 ``:scale:`` 参数改变图像大小。 

指令
====
| “指令”是rst中一种定义块级元素的方式，比如上述的表格、图片等都是采用指令生成，rst通过不同的指令可以丰富文档的展示效果。
| 指令的语法格式为 ``.. directive::``

强调
----
使用 ``.. 强调::`` 指令生成各种强调窗口。

.. admonition:: 使用admonition可以自定义标题

   - 自定义标题是 ``.. admonition::`` 指令与其他强调语法的不同之处，其余格式均相同
   - 较短内容可以直接在指令后填写，需要和指令保持一个空格
   - 较长的文字可以在区块上继续说明，需要保持相同缩进，不可以顶格

.. note:: 这是note指令，显示为蓝色

.. seealso:: 这是seealso指令，显示为蓝色

.. attention:: 这是attention指令，显示为橙色

.. warning:: 这是warning指令，显示为橙色

.. caution:: 这是caution指令，显示为橙色

.. danger:: 这是danger指令，显示为红色

.. error:: 这是error指令，显示为红色

.. hint:: 这是hint指令，显示为绿色

.. important:: 这是important指令，显示为绿色

.. tip:: 这是tip指令，显示为绿色

选项卡
------
使用 ``.. tabs::`` 指令定义选项卡，子卡页面使用 ``.. tab::`` 指令生成。 

.. tabs:: 

   .. tab:: RTL8762C

      1. Keil MDK-ARM Essential（或更高）V5（或更新）版本
      2. J-Link Software v5.02d（或更新）版本
      3. RTL8762C SDK
      4. EVB Kit

   .. tab:: RTL8762D

      1. Keil MDK-ARM Lite V5（或更新）版本
      2. J-Link Software v6.32（或更新）版本
      3. RTL8762D SDK
      4. EVB Kit
    
   .. tab:: RTL8762E

      1. Keil MDK-ARM Essential（或更高）V5（或更新）版本
      2. J-Link Software v6.32（或更新）版本
      3. RTL8762E SDK
      4. EVB Kit
   
   .. tab:: RTL87x2G

      1. Keil MDK-ARM Essential V5.36（或更新）版本，并搭配ARMCLANG Compiler6.17
      2. J-Link Software v6.44（或更新）版本
      3. RTL87x2G SDK
      4. EVB Kit

.. tabs::

   .. tab:: Linux

      在Linux下运行这个命令。

   .. tab:: Windows

      在Windows下运行另一个命令。

----

.. note:: 
   - 选项卡组件对于展示不同版本的代码，或者展示同一内容的不同语言版本等方面非常有用。
   - 可以在选项卡指令后添加\ :ref:`分隔线`\ ，方便与正文做区分。

折叠
----
使用 ``.. toggle::`` 指令生成折叠，用于创建一个可折叠和展开的段落或者章节。。

.. toggle:: 

   Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

.. note:: 这个功能在需要提供额外信息，但又希望初始显示时保持页面简洁的情况下非常有用。

.. _脚注:
脚注
-----
使用 ``[#name]_`` 产生脚注，并使用 ``.. [#name]`` 声明脚注信息。

脚注在文中会以编号的形式输出 [#第一条]_，且格式上位于上标位置 [#自定义]_。点击文中脚注的链接可以跳转到脚注说明处，通常使用 ``.. rubric::`` 指令置于文档或者段落末尾，之后可以点击该脚注的链接重新跳回之前文中的阅读位置。

.. rubric:: 脚注申明

.. [#第一条] 第一条脚注说明
.. [#自定义] 第二条脚注说明

.. note:: 
   - ``#`` 添加在脚注名称之前可以自动编号，避免序号混乱。
   - 脚注名称不会显示，用作本地区分，HTML上仅显示标号。
   - 脚注的定义不能放在引用之前，否则定义将无法找到。

参考
-----
使用 ``[name]_`` 来引用参考资料，与脚注的用法及其类似，除了除了它在文中输出的是正文格式的文本而不是上标格式的编号。如： [cit2002]_

.. [cit2002] 添加参考资料信息

.. admonition:: 脚注和参考的语法相同，在显示上有以下差别
  
   - HTML：脚注会显示成数字上标，参考则显示与文本一致。
   - PDF：脚注会在页脚作注释，而参考文档信息在整篇PDF最后生成一页\ :ref:`BIBLIOGRAPHY`\ ，放置所有资料。

.. figure::  ../figures/BIBLIOGRAPHY.*
   :align: center
   :name: BIBLIOGRAPHY

   BIBLIOGRAPHY

下载
----
使用 ``:download:`` 进行下载，该指令只对HTML输出有效。

图片下载：:download:`熊猫 <../figures/panda.jpg>`

文件下载：:download:`SDK Doc Maintain Structure <../files/sdk_doc_maintain_structure.pdf>`

程序下载：:download:`example.py <../files/test1.py>`

.. note:: 
   - 对于 ``.png`` ``.jpg`` 格式的图片，浏览器默认会直接打开图片。
   - 对于无法直接打开的文件类型，如 ``.py`` ``.pdf`` ``.doc``，浏览器会弹出窗口选择下载开始通过某种方式打开。

超链接
------
行内超链接
~~~~~~~~~~
| 使用 ```显示名称 <链接地址>`_`` 生成行内超链接，点击可以直接跳转。
| `RealMCU <https://www.realmcu.com/zh/Home>`_ 直接在行内显示。

多处超链接
~~~~~~~~~~
| 如果在文中需要多次使用超链接，避免每次定义，也可以使用以下方式。
| 使用 ``.. _显示名称: 链接地址`` 声明超链接（一般在文末统一定义），并在需要使用时直接用 ``显示名称_`` 即可多次引用。

| 百度_
| RealMCU_

.. _百度: http://www.baidu.com
.. _RealMCU: https://www.realmcu.com/zh/Home

.. note::  ``.. _显示名称: 链接地址`` 这些链接声明不会在文档中显示。

序列图
------

.. msc::
   hscale = "1.3";
   Module,Application;
   Module<<Application      [label="nrf_cloud_connect() returns successfully"];
   Module>>Application      [label="NRF_CLOUD_EVT_TRANSPORT_CONNECTED"];
   Module>>Application      [label="NRF_CLOUD_EVT_USER_ASSOCIATION_REQUEST"];
   Module<<Application      [label="nrf_cloud_user_associate()"];
   Module>>Application      [label="NRF_CLOUD_EVT_USER_ASSOCIATED"];
   Module>>Application      [label="NRF_CLOUD_EVT_READY"];
   Module>>Application      [label="NRF_CLOUD_EVT_TRANSPORT_DISCONNECTED"];

.. msc::
   a,b,c;
   a->b [ label = "ab()" ] ;
   b->c [ label = "bc(TRUE)" ];
   c=>c [ label = "process(1)" ];
   c=>c [ label = "process(2)" ];
   ...;
   c=>c [ label = "process(n)" ];
   c=>c [ label = "process(END)" ];
   a<<=b [ label = "callback()"];
   --- [ label = "If more to run", ID="*"];
   a->a [ label = "next()"];
   a<<=c [ label = "finished()"];


.. msc::
   a,b,c;

   a->b [label="message"];
   b->c [label="response"];


.. mermaid::

   graph TD;
      A-- This is the text! -->B;
      B-- Text! -->C;
      C-- More text! -->A;

.. mermaid::

   graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;

.. mermaid::

   sequenceDiagram
      participant Alice
      participant Bob
      Alice->>John: Hello John, how are you?
      loop Healthcheck
          John->>John: Fight against hypochondria
      end
      Note right of John: Rational thoughts!
      John-->>Alice: Great!
      John->>Bob: How about you?
      Bob-->>John: Jolly good!

.. mermaid::

   graph LR
      A[Start] -- input --> B{Is it valid?}
      B -- yes --> C[Process]
      B -- no --> A
      C -- output --> D[Stop]

.. mermaid::

   gantt
      dateFormat  YYYY-MM-DD
      title Adding GANTT diagram to mermaid
      section A section
      Completed task            :done,    des1, 2014-01-06,2014-01-08
      Active task               :active,  des2, 2014-01-09, 3d
      Future task               :         des3, after des2, 5d
      Future task2               :         des4, after des3, 5d

.. mermaid::

   sequenceDiagram
      participant User
      participant System
      User->>System: Start
      System-->>User: OK
      User->>System: Check
      System-->>User: Valid
      User->>System: Process
      System-->>User: Done
