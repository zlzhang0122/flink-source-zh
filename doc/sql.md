### Flink SQL

Flink SQL是Flink内部最高级的API，使用者可以通过SQL语句执行流批任务，屏蔽了底层DataStream/DataSet的细节，从而降低了使用门槛。
那么一条Flink SQL语句究竟是如何转化为可执行的任务的呢？就让我们来深入的看一看吧。

当然，在此之前有些前置知识需要先介绍一下，这就是Apache Calcite和Blink Planner。
Flink使用了通用的SQL解析与优化引擎Apache Calcite，Calcite在Flink中主要承担以下任务：
  * 解析：将SQL语句转化为抽象语法树(AST)，即SqlNode树;
  * 验证：根据Catalog中的元数据进行语法检查;
  * 逻辑计划：根据AST和元数据构造出逻辑计划，即RelNode树;
  * 逻辑计划优化：按照预定义的优化规则RelOptRule优化逻辑计划。Calcite中的优化器RelOptPlanner有两种，一是基于规则优化(RBO)的HepPlanner，
  二是基于代价优化(CBO)的VolcanoPlanner;
  * 物理计划：将优化的逻辑计划翻译成对应执行逻辑的物理计划;

在物理计划之后，还需要通过代码生成(code generation)将SQL转化为能够直接执行的DataStream/DataSet API程序。