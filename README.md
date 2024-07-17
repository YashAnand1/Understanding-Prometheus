<div align=center>
__________________________________________

![img](https://i.imgur.com/GzEatYl.png)
# Understanding Prometheus: An Introduction
</div>

## Monitoring & Prometheus         

Monitoring is a way of actively observing & analysing something for keeping track of and ensuring the expected progress. With Monitoring Tools, we can prioritise prevention over cure - As something that is about to go wrong in an application, server or a machine, can be prevented through tools like Prometheus or at least fixed by referring to the information gathered by them during monitoring. 

Prometheus is a similar Monitoring tool that also allows Visualisation, Alerting and Querying. Prometheus can monitor various **Metrics** of various **targets** by **scraping** them at a specified **interval** of time into a **Time-Series Database with timestamps**. As an example, we can monitor and visualise the available FileSystem space using the [`node_filesystem_avail_bytes` Metric](https://prometheus.io/docs/guides/node-exporter/#exploring-node-exporter-metrics-through-the-prometheus-expression-browser) of my laptop or target by recording data at every hour (Scraping Interval) and storing it in its Time-Series Database with timestamps. 

Prometheus and its components can be understood in the following manner:
- **Prometheus**: A Monitoring Tool that scrapes Metrics periodically from Targets and stores them into TSDB with timestamps and provides Visualisation, Alerting & Querying
- **Metrics**: A measurement of something (with respect to time here) - Ex: process_cpu_seconds_total, etc.
- **Target**: Metric will be of a source - Where is the metric being taken from? - Ex: My laptop, a Game Server, an App, etc. 
- **Scraping Interval**: How relentlessly/persistently the metrics are being scraped/collected - Ex: 2s, 8h, 6d, 11w, 1y, etc.
- **Database**: Time Series Database stores data with timestamps - Ex: Values of a metric will be collected at different timestamps

We can also consider the following analogy for gaining a better understanding of Prometheus:
> Doctor creates a chart to measure how many times patient drinks water along and creates a table with timestamps

The following similarities can be seen in the examples provided above:
<div align=center>

| Term | Prometheus |  Odometer | Superset |	
|------|----------|---------|------------|	
| Monitoring Tool | Prometheus  | Odometer | Superset |
| Target          | Laptop  | Car | Redmine |
| Metrics         | node_filesystem_avail_bytes | Distance Travelled  | Tickets |
| Scraping Interval| 1h  | No Intervals | 24h |
| TSDB        | Is a [Time Series Database](https://www.reddit.com/r/Database/comments/1ayaj1b/time_series_database/) - Metrics stored with timestamps | Not a time-series database as timestamps are not recorded | Not adding data *to* but pulling *from* a database for visualisation |
</div>

In the following section, we will be better understanding the basics of how everything works within Prometheus.

-------------------------------------------

## Architecture of Prometheus

<div align=center>

![img](https://i.imgur.com/oBXCMLv.png)

[Image Source](https://medium.com/@extio/unveiling-the-architectural-brilliance-of-prometheus-af07cca14896)
</div>

This architecture involves the following sequence for collecting metrics to recording them into the TSDB for querying and visualisation:
> Target Sources are first learnt of using Service Discovery and then their metrics are scraped by hitting their endpoints. The scraped data is stored and then made available for querying, visualisation and alerts.  

A slightly more detailed explanation of the above sequence is as follows:
### 1. Service Discovery (Discovering Target Sources)
This comes before the scraping process as the endpoints of the target source whose metrics are to be scraped have to be first learnt of. The targets are defined within the `Prometheus.yml` configuration file in the following manner:
```
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"] 
```
metrics

### 2. Scraping Metrics From Exporters or Client Library
The metrics to be selected for applications, servers or machines are first defined and then made available at an endpoint that ends with `/metrics`. The metric endpoints can be Instrumented or defined within the source-code of an application with Client Libraries so that when it is run, its metrics would be available on an endpoint.

Otherwise, exporters can be used for an existing target for putting the metrics at an endpoint: The source code of Linux need not be Instrumented since Exporters can generate metrics and Exporters will convert the metrics into a text-format that is understood by Prometheus.

Prometheus will send HTTP request / Scrape Request to the target endpoint at a defined Scrape Interval, pull the metrics and store them in the TSDB.

### 3. Storage, Querying and Alerting For Metrics
**Storage**
- The pulled/scraped metrics are pushed to Prometheus' TSDB, which is located on the same local disk as the Prometheus Server
- This DB can process millions of samples per second which makes the Prometheus Server good at fetching metrics of 1000s of machines            
**Querying**
- PromQL can be used for instant querying that can generate output in the form of graphs and tables     
**Alerting**
- Prometheus uses Alert Manager for sending alerts to which are then converted into Notifications for Email, Telegram, etc.

------------------------------------

## Types of Metrics

**1. Counter**
   - A metrics type where the value only gets incremented or reset
   - **Example**: Height as it only increments or the number of requests made
   to a server
   - **Use Cases**:
      - When a value that only goes up is to be recorded
      - When you want to know how fast/slow the value is increasing over
      time
      - When cumulative data is required

Used for measurements that only increase or get reset. Cumulative so actual value is not very useful but it helps find the rate of change by differentiating between 2 timestamps. 

Counters usually end with `_total` suffix and is mostly useful when used with the `rate()` function to find the how much or fast a change happened. In the following example, we understand how many average requests per second were received:
```
rate(prometheus_http_requests_total{handler="/api/v1/query"}[1m])
```

Similarly, the `increase()` function can also be used to find the overall difference:
```
increase(prometheus_http_requests_total[1m])
```

An example of such a metric is:
```
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 984.44
```

**2. Gauge**
   - Snapshot of current state
   - A metric type used for values that can increase or decrease over time
   - **Example**: Weight, temperature, cpu, ram, storage, etc. as they can increase & decrease
   - **Use Cases**:
      - When a value that can increase or decrease or fluctuate over time
      is to be recorded
      - When the recording of rate is not required - Further research
      required

Example of such a metric is as follows:
```
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 29
```

**3. Histogram**
   - A metric type that records frequency of events over a time range or
   buckets (predefined ranges to group metrics)
   - Accurate aggregation 
   - More consuming in storage - every sample is not just a single number but a few samples (one per bucket).
   - **Example**: Tracking time taken to start PC daily after shutdown or
   tracking http request duration - In former, bucket could be **known**
   range of weight time like 0-5 Min, 5-10 Min, etc.
   - **Use Cases**:
      - When approximation and slight inaccuracy of metrics is not an issue
      - When many measurements of a value is to be taken for averaging
      later on - Sum of count and time is calculated by default
      - When cumulative data and previous values being added with current
      one is not an issue

Distribution of measurements used to represent duration or size. The entire range is divided into Intervals/Buckets and count how many measurements fall into each bucket.

The components of a histogram metric are usually as follows:
- Counter with total number of measurements - "_count" suffix
- Counter with sum of values of measurements - "_sum" suffix
- Histogram buckets have "_bucket" suffix with `le label` (less than or equal to label) to denote upper bound of range

Buckets or Ranges in buckets are Upper Inclusive Bound, meaning the higest point in the range will also be included as a time-frame 

Example of such a metric is as follows:
```
# HELP prometheus_tsdb_compaction_chunk_range_seconds Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range_seconds histogram
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="100"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="400"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1600"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6400"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="25600"} 690
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="102400"} 1071
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="409600"} 186720
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1.6384e+06"} 186720
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6.5536e+06"} 187760
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="2.62144e+07"} 187760
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="+Inf"} 187760
prometheus_tsdb_compaction_chunk_range_seconds_sum 2.4053577485e+10
prometheus_tsdb_compaction_chunk_range_seconds_count 187760
```

Rate can be found from the sum and count from the above example:
```
sum(rate(prometheus_tsdb_compaction_chunk_range_seconds_sum[5m])) / sum(rate(prometheus_tsdb_compaction_chunk_range_seconds_count[5m]))      
```

#### Histogram_Quantile
Quantile is Percentile but instead of a 0 to 100 scale, it uses a 0 to 1 scale. This way, the 99th percentile or the 0.99th quantile can be found for a metric by:
```
histogram_quantile(0.99, rate(prometheus_tsdb_compaction_chunk_range_seconds_bucket[5m]))
```

The aggregation of histograms is also possible in Prometheus by writing queries like:
```
histogram_quantile(0.99, sum by (le) (rate(prometheus_tsdb_compaction_chunk_range_seconds_bucket[5m])))
```

**4. Summary**
   - Similar to Histograms, Summaries also help with measuring - In this sense they are an alternative to Histogram without needing to configure its buckets
   - Lower storage cost & best for monitoring accurate value latency (duration)
   - Cannot aggregate and came before Histogram Prometheus in
   - This metrics consists of 2 counter with `count` and `sum` suffix
   - **Use Cases**:
      - When measurement of many values is to be taken for calculating
      average
      - When approximation is not required and accuracy is needed
      - When range of values is not known before like histograms

An example of a summary metric is as follows:
```
# HELP prometheus_rule_group_duration_seconds The duration of rule group evaluations.
# TYPE prometheus_rule_group_duration_seconds summary
prometheus_rule_group_duration_seconds{quantile="0.01"} NaN
prometheus_rule_group_duration_seconds{quantile="0.05"} NaN
prometheus_rule_group_duration_seconds{quantile="0.5"} NaN
prometheus_rule_group_duration_seconds{quantile="0.9"} NaN
prometheus_rule_group_duration_seconds{quantile="0.99"} NaN
prometheus_rule_group_duration_seconds_sum 0
prometheus_rule_group_duration_seconds_count 0
```

## Querying With PromQL   
Millions of metrics will be stored in TSDB - How to aggregate metrics to understand how app is performing or answer questions like how much traffic is coming to app? Why can't we use SQL-like languages as well? SQL languages tend to lack expressive power when it comes to the sort of calculations you would like to perform on time series.

Use cases of PromQL:
    - Fetching metrics
    - Aggregating metrics
    - Building dashboards
    - Setup alerts

### Basics
**Value Data-Types**
 In Prometheus there are 3 primary datatypes used to represent metrics and their values

 | Type | Query | Result | Explanation	|	
|--|---|---------------------|-----------------|			
| Scalar | sum(http_server_requests_seconds_count) | 20 | Simple numeric floating point values - 20 is not associated with any timestamp as its an aggragated value |
| Instant Vector | http_server_requests_seconds_count | 20@1720606263 | Commonly used in promql queries to fetch current values or instant calc | 
| Range Vector | http_server_requests_seconds_count[1s] | 21@1720606263 22@1720606264 23@172060625 | List of values - range of samples until current time - Helps calculate rates of change, calculating, performing aggregations |

### Examples:

[Operators In Prometheus](https://prometheus.io/docs/prometheus/latest/querying/operators/)

| Criterion | PromQL |	Explanation	|	
|--|---|---------------------|-----------------|			
| Viewing a metric |prometheus_http_requests_total | In PromQL, its enough to write the metrics |	
| Filtering a metric | prometheus_http_requests_total{handler="/api/v1/metadata"} | Use {} to provide the label/key-value & operator like = or !=  | 
| Multiple-Filtering (AND) | prometheus_http_requests_total{handler="/api/v1/metadata", job="prometheus"} | Separate the label-values to be filtered with a comma |
| Multiple-Filtering (OR+Like) | prometheus_http_requests_total{handler=~"/api/v1/metadata|/metrics"} | No result shown - ISSUE |
| Multiple-Filtering (OR) | prometheus_http_requests_total{handler!~"/api/v1.+"} | Incorrect result shown - Equal value still being shown |

Additional querying using PromQL:
    - Finding metrics at a specifc unix time:
 ```
 prometheus_http_requests_total@1720606263
 ```
   - Finding metrics x d/s/m/h/etc ago **From Current Time**
 ```
 # If current time = 12:00 PM, value given will be AT 11:55 AM
 prometheus_http_requests_total offset 5m 
 ```
   - Finding metrics within a specific period **from now**

 ```
 # All values between 12:00 PM and 11:55 AM
 prometheus_http_requests_total[5m]    
 ```

**Some Common Functions**
Some [important functions](https://prometheus.io/docs/prometheus/latest/querying/functions/) that can be used while querying include:

sum()
When we run `prometheus_http_requests_total@1720606263, we get all metric-values at a particular time frame. What if we wish to aggregate them?
```
sum(prometheus_http_requests_total@1720606263)

{}                      119
```
**Date-Type**: Scaler because no timestamp is associated

#### Point of Confusion
 | Request | Response Time |	
|----------|---------------|			
| 10:01:00 | 10 | 	
| 10:01:30 | 15 |
| 10:02:00 | 30 | ---> Checking this

**read()**
- Helpful to know how fast or slow the values of metrics are changing
- Below is our Counter-Metrics, where values are cumulative and only increase:
```
rate(prometheus_http_requests_total[1m])

(30-10)/(1*60) = 0.33
(Difference b/w Highest & Lowest value within 1 min) / (60 seconds)
```

**irate()**
- Instead of taking last and first data points, irate considers its last and previous data point within the time-frame
- Used when we want to look at the sudden spikes or drops in graph by looking at the instant rate of change
- More instant response can be provided 
```
irate(prometheus_http_requests_total[1m])

(30-15) / (1*60) = 0.25
(Difference b/w highest & immediate previous value within 1 min) / (60 seconds) 
```

**increase()**
- To know the net change between a counter value over a specific period
- Similar to rate but gives an absolute value increase rather than a per second increase 
```
increase(prometheus_http_requests_total[1m]) 

(30-10) = 20 
(Difference b/w Highest & Lowest value within 1 min) 
```

Based on our communication in the morning, today I worked towards exploring the following PromQL-related functions for being able to fetch event aggregations:
max_over_range
min_over_range
increase()
delta()

Out of these, an arithmetic operation involving the `max_over_range` and `min_over_range` was utilised to fetch the difference within a time-range, in the following manner:

`max_over_time(prometheus_http_requests_total{handler="/metrics"}[1s] @1720681325) - min_over_time(prometheus_http_requests_total{handler="/metrics"}[1s] @1720681320)`
{code="200", handler="/metrics", instance="localhost:9090", job="prometheus"}                                                       5

## Setup
To begin using Prometheus for monitoring your systems, follow these steps for installation and setup:

1.Download Prometheus:

Visit the Prometheus download page (https://prometheus.io/download/) and choose the appropriate version for your operating system. For example, to download Prometheus for Linux, you can use the following command:

wget https://github.com/prometheus/prometheus/releases/download/v2.30.0/prometheus-2.30.0.linux-amd64.tar.gz

2.Extract the Tarball:

Once the download is complete, extract the tarball using the following command:

tar -xzf prometheus-2.30.0.linux-amd64.tar.gz

3.Navigate to the Prometheus Directory:

Enter the Prometheus directory that was extracted from the tarball:

cd prometheus-2.30.0.linux-amd64

4.Configure Prometheus:

Create a configuration file named prometheus.yml to define your Prometheus configuration. You can use a text editor like Nano or Vim to create this file:

nano prometheus.yml

Inside prometheus.yml, you can specify your scraping targets, alerting rules, and other configurations according to your monitoring needs.

5.Start Prometheus:

After configuring Prometheus, you can start the Prometheus server using the following command:

./prometheus --config.file=prometheus.yml

This command will start Prometheus with the configuration defined in prometheus.yml.

6.Access the Prometheus Web UI:

Open your web browser and navigate to http://localhost:9090 to access the Prometheus web user interface. Here, you can explore metrics, run queries using PromQL, and configure alerting rules.
======================================================='
