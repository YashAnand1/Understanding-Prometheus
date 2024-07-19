# Resources

PromQL:
https://isitobservable.io/observability/prometheus/how-to-build-a-promql-prometheus-query-language
https://promlabs.com/blog/2020/06/18/the-anatomy-of-a-promql-query/
https://www.youtube.com/watch?v=ewA8hb0w314&list=PLrMP04WSdCjrL4OBnaqXRy8X3XEd7ZrKf&index=13
https://valyala.medium.com/promql-tutorial-for-beginners-9ab455142085
https://prometheus.io/docs/prometheus/latest/querying/basics/
https://prometheus.io/docs/prometheus/latest/querying/operators/
https://promlabs.com/blog/2021/01/29/how-exactly-does-promql-calculate-rates/
https://stackoverflow.com/questions/40729406/most-recent-value-or-last-seen-value
https://stackoverflow.com/questions/70798035/increase-vs-changes-function-for-counters
https://promlabs.com/promql-cheat-sheet/

Metrics
https://logz.io/learn/prometheus-metrics-guide/#Conclusion
https://chronosphere.io/learn/an-introduction-to-the-four-primary-types-of-prometheus-metrics/
https://www.timescale.com/blog/four-types-prometheus-metrics-to-collect/
https://last9.io/blog/prometheus-metrics-types-a-deep-dive/#:~:text=Prometheus%20primarily%20deals%20with%20four,temperature%20or%20current%20memory%20usage.

Understanding Prometheus
https://www.youtube.com/watch?v=fhx0ehppMGM&list=PLyBW7UHmEXgylLwxdVbrBQJ-fJ_jMvh8h&index=6
https://blog.palark.com/prometheus-architecture-tsdb/
Summing Distinct Values is a requested feature: https://github.com/prometheus/prometheus/issues/9822
How querying works https://www.timescale.com/blog/how-prometheus-querying-works-and-why-you-should-care/
https://theswissbay.ch/pdf/Books/Computer%20science/prometheus_upandrunning.pdf
https://learning.oreilly.com/library/view/prometheus-up/9781492034131/ch14.html#idm45497354458304

Extras
https://stackoverflow.com/questions/64318141/promql-sum-over-time
https://stackoverflow.com/questions/54494394/do-i-understand-prometheuss-rate-vs-increase-functions-correctly

URLs:
http://localhost:9090/ will be redirected to http://localhost:9090/graph
http://localhost:9090/metrics 
http://localhost:9090/api/v1/query_range?query=sum_over_time(go_memstats_alloc_bytes[9s])&start=1720681324&end=1720681324&step=1
