<div align=center>
NOTE: This document is currently a work-in-progress and its content and formatting are liable to major changes everyday, until its completion.  

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
- **Target**: Metric will be of something - Where is the metric being taken from? - Ex: My laptop, a Game Server, an App, etc. 
- **Scraping Interval**: How relentlessly/persistently the metrics are being scraped/collected - Ex: 2s, 8h, 6d, 11w, 1y, etc.
- **Database**: Time Series Database stores data with timestamps - Ex: Values of a metric will be collected at different timestamps

We can also consider an [Odometer](https://media.istockphoto.com/id/1398823521/vector/electric-counter-electric-meter-with-numbers-display-of-odometer-counter-of-kilowatt-energy.jpg?s=1024x1024&w=is&k=20&c=SEBWLgQxMS9M-ndi9AM6Am5_KyKLasJMAM9kRftDEL0=) and PEMS' Superset as a day-to-day real life example of a Monitoring tool for a better understanding of Prometheus:
> When a car moves forward, the distance covered is constantly recorded and visualised through the Odometer 
> PEMS' Superset visualises (Not records!) Tickets' from Redmine at every 24 hour interval using the tickets 

The following similarities can be seen in the examples provided above:
<div align=center>

| Term | Prometheus |  Odometer | Superset |	
|------|----------|---------|------------|	
| Monitoring Tool | Prometheus  | Fitness Tracker | Superset |
| Target          | Laptop  | Car | Redmine |
| Metrics         | node_filesystem_avail_bytes | Distance Travelled  | Tickets |
| Scraping Interval| 1h  | No Intervals | 24h |
| TSDB        | Is a [Time Series Database](https://www.reddit.com/r/Database/comments/1ayaj1b/time_series_database/) | Not a time-series database as timestamps are not recorded | Not adding data *to* but pulling *from* a database for visualisation. Note: Superset is a visualisation tool, not a Monitoring tool |
</div>

In the following section, we will be better understanding the basics of how everything works within Prometheus.

## Architecture of Prometheus

![img](https://i.imgur.com/VvNlrv7.gif)

Types of Metrics

**1. Counter**
   - A metrics type where the value only gets incremented or reset and the
   [only type that works with RATE](https://stackoverflow.com/questions/66674880/understanding-of-rate-func=tion-of-promql)
      -  Still unclear on WHY rate() only works with Counter
   - **Example**: Height as it only increments or the number of requests made
   to a server
   - **Use Cases**:
      - When a value that only goes up is to be recorded
      - When you want to know how fast/slow the value is increasing over
      time
      - When cumulative data is required

**2. Gauge**
   - A metric type used for values that can increase or decrease over time
   - **Example**: Weight as it can increase & decrease or the Memory usage
   - **Use Cases**:
      - When a value that can increase or decrease or fluctuate over time
      is to be recorded
      - When the recording of rate is not required - Further research
      required

**3. Histogram**
   - A metric type that records frequency of events over a time range or
   buckets (predefined ranges to group metrics)
   - **Example**: Tracking time taken to start PC daily after shutdown or
   tracking http request duration - In former, bucket could be **known**
   range of weight time like 0-5 Min, 5-10 Min, etc.
   - **Use Cases**:
      - When approximation and slight inaccuracy of metrics is not an issue
      - When many measurements of a value is to be taken for averaging
      later on - Sum of count and time is calculated by default
      - When cumulative data and previous values being added with current
      one is not an issue

**4. Summary**
   - Further understanding of Quantiles is required to better understand
   this Metrics-type.
   - **Use Cases**:
      - When measurement of many values is to be taken for calculating
      average
      - When approximation is not an issue
      - When range of values is not known before like histograms

## Understanding PromQL   
Millions of metrics will be stored in TSDB - How to aggregate metrics to understand how app is performing or answer questions like how much traffic is coming to app?    

### Use cases of PromQL:
    - Fetching metrics
    - Aggregating metrics
    - Building dashboards
    - Setup alerts
    
### Comparison with SQL:

[Operators In Prometheus](https://prometheus.io/docs/prometheus/latest/querying/operators/)

| Criterion | SQL | PromQL |	Explanation	|	
|--|---|---------------------|-----------------|			
| Viewing a metric | SELECT * FROM prometheus_http_requests_total | prometheus_http_requests_total | In PromQL, its enough to write the metrics |	
| Filtering a metric | SELECT * FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" | prometheus_http_requests_total{handler="/api/v1/metadata"} | Use {} to provide the label/key-value & operator like = or !=  | 
| Multiple-Filtering (AND) | SELECT * FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" AND job="prometheus" | prometheus_http_requests_total{handler="/api/v1/metadata", job="prometheus"} | Separate the label-values to be filtered with a comma |
| Multiple-Filtering (OR+Like) | SELECT * FROM prometheus_http_requests_total WHERE handler LIKE "/api/v1/%" handler LIKE OR handler LIKE "/api/v2/%" | prometheus_http_requests_total{handler=~"/api/v2...|api/v1..."} | No result shown - ISSUE |
| Multiple-Filtering (OR) | SELECT * FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" | prometheus_http_requests_total{handler!~"/api/v1.."} | Incorrect result shown - Equal value still being shown |

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

**Data-Types**
 In Prometheus there are 3 primary datatypes used to represent metrics and their values
 
 | Type | Query | Result | Explanation	|	
|--|---|---------------------|-----------------|			
| Scalar | sum(http_server_requests_seconds_count) | 20 | Simple numeric floating point values - 20 is not associated with any timestamp as its an aggragated value |
| Instant Vector | http_server_requests_seconds_count | 20@1720606263 | Commonly used in promql queries to fetch current values or instant calc | 
| Range Vector | http_server_requests_seconds_count[1s] | 21@1720606263 22@1720606264 23@172060625 | List of values - range of samples until current time - Helps calculate rates of change, calculating, performing aggregations |

**Functions**
Some [important functions](https://prometheus.io/docs/prometheus/latest/querying/functions/) that can be used while querying include:

sum()
When we run `prometheus_http_requests_total@1720606263, we get all metric-values 
at a particular time frame. What if we wish to aggregate them?
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
- Below is our Counter-Metrics, where values are cumulative and only increase : 
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

Within the context of Gauge Metric-Type, the following is the output of trying to find the values of a Metric sum

https://dev.to/sre_panchanan/decoding-promql-a-deep-dive-into-prometheus-query-language-4h23#scalar
https://promlabs.com/blog/2020/06/18/the-anatomy-of-a-promql-query/
Prometheus instant vector vs range vector - https://stackoverflow.com/questions/68223824/prometheus-instant-vector-vs-range-vector
