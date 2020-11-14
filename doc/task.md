### StreamTask

当StreamTask被调度执行的时候，其具体的生命周期如下：
*  -- setInitialState -> provides state of all operators in the chain
*  -- invoke()
*        |
*        +----> Create basic utils (config, etc) and load the chain of operators
*        +----> operators.setup()
*        +----> task specific init()
*        +----> initialize-operator-states()
*        +----> open-operators()
*        +----> run()
*        +----> close-operators()
*        +----> dispose-operators()
*        +----> common cleanup
*        +----> task specific cleanup()