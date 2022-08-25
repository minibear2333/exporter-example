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

### 源码讲解

注册 
1、https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L396

2、https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L264:20


收集
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L405

收集到的数据
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L478#tab=def

数据组装
1、https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L581

2、https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/internal/metric.go?L69

数据转换TEXT
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/common/expfmt/text_create.go?L67:6#tab=references

写入io.Writer流
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/common/expfmt/encode.go?L64:6#tab=references

TCP返回
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/promhttp/http.go?L163:17

数据写入文件
https://sourcegraph.com/github.com/minibear2333/exporter-example@ece1f0b9acc0f92ac14c335f1bdc405e253d0fc3/-/blob/vendor/github.com/prometheus/client_golang/prometheus/registry.go?L554

### pushgateway

`pushgateway`的样例

```go
func (p *PushGateway) Push(name string, help string, labels map[string]string, value float64) {
	collectDurabilityPercent := prometheus.NewGauge(prometheus.GaugeOpts{
		Name: name,
		Help: help,
	})
	collectDurabilityPercent.Set(value)
	pusher := push.New(p.QueryAddress, p.JobName).BasicAuth(p.Username, p.Password)
	pusher.Collector(collectDurabilityPercent)
	for k, v := range labels {
		pusher.Grouping(k, v)
	}
	if err := pusher.Push(); err != nil {
		fmt.Println("Could not push completion time to Pushgateway:", err)
	}
}
```

### 参考

[认识Prometheus，开发自己的exporter](https://www.jianshu.com/p/5db23a280e1d)
