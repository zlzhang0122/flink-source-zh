任务提交
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

 上面的流程主要包含Client,JobManager,ResourceManager,TaskManager共四个部分.接下来就对每个部分进行详细的分析.

生成StreamGraph
----------------
在用户编写一个Flink任务之后是怎么样一步步转换成Flink的第一层抽象StreamGraph的呢?本节将会对此进行详细的介绍.

StreamGraph生成的主要流程如下:

 * 用户对DataStream声明的每个操作都会将该操作对应的Transformation添加到Transformations列表:List
 * 用户程序中调用env.execute后(batch调用print方法类似),Flink将从List的Sink开始自底向上进行遍历,这也是为何Flink一定要写Sink的原因,没有Sink就无法生成StreamGraph.
 * 如果上游Transformation还没有进行处理,会先对上游的Transformation进行处理,处理即包装成一个StreamNode,再通过Edge建立上下游StreamNode的联系.
 * StreamGraphGenerator.generate()方法会最终生成一个完整的StreamGraph


Or automatically numbered:

 #. Item 1
 #. Item 2

Inline Markup
-------------
Words can have *emphasis in italics* or be **bold** and you can define
code samples with back quotes, like when you talk about a command: ``sudo``
gives you super user powers!