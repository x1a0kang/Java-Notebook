* checkpoint
* source，transformation/Operator，sink
  * source，自定义kafka算子，获得dataStream；
  * transformation：map，filter，keyby等，还有自定义Window
  * sink：到本地文件，数据库，kafka等等
* jobmanager，taskmanager
  * jobmanager负责任务的调度，检测taskmanager的心跳，协调checkpoint和容错
  * 一个taskmanager就是一个JVM进程，内部可以多线程执行；
  * taskmanager内分为多个槽，每个槽可以执行一个任务，taskmanager最多能同时并发执行的任务数就是槽的数量
* Window：
  * 滚动窗口：以固定长度对时间切片，每片之间不重叠，适合做每个时间段的统计；
  * 滑动窗口：以固定长度滑动，时间片之间有重叠，适合做近10分钟这种统计；
  * 会话窗口：一系列时间组合
* eventtime和waterMark
  * flink可以定义事件发生的事件eventTime，比如前一分钟的数据因为网络延迟，到了下一分钟才到达flink，flink可以读取日志的时间字段判断真正的时间，这样不会让前一分钟的日志和下一分钟的日志产生乱序；
  * waterMark水位，就给flink处理设置一个延迟时间，比如一分钟，flink就会在下一分钟结束时，处理上一分钟的数据；

* flink提供了储存的功能，所以它是有状态的，且有exactly one精确一次性
  * flink的精确一次性指的是，状态只持久化一次到最终的储存介质中，如本地数据库/HDFS
  * 多久储存一次可以手动配置，checkpoint interval
  * checkpoint就是flink储存的上一次的状态信息，当flink挂掉后，可以取出上一次的状态信息，重新开始执行；
  * flink内部有barrier，每个环节处理到barrier都会上报，sink也上报了就说明checkpoint处理完了；