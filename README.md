<div align=center>

![img](https://i.imgur.com/GzEatYl.png)
# Understanding Prometheus: An Introduction
</div>

Metrics
> When I was young I wanted to grow tall and to measure it I used height as a metric. I asked my dad to measure my height everyday and keep a table of my height on each day. Here, Father = Monitoring System & My Height = Metric

Similar analogy can be made for Measuring water consumption, weight,  

Metric in Prometheus is a standard for measurement OVER TIME. The last 2 words also help explain Prometheus being a Time-Series Database, where data is collected with Timestamps. 

Types of Metrics

**1. Counter**

   - A metrics type where the value only gets incremented or reset and the
   only type that works with RATE
   <https://stackoverflow.com/questions/66674880/understanding-of-rate-func=
tion-of-promql>
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
| Viewing a metric | SELECT ** FROM prometheus_http_requests_total | prometheus_http_requests_total | In PromQL, its enough to write the metrics |	
| Filtering a metric | SELECT ** FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" | prometheus_http_requests_total{handler="/api/v1/metadata"} | Use {} to provide the label/key-value & operator like = or !=  | 
| Multiple-Filtering (AND) | SELECT ** FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" AND job="prometheus" | prometheus_http_requests_total{handler="/api/v1/metadata", job="prometheus"} | Separate the label-values to be filtered with a comma |
| Multiple-Filtering (OR+Like) | SELECT ** FROM prometheus_http_requests_total WHERE handler LIKE "/api/v1/%" handler LIKE OR handler LIKE "/api/v2/%" | prometheus_http_requests_total{handler=~"/api/v2...|api/v1..."} | No result shown - ISSUE |
| Multiple-Filtering (OR) | SELECT ** FROM prometheus_http_requests_total WHERE handler="/api/v1/metadata" | prometheus_http_requests_total{handler!~"/api/v1.."} | Incorrect result shown - Equal value still being shown |

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
When we run prometheus_http_requests_total@1720606263, we get all metric-values 
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


