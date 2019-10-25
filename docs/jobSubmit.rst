任务提交源码解析
===============
Flink任务在被提交到Yarn上后会经过如下流程,具体如下:

.. image:: image/flink-submit-to-yarn.png

 #. Client从客户端代码生成的StreamGraph提取出JobGraph;
 #. 上传JobGraph和对应的jar包;
 #. 启动App Master;
 #. 启动JobManager;
 #. 启动ResourceManager;
 #. JobManager向ResourceManager申请slots;
 #. ResourceManager向Yarn ResourceManager申请Container;
 #. 启动申请到的Container;
 #. 启动的Container作为TaskManager向ResourceManager注册;
 #. ResourceManger向TaskManager请求slot;
 #. TaskManager提供slot给JobManager,让其分配任务执行.

Subject Subtitle
----------------
Subtitles are set with '-' and are required to have the same length
of the subtitle itself, just like titles.

Lists can be unnumbered like:

 * Item Foo
 * Item Bar

 .. image:: image/flink-submit-to-yarn.png

Or automatically numbered:

 #. Item 1
 #. Item 2

Inline Markup
-------------
Words can have *emphasis in italics* or be **bold** and you can define
code samples with back quotes, like when you talk about a command: ``sudo``
gives you super user powers!