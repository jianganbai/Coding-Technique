# Hadoop学习笔记

## 架构

- Hadoop生态很大，Spark是其中之一

### 核心组件

- HDFS（分布式文件系统）：数据存储，基于网络
- YARN（作业调度和集群资源管理的框架）：资源任务调度，2.0版本后引入
- MAPREDUCE（分布式运算编程框架）：数据计算

### HDFS文件系统

- 1个NameNode：主服务器上
  - 将大数据分成数据块，记录数据块所在节点
  - 向DataNode发送指令
- 多个DataNode：
  - 管理本节点的数据块
  - 向NameNode报告，接收NameNode的指令

### MapReduce

- 方法
  - **Map**：并行处理数据块分，输出key-value对
    - Hadoop是存在硬盘中，Spark是存在内存中
  - **Group by key**：把所有小块Map的结果中相同的key放在一块，sort and shuffle
  - **Reduce**：对相同key的再处理（自定义）
- **Hadoop 1.0的进程**
  - JobTracker：主服务器上，调度资源、监视不同节点上任务执行情况 => 任务繁杂
  - TaskTracker：从节点上，接收JobTracker提供的任务
- **Hadoop 2.0 => YARN**
  - ResourceManager：主服务器上，只管资源调度
  - NodeManager：从节点上，管理节点资源并包包节点信息
  - ApplicationMaster：将1.0的JobTracker拆分成ResourceManager和ApplicationMaster
    - 每个程序1个ApplicationMaster，执行完成自动销毁
    - 可针对不同任务自定义

## HDFS指令

- HDFS文件系统的操作指令
  - `hadoop fs`等价于`hdfs fs`
  - 指令在linux的shell命令行里输，加`hadoop fs`才能区分是给linux的指令还是给hadoop的指令
- 与linux完全一致的
  - `hadoop fs -mv <pth1> <pth2>`：移动文件
  - `hadoop fs --mkdir <dir>`：创建目录
  - `-ls`, `-cat`, `-cp`, `-rm`, `-du`
- 有改变的
  - `hadoop fs -put <local_files> <hdfs_path>`：从local向hdfs上传文件
  - `hadoop fs -get <hdfs_path> <local_files>`：从hdfs向local下载文件
  - `hadoop fs -touchz <path>`：创建文件
  - `hadoop fs -count <dir>`：统计文件总数&大小
  - `hadoop fs -df <path>`：统计文件系统详细信息
  - `hadoop fs -tail <file>`：查看文件尾部
  - `hadoop fs -getmerge <src> <local>`：将\<src\>下的文件合并（类似于打包），然后下载到local

## 管理指令

- `yarn application -list`：查看正在运行的application，包含application id
- `yarn application -kill <application id>`：根据application id终止application

## PySpark

- spark本身是用scala语言写的，pyspark包提供了python接口
  - 可按python语法处理数据，再编译成spark命令

### 示例代码

- ```python
  from pyspark import SparkContext
  
  # 语料库：We investigate visualisations of networks on a 2-dimensional torus topology.
  sc = SparkContext(appName='demo:test')  # 提交的任务名称
  rdd_file = sc.textFile(appName='./readme.txt') # 读取文件，rdd_file即为新RDD
  # 若读取本地文件，则master参数为'local'
  
  # 对RDD操作，其它操作：.count(), .first(), .filter(lambda x: ???)
  rdd_line = rdd_file.flatMap(lambda x: x.split(' ')).map(lambda x: (x, 1)).reduceByKey(lambda x,y: x + y).sortBy(lambda x: x[1], ascending=False)
  # 分词 flatMap(lambda x: x.split(' ')) 
  # 统计词频 map(lambda x: (x, 1)).reduceByKey(lambda x,y: x+y)
  # 排序 sortBy(lambda x: x[1], ascending = False)
  
  print(rdd_line.take(10))  # 取结果
  
  rdd_line.saveAsTextFile('./results')  # 储存结果 (hdf)
  ```
  
  - `sc.textFile`返回的是RDD

### 提交任务

- 提交任务：`spark-submit --master=yarn xxx.py`
  - `--master`：集群主节点的URL
  - `--py-files`：python库依赖
  - `--num-executors`：executor个数（类似于进程数）
  - `--executor-cores`：每个executor的核数（类似于线程数，不是CPU核），每个核只执行1个task
  - `--executor-memory`：每个executor-memory的内存大小
  - `--conf`：设置其它参数
    - `--conf spark.port.`
- 交互式编程：`pyspark --master=yarn`
  - 可在新命令行中按行输入代码调试

### SparkContext操作

- `sc = SparkContext(appName=任务名)`：构建Spark任务

- `sc.parallelize(list, partition_num)`：由内存创建RDD

- `sc.textFile(大文本文件)`：由大文件创建RDD
  
  - 一般每行为一个元素，默认分区数等于大文件的分区数

- `sc.setLogLevel()`：设置日志输出等级，分别为'INFO', 'WARN', 'ERROR'

### RDD操作

- 对RDD的操作分为Actions和Transformations
  - Actions：返回python变量
  - Transformations：返回新RDD

#### Actions

- `rdd.count()`：返回元素个数

- `rdd.first()`：返回第1个元素

- `rdd.take(10)`：取前10个结果
  
  - 从1开始计数，`rdd.take(0)`无结果

- `rdd.distinct()`：统计不同元素个数

- `rdd.collect()`：将运行结果全部送到一个节点上并返回python变量
  
  - `rdd.coalesce(1)`：将结果送到一个节点，仍为RDD

- `rdd.sum()`：对所有元素求和，前提是元素可加

#### Transformations

- 说明
  - Spark变换函数既可以是lambda函数，也可以是自定义有名函数
  - 既可按MapReduce的框架变换，也可按别的框架变换
- `rdd.filter(lambda x: True or False)`：提取满足条件的元素，类似于python中的lambda&filter函数
- `rdd.map(func)`：对每个元素执行func变换，保留原来的结构
- `rdd.flatMap(func)`：对每个元素执行func变换，再将所有RDD的输出拼在一起
  - map是RDD处理结果拼接时保留将同RDD的放在子列表中
- `rdd.reduceByKey(func)`
  - rdd的元素为2元元组，第1项为key，第2项为value，上一步往往为`.map`
  - 将有相同key的所有value按func规则合在一起，返回[(key, value)]

#### Partition

- Partition分区
  
  - 用textFile读取时，分区数=part数

- `rdd.getNumPartitions()`：获取RDD的分区数

- `rdd.repartition(num_partition)`：增加/减少RDD的分区数至num_partition
  
  - RDD的分区数建议为CPU内核总数的3-4倍

- `rdd.coalesce(num_partition)`：将RDD合并为num_partition个分区
  
  - `.repartition`和`.coalesce`复杂度都很高，慎用
    - `.repartition`需要shuffle
    - `.coalesce`仅部分executor合并，其余在空计算
    - 若合并到1个分区，建议使用`.repartition`，`.coalesce`大量executor在空跑
    - 仅建议在原分区>目标分区且二者都很大时，采用`.coalesce`

#### 保存

- `rdd.cache()`：将RDD保存到内存，加快访问

- `rdd.saveAsTextFile(hdfs_pth)`：将RDD以文本形式保存到hdfs里
  
  - 在hdfs_pth创建文件夹，在下面存放part-00000, part-00001等，文件数=分区数
  - 保存时hdfs_pth路径不能存在

- `rdd.checkpoint()`：将需要复用的中间结果存放到HDFS
  
  - ```python
    sc.setCheckpointDir(hdfs_pth)
    rdd.checkpoint()
    ```

- `rdd.unpersisit()`：删除RDD
