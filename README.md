# exporter-example
exporter标准样例项目

### 使用方法
可直接fork，在此基础上进行开发即可

目录结构
```
exporter/
├── collector
│   └── node.go #采集器位置
├── go.mod
└── main.go #入口
```

**collector/node.go** 文件涵盖了`Counter`、`Gauge`、`Histogram`、`Summary`四种情况

### Prometheus 中 metric 的格式

格式：

```
<metric name>{<label name>=<label value>, ...}
```

例如：

```
api_http_requests_total{method="POST", handler="/messages"}
```

**metric name** ：唯一标识，命名遵循 `[a-zA-Z_:][a-zA-Z0-9_:]*.`

### Prometheus 中 metric 的类型

**Counter**

一个 `Counter` 表示一个累计度量，只增不减，重启后恢复为 0。适用于访问次数统计，异常次数统计等场景。

**Gauge**

`Gauge` 表示可变化的度量值，适用于 `CPU` , 内存使用率等

**Histogram**

`Histogram` 对指标的范围性（区间）统计。比如内存在 0%-30%，30%-70%之间的采样次数。

Histogram 包含三个指标：

- `<basename>` ：度量值名称
- `<basename>_count` ： 样本反正总次数
- `<basename>_sum` ：样本发生次数中值得综合
- `<basename>_bucket{le="+Inf"}` ： 每个区间的样本数

**Summary**

和 `histogram` 类似，提供次数和总和，同时提供每个滑动窗口中的分位数。

### histogram 和 Summary 的对比

| 序号         | histogram                    | Summary                  |
| ------------ | ---------------------------- | ------------------------ |
| 配置         | 区间配置                     | 分位数和滑动窗口         |
| 客户端性能   | 只需增加 counters 代价小     | 需要流式计算代价高       |
| 服务端性能   | 计算分位数消耗大，可能会耗时 | 无需计算，代价小         |
| 时序数量     | \_sum、\_count、bucket       | \_sum、\_count、quantile |
| 分位数误差   | bucket 的大小有关            | φ 的配置有关             |
| φ 和滑动窗口 | Prometheus 表达式设置        | 客户端设置               |
| 聚合         | 根据表达式聚合               | 一般不可聚合             |


### 参考

[认识Prometheus，开发自己的exporter](https://www.jianshu.com/p/5db23a280e1d)
