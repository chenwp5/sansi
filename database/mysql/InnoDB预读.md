# 预读

读取预取请求是一种异步的I/O请求，用于在预期即将需要这些页面时，将多个页面提前加载到缓冲池中。这些请求会一次性获取一个区段中的所有页面。InnoDB使用两种读取预取算法来提高I/O性能：

- 线性读取预取是一种预测哪些页面可能很快被需要的技巧，基于缓冲池中按顺序访问的页面。您可以通过调整触发异步读取请求所需的连续页面访问次数来控制InnoDB何时执行读取预取操作，使用配置参数`innodb_read_ahead_threshold`。
- 随机读取预取是一种预测页面可能很快被需要的技巧，基于缓冲池中已经存在的页面，而不考虑这些页面的读取顺序。如果缓冲池中有来自同一区段的13个连续页面，InnoDB将异步发出请求，预取该区段中剩余的页面。要启用此功能，请将配置变量`innodb_random_read_ahead`设置为ON。

`SHOW ENGINE INNODB STATUS`命令会显示统计信息，以帮助您评估读取预取算法的有效性。这些统计信息包括以下全局状态变量的计数器信息：

- `Innodb_buffer_pool_read_ahead`
- `Innodb_buffer_pool_read_ahead_evicted`
- `Innodb_buffer_pool_read_ahead_rnd`

这些信息在微调`innodb_random_read_ahead`设置时非常有用。