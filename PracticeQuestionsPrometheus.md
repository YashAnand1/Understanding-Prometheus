 
# Counter Type Queries

All of these questions are related to a **Counter-Type** Metric called **go_memstats_alloc_bytes_total**.

1. What is the total allocated memory over the last 1 hour?
```
// I could not use sum() directly because it does not allow range but sum_over_time does & also sums values
sum(sum_over_time(go_memstats_alloc_bytes_total[5m]))
```

2. How much memory has been allocated in the last 5 minutes?
```
sum(sum_over_time(go_memstats_alloc_bytes_total[5m]))
```

3. What is the average rate of memory allocation per second over the last 10 minutes?
```
avg(rate(go_memstats_alloc_bytes_total[10m]))
```

4. Find the peak memory allocation rate in the last 15 minutes
```
max(rate(go_memstats_alloc_bytes_total[15m]))
```

5. Determine the increase in allocated memory over the last 24 hours
```
sum(increase(go_memstats_alloc_bytes_total[24h]))
```

6. Calculate the memory allocation rate per minute over the past 30 minutes
```
Rate per minute - multiply by 60 
rate(go_memstats_alloc_bytes_total[24h]) * 60
```

7. Show the total allocated memory per instance for the past 1 hour.
```
// https://stackoverflow.com/questions/70350439/promql-how-combine-sum-over-time-with-by <==== An actual "Workaround"
sum(sum_over_time(go_memstats_alloc_bytes_total{instance="localhost:9090"}[1h]))
```

8. What is the total memory allocated by each job in the last 6 hours?
```
sum by (job) (go_memstats_alloc_bytes_total[6h])
```

9. Compare the memory allocation rate between two instances for the last 1 hour.
```
rate(go_memstats_alloc_bytes_total{instance="localhost:9090"|"localhost:9999"}[1h])
```

10. Find the highest memory allocation spike in the last 7 days.
```
max_over_time(go_memstats_alloc_bytes_total[7d])
```

11. Display the memory allocation rate in the last 10 minutes as a graph.
```
max_over_time(go_memstats_alloc_bytes_total[7d])
```

12. What is the memory allocation rate per second for the last 1 hour, grouped by instance?
```
rate(go_memstats_alloc_bytes_total[1h])
```

13. Show the cumulative memory allocated in the past 24 hours.
```
sum(increase(go_memstats_alloc_bytes_total[24h]))
```

14. Identify the instances with the highest memory allocation rate in the last 12 hours.
```
max by (instance) (rate(go_memstats_alloc_bytes_total[12h]))
```

15. Calculate the rolling average of the memory allocation rate over the last 30 minutes.
```
avg(rate(go_memstats_alloc_bytes_total[30m])) by (instance)
```
16. Display the change in memory allocation over the past 3 hours, grouped by job.
```
sum by (job) (increase(go_memstats_alloc_bytes_total[3h]))
```

17. What is the 90th percentile of memory allocation rates in the last 2 hours?
```
// SImilar to this https://stackoverflow.com/questions/70881971/how-to-get-the-95th-percentile-of-an-average-in-prometheus
quantile(0.90, rate(go_memstats_alloc_bytes_total[2h]))
```
18. Show the memory allocation rate for the top 5 instances over the last 1 hour.
```
// Still looking into this
```

19. Find the average memory allocation per instance in the last 24 hours.
```
avg by (instance) (increase(go_memstats_alloc_bytes_total[24h]))
```
20. Calculate the memory allocation rate per second over the past week.
```
rate(go_memstats_alloc_bytes_total[1w])
```
