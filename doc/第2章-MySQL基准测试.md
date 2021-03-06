# 第2章 MySQL基准测试

标签：MySQL


---

**该笔记用于简记MySQL和基于MySQL基准测试的重要性、策略和工具**

## 基准测试的使用场景

- 简单记对不同硬件软件和操作系统的配置正确与否，与对设备的配置正确性测试
- 为对系统异常、和解决
- 对假设的验证、和相关优化策略的验证
- 对未来业务拓展、对更高负载、对适应可变环境的能力测试

---

## 基准测试的策略

- 两方面
	- 测试整个应用系统
	- 测试单独的MySQL
- 测试整个应用系统
	- 优点: 关注不仅MySQL而是整体性能、各部分缓存对MySQL的影响、真实表现
	- 缺点: 很难建立，难配置，设置真实数据的基准测试复杂并耗时；且若单方面出错，即对MySQL结论出错
- 测试单独MySQL
	- 优点: 适合用于开发初期，需要可靠的数据本身和大小量，可针对某个具体问题检测
	- 缺点：对后期扩张的性能表现只能通过模拟大量的数据和压力进行
- 该问题可以考虑：可针对的问题、数据量、数据来源

---

## 测试重要指标

- 吞吐量
	- 指单位时间内的事务处理数
- 响应时间或者延迟
- 并发性
	- 区分web服务器的并发性与MySQL数据库并发性的区别
	- web服务器的并发性仅表示会话存储机制可以处理多少数据的能力
	- MySQL数据库并发性，如论坛用户访问并不等同于对MySQL有高并发操作。可能同时有n个MySQL数据库服务器连接，而仅有10~15个并发请求到MySQL数据库。关注MySQL并发性是指正在工作的并发操作，或者时同时工作中的线程数或者连接数。
- 可扩展性
	- 对系统进行扩展，看是否对吞吐量等指标有线性增长。一般都为非线性，吞吐量、响应时间等会受限随着数据量增大等非线性变化

---

## 基准测试方法

- 基准测试常见的错误几个方面
	- 数据来源的不真实(数据子集、单一来源等)
	- 对真实环境的模拟(忽略用户真实行为等)
	- 测试环境的配置不当（如服务器时间配置，或忽略了本地测试环境出现的误差如系统预热等）
- 注意：还可加入网络负载和速度模拟真实环境部署

---

## 设计和规划基准测试

- 测试主要流程
	- 第一步提出问题并明确目标
	- 选择合适的测试方案（标准的基准测试或设计专用的测试）
	- 建立一个单元测试集作初步的测试，并运行多遍；后应进行真实数据集的测试，选择代表性的时间段
	- 选择在不同级别记录查询（集成式查询可记录Web服务器上的HTTP请求，也可打开MySQL的查询日志）
	- 应写下测试规划，便于日后做测试的其他人进行参考，或者用于日后自己重看测试数据，包括测试数据、系统配置的步骤、如何测量和分析结果，以及预热方案等
	- 建立将参数和结果文档化的规范，每一轮测试都必须进行详细记录
- 注意
	- 时间要足够长，保证测试结果真实性
	- 有一个扩展基准测试I/O性能图，测试不充足相当于花费的所有时间都是一种浪费。有时候要相信别人的测试结果，这比做一次半粒子的测试来得到一个错误的结论要好

---

## 获取系统性能和状态

- p43提供一个收集MySQL测试数据的shell脚本（待学）
- 准确的测试结果受到以下部分影响：系统状态、数据置入方式（插入或利用快照还原等）、外部压力、性能分析和监控系统、详细的日志记录、周期性作业等
- 建议：
	- 为每一轮基准测试创建单独的子目录，将测试结果、配置结果、测试指标、脚本和其他相关说明都包存在其中。因为完整的基准测试结果非常宝贵，不管当前是否需要用到某些结果，也应该先保存下来。
	- 每一次修改的数据应该小，部分修改逐步对比
	- 出现异常，也应认真分析，可能会得出有价值的结果，或者一个严重的错误
	- 推荐自动化基准测试，避免测试人员偶尔遗漏步骤，规划统一操作
	- p47提供一个脚本从前面的数据才叫脚本采集到的数据中抽取时间维度信息（待学）
- 绘图：作用有利于对比与正常值区别，波动，得到一些在平均值单数据信息中无法显示出的问题

---

## 基准测试工具

- 集成式测试工具
	- ab：Apache HTTP服务器基准测试工具。可以测试HTTP服务器美妙可以处理多少请求，用途仅针对单URL进行压力测试
	- http_load：类似ab，但可通过一个输入文件提供多个URL
	- JMeter：可测试Web应用、FTP服务器、或通过JDBC进行数据库查询测试，比前两种复杂，且拥有绘图接口
- 单组件式绘图工具
	- mysqlshap：可模拟服务器的负载，并输出计时信息。包含在MySQL5.1发行保重，测试时可以执行并发连接数
	- MySQL Benchmark Suite(sql-bench)：可用在不同数据库服务器上比较测试，单线程，主要用于测试服务器执行查询的速度。包含了大量预定义的测试蛮容易测试，比较适合对比不同存储引擎或不同配置的性能测试
	- Super Smark：可模拟多用户访问，可加载测试数据到数据库，并支持使用随即数据填充测试表
	- Database Test Suit：类似某些工业标准测试的测试工具及，dbt2时免年费的TPC-C OLTP测试工具。作者之前经常使用
	- sysbench：多线虫系统压测工具。可以根据影像数据库务夫妻性能的各种因素来评估系统的性能，例如，可以用来测试文件I/O，操作系统调度器、内存分配和传输速度、POSIX线程，以及数据库服务器等。全能测试工具，支持MySQL、操作系统和硬件的硬件测试
- MySQL的BENCHMARK()函数：简单返回服务器执行表达式的时间，不涉及分析和优化的开销。表达式必须包含用户定义的变量，否则多次执行同样的表达式会因为系统缓存命中而影响结果

```MySQL
mysql> SET @input:='hello world';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT BENCHMARK(1000000,MD5(@input));
+--------------------------------+
| BENCHMARK(1000000,MD5(@input)) |
+--------------------------------+
|                              0 |
+--------------------------------+
1 row in set (0.19 sec)

mysql> SELECT BENCHMARK(1000000,SHA1(@input));
+---------------------------------+
| BENCHMARK(1000000,SHA1(@input)) |
+---------------------------------+
|                               0 |
+---------------------------------+
1 row in set (0.25 sec)
```

- 基准测试案例（待学）
	- http_load
	- MySQL Benchmark Suite
	- sysbench
	- dbt2 TPC-C测试