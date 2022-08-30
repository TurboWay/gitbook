## map-reduce 图解

![image-20210302144302050](http://pic.turboway.top/blogimg/image-20210302144302050.png)


## map-reduce 详细图解 [](https://blog.csdn.net/bingduanlbd/article/details/51933914)

![image-20210302144009473](http://pic.turboway.top/blogimg/image-20210302144009473.png)


>[MapReduce shuffle过程剖析及调优](https://blog.csdn.net/bingduanlbd/article/details/51933914)

* map端
  * map()输出结果先写入环形缓冲区
  * 缓冲区100M；写满80M后，开始溢出写磁盘文件
  * 此过程中，会进行分区、排序、combine（可选）、压缩（可选）
  * map任务完成前，会将多个小的溢出文件，合并成一个大的溢出文件（已分区、排序）
  
* reduce端
  * 拷贝阶段：reduce任务通过http将map任务属于自己的分区数据拉取过来
  * 开始merge及溢出写磁盘文件
  * 所有map任务的分区全部拷贝过来后，进行阶段合并、排序、分组阶段
  * 每组数据调用一次reduce()
  * 结果写入HDFS