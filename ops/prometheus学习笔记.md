# Prometheus



![image-20240605185307872](/Users/yinxl/Library/Application Support/typora-user-images/image-20240605185307872.png)



## 数据采集

### 采集方式

-   shell 运维的入门脚本
-   python
-   awk
-   lua
-   php
-   go
-   ...



### 采集形式

-   一次性采集

    使用比较简单的方式去采集，按照固定的周期采集，稳定性好，开发逻辑简单，实现快速。

    不够精细智能

-   后台式采集

    数据准确性高，采集精度精细，管理方便，如java，go方式采集的数据

    但是如果开发不够仔细，可能会出现各种内存泄漏，僵尸进程，性能瓶颈等问题。

-   桥接式采集

    本身以后台进程方式运行，但是采集不能独立，依然跟服务器关联。





完整的自愈式的监控体系

真实链路式监控



## 数据模型

prometheus的数据模型是基于时间序列的多维数据模型，主要包括以下概念：

1.  **指标名（Metric Name）**
2.  **标签（Labels）**
3.  **样本（Samples）**

#### 1. 指标名（Metric Name）

指标名，是时间序列的基本标识符，用于描述时间序列的类型。比如cpu使用率，内存使用量等。

指标名通常遵循一种命名规范，以便比较清晰的表达其含义，例如 `http_requests_total` 表示HTTP请求的总数。

#### 2. 标签（Labels）

标签是键值对，用于进一步描述和区分相同指标名的不同时间序列。

每个时间序列由一个指标名和一组标签唯一标识。标签允许你对时间序列进行细分。

例如，对于指标名 `http_requests_total`，你可以使用标签 `method="GET"` 和 `handler="/api/v1"` 来区分不同的HTTP请求类型和路径。

标签有两类：

-   **全局标签（Global Labels）：** 在配置文件中定义，适用于所有的时间序列。
-   **特定标签（Specific Labels）：** 由抓取目标提供，适用于特定的时间序列。



### 数据模型示例

假设你有一个关于HTTP请求总数的时间序列数据，它的指标名为 `http_requests_total`，并使用以下标签：

-   `method`：HTTP请求方法，例如 `GET` 或 `POST`
-   `handler`：处理HTTP请求的路径，例如 `/api/v1`
-   `instance`：监控的主机或实例，例如 `localhost:9090`

下面是一些示例数据点：

```
plaintext
复制代码
{__name__="http_requests_total", method="GET", handler="/api/v1", instance="localhost:9090"} => 1520 @1627398000
{__name__="http_requests_total", method="POST", handler="/api/v1", instance="localhost:9090"} => 320 @1627398000
{__name__="http_requests_total", method="GET", handler="/api/v1", instance="localhost:9091"} => 1420 @1627398000
{__name__="http_requests_total", method="GET", handler="/api/v2", instance="localhost:9090"} => 520 @1627398000
```



### 使用PromQL查询数据

PromQL（Prometheus Query Language）是Prometheus的查询语言，允许你灵活地查询和分析时间序列数据。以下是一些常见的查询示例：

#### 查询某个指标的所有时间序列

```
promql
复制代码
http_requests_total
```

#### 使用标签过滤

```
promql
复制代码
http_requests_total{method="GET"}
```

#### 聚合操作

求和所有时间序列的值：

```
promql
复制代码
sum(http_requests_total)
```

按 `handler` 标签进行聚合：

```
promql
复制代码
sum by (handler) (http_requests_total)
```

#### 计算速率

计算过去5分钟的HTTP请求速率：

```
promql
复制代码
rate(http_requests_total[5m])
```



### 配置文件示例

在Prometheus的配置文件 `prometheus.yml` 中，你可以定义抓取目标和标签配置。例如：

```
yaml
复制代码
global:
  scrape_interval: 15s  # 默认抓取间隔

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          instance: 'localhost:9100'
  - job_name: 'webapp'
    static_configs:
      - targets: ['localhost:8081']
        labels:
          instance: 'localhost:8081'
```

### 小结

Prometheus的数据模型通过使用指标名和标签来灵活地管理和查询时间序列数据。标签使得同一个指标可以通过多维度的数据来进行区分和聚合，从而实现细粒度的监控和分析。PromQL提供了强大的查询能力，使用户能够对收集的数据进行灵活的操作和分析。



# 指标类型

在Prometheus中，指标（metrics）是用来度量和监控系统行为的核心元素。Prometheus支持四种主要类型的指标，每种指标类型都有其特定的用途和特性。下面是对每种指标类型的详细介绍：

### 1. Counter（计数器）

**特性:**

-   **只增不减的累加器**，适用于只能增加的量度，如请求数量、错误数量等。
-   **值只能增加或在重新启动时重置为零。**

**示例:**

```
http_requests_total{method="post", handler="/messages"} 1027
```

这个示例表示一个计数器，用··于统计通过POST方法访问`/messages`路径的HTTP请求总数。

**使用场景:**

-   统计请求总数
-   统计错误总数
-   统计处理的事件数量

### 2. Gauge（仪表）

**特性:**

-   **可以任意增减**，适用于度量瞬时值，如温度、内存使用量、队列长度等。
-   可以增加、减少或保持不变。

**示例:**

```
plaintext
复制代码
memory_usage_bytes{type="heap"} 4.2e+06
```

这个示例表示一个仪表，用于统计当前堆内存的使用量。

**使用场景:**

-   度量当前温度
-   度量内存使用量
-   度量队列长度

### 3. Histogram（直方图）

Histogram是一种用于测量和展示值分布的度量类型，特别适用于需要统计数据分布的情况，如请求延迟，响应大小等。Histogram不仅会记录样本值的分布情况，还会记录样本的总数和总和。

**特性:**

-   **值分布统计：** 通过预定义的桶（buckets）记录值的分布。
-   **记录总数和总和：** 除了分布之外，Histogram还记录样本值的总数和总和。
-   **适用于高动态范围的值：** 适合用来监控和分析响应时间、请求大小等具有较大动态范围的指标。

**示例:**

```
plaintext
复制代码
http_request_duration_seconds_bucket{le="0.1"} 5932
http_request_duration_seconds_bucket{le="0.5"} 7845
http_request_duration_seconds_bucket{le="1.0"} 8954
http_request_duration_seconds_sum 12934.560
http_request_duration_seconds_count 9123
```

这个示例表示一个直方图，用于统计HTTP请求的响应时间分布情况。每个`bucket`表示响应时间落在该桶中的请求数，`sum`表示所有请求的响应时间总和，`count`表示总的请求数。

**使用场景:**

-   统计请求延迟
-   统计响应大小
-   统计处理时间

### 4. Summary（摘要）

**特性:**

-   类似于直方图，但记录的是分位数（quantiles），同时记录样本值的总数和总和。
-   适用于统计请求延迟、响应大小等的分位数。

**示例:**

```
plaintext
复制代码
http_request_duration_seconds{quantile="0.5"} 0.245
http_request_duration_seconds{quantile="0.9"} 0.543
http_request_duration_seconds{quantile="0.99"} 0.901
http_request_duration_seconds_sum 12934.560
http_request_duration_seconds_count 9123
```

这个示例表示一个摘要，用于统计HTTP请求的响应时间分位数。`quantile`表示分位数，`sum`表示所有请求的响应时间总和，`count`表示总的请求数。

**使用场景:**

-   统计请求延迟分位数
-   统计响应大小分位数
-   统计处理时间分位数



# prometheus 框架



![img](/Users/yinxl/Desktop/architecture.png)



### Prometheus架构

1.  **Prometheus Server**
    -   **Retrieval:** 负责从配置的抓取目标（targets）中获取监控数据。这些目标可以是任何暴露Prometheus兼容指标的服务。Retrieval，检索的意思。
    -   **TSDB (Time Series Database):** 一个高效的时间序列数据库，用于存储采集到的监控数据。数据存储在本地的HDD或SSD上。
    -   **HTTP Server:** 提供HTTP接口，允许用户使用PromQL查询和可视化存储的数据。还可以通过HTTP接口进行配置和管理。
2.  **Service Discovery**
    -   **Kubernetes:** 动态发现Kubernetes中的服务和Pod，以便自动更新抓取目标。
    -   **file_sd:** 使用文件中的静态配置进行目标发现。
3.  **Jobs/Exporters**
    -   **Jobs/Exporters:** 各种导出器和应用程序（如node_exporter、cAdvisor、黑盒探测器等），它们会暴露Prometheus兼容的指标，供Prometheus Server抓取。
4.  **Pushgateway**
    -   **Pushgateway:** 用于短期运行的批处理作业，这些作业在完成时将指标推送到Pushgateway。Prometheus Server从Pushgateway中抓取这些指标。
5.  **Alertmanager**
    -   **Alertmanager:** 处理从Prometheus Server发送的报警通知。Alertmanager可以进行去重、分组和路由，并将报警通知发送到各种通知渠道（如Email、PagerDuty等）。
6.  **Prometheus Web UI**
    -   提供一个内置的用户界面，允许用户浏览存储的数据和查询结果。
7.  **Grafana**
    -   一款流行的开源数据可视化工具，可以与Prometheus集成，用于创建和共享监控仪表板。
8.  **API Clients**
    -   使用Prometheus的HTTP API进行数据查询和操作的客户端程序。



### 工作流程

1.  **数据采集**
    -   Prometheus Server通过HTTP协议从各种抓取目标（Jobs/Exporters、Pushgateway等）中定期拉取监控数据。
    -   Service Discovery机制帮助Prometheus Server自动发现并更新这些抓取目标。
2.  **数据存储**
    -   采集到的数据存储在本地的时间序列数据库（TSDB）中。
    -   数据按照时间块（通常是2小时）进行存储和压缩，以优化查询性能和存储空间。
3.  **数据查询**
    -   用户可以通过Prometheus Web UI或Grafana使用PromQL进行查询，获取所需的监控数据。
    -   Prometheus的HTTP API也允许客户端程序进行查询和操作。
4.  **报警处理**
    -   Prometheus Server根据预定义的报警规则生成报警。
    -   这些报警通过HTTP推送到Alertmanager，由Alertmanager进行处理和通知。





# Thanos



![收到](https://docs.google.com/drawings/d/e/2PACX-1vRdYP__uDuygGR5ym1dxBzU6LEx5v7Rs1cAUKPsl5BZrRGVl5YIj5lsD_FOljeIVOGWatdAI9pazbCP/pub?w=960&h=720)



