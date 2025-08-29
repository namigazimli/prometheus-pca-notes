1. There are three jobs configured in Prometheus: `multimedia`, `auth`, and `api`. Construct a Prometheus query to fetch the `node_cpu_seconds_total` metric for all jobs and sort the results in ascending order. Save this query to the file `/root/ascending.txt`.
- response: 
```sh
sort(node_cpu_seconds_total)
```

2. Construct a Prometheus query that returns the metric `node_memory_Active_bytes` for all jobs, and sorts the results in descending order. Save this query to the `/root/descending.txt` file
- response: 
```sh
sort_desc(node_memory_Active_bytes)
```

3. Calculate the percentage of free space for all filesystems on all instances under all jobs using the following query:
```bash
node_filesystem_avail_bytes * 100 / node_filesystem_size_bytes
```
Note: The resulting percentages may have several decimal places. Now, modify the query to use the round function to round the result to the nearest integer. Save the final rounded query in the `/root/percentage.txt` file. 
- response: 
```sh
round(node_filesystem_avail_bytes * 100 / node_filesystem_size_bytes)
```

4. In the Prometheus expression browser, use the following query to get the number of context switches on `node01:9100`:
```bash
node_context_switches_total{instance="node01:9100"}
```
Click the Graph button to visualize the context switches over time. You’ll notice that, as a counter, the value only increases, which is expected but not as useful. Instead, what is useful is the rate at which context switches are occurring. Construct a Prometheus query to show the rate of context switches using a `2-minute` window. Save this query to the file `/root/context.txt`.
- response: 
```sh
rate(node_context_switches_total{instance="node01:9100"}[2m])
```

5. Management wants to monitor the rate of bytes received by each instance. Since each instance has two network interfaces, the rate of incoming traffic must be combined (summed) for all interfaces per instance.
    - Calculate the rate of received bytes using the `node_network_receive_bytes_total` metric and a `2-minute` window.
    - Sum the rates for all interfaces and group the result by instance.
    - Save the final query in `/root/traffic.txt`.
- response: 
```sh
sum(rate(node_network_receive_bytes_total[2m])) by (instance)
```

6. There have been reports of a brief application outage, and alerts are indicating high CPU iowait during the incident. To better understand this event, management wants to know the peak rate at which all CPUs, combined, were spending time in iowait mode over the past 10 minutes. Your task is to:
    - Use the metric `node_cpu_seconds_total` with the label filter `mode="iowait"`.
    - Calculate the per-second rate of iowait using the rate() function with a 1-minute window.
    - Use a Prometheus subquery with a 10-minute range and a 30-second step (`[10m:30s]`) to evaluate the rate at regular intervals.
    - Apply `max_over_time()` to this subquery to find the highest observed total iowait rate over the 10-minute window.
    - Save your query in `/root/iowait.txt`.
- response: 
```sh
max_over_time(rate(node_cpu_seconds_total{mode="iowait"}[1m])[10m:30s]) 
```

7. The application exposes a histogram metric for HTTP requests, with the total count of requests available from the corresponding _count time series, called `http_request_total_count`.
    - Construct a PromQL query that returns the total number of HTTP requests handled by the instance `node01:3000` using the `http_request_total_count` metric.
    - Save your query in the file `/root/http.txt`.
- response: 
```sh
sum(http_request_total_count{instance="node01:3000"})
```

8. The application tracks HTTP request durations using a Prometheus histogram metric called `http_request_total_bucket`. Each time series for this metric includes:
- a route label (indicating the endpoint),
- an le label (representing the upper bound on latency for each bucket).
Your task is to:
Write a PromQL query that returns the cumulative number of HTTP requests for the `/events` route that completed in less than `0.4` seconds, across all nodes.
    - Use the `http_request_total_bucket` metric.
    - Select only the series where `route="/events"` and `le="0.4"`.
    - Save your query in the file `/root/events.txt`.
- response: 
```sh
sum(http_request_total_bucket{route="/events", le="0.4"})
```

9. The application exposes request latency as a Prometheus histogram metric named http_request_total_bucket, which counts the total number of requests completed within a given time bucket, specified by the le (less than or equal) label. Your task is to:
    - Construct a PromQL query to get the exact number of HTTP requests on instance node02:3000 that took longer than 0.08 seconds but less than or equal to 0.1 seconds.
    - To achieve this, subtract the count of requests in the bucket with `le="0.08"` from the count in the bucket with `le="0.1"`. The result is the number of requests whose duration was >0.08s and ≤0.1s.
    - Since the `le` label differs between the two time series, use the `ignoring(le)` clause to prevent label mismatch errors.
    - Save your query in the file `/root/requests.txt`.
- response: 
```sh
http_request_total_bucket{instance="node02:3000", le="0.1"} - ignoring(le) http_request_total_bucket{instance="node02:3000", le="0.08"}
```

10. The application exposes request latencies as a Prometheus histogram metric called http_request_total_bucket, which counts the cumulative number of HTTP requests completed within a given latency bucket, indicated by the le (less than or equal) label. Your task is to:
Construct a PromQL query that calculates the per-second rate of HTTP requests (using a 1-minute window) whose latency was less than 0.08 seconds, across all nodes.
    - Use the bucket for le="0.08" in your selector to include all requests that completed in less than or equal to 0.08 seconds.
    - Apply the rate() function to the selected time series over a 1-minute window.
    - Save your query in the file /root/rate.txt.
- response: 
```sh
rate(http_request_total_bucket{le="0.08"}[1m])
```

11. Construct a PromQL query to calculate the average latency of a request over the past 4 minutes.
    - Use the metric http_request_total_sum, which contains the sum of all request latencies, and http_request_total_count, which contains the total number of requests.
```sh
rate of sum-of-all-requests / rate of count-of-all-requests
```
    - Save your query in /root/request_sum.txt.
- response:
```sh
rate(http_request_total_sum[4m]) / rate(http_request_total_count[4m])
```

12. Management would like to know the 95th percentile latency for HTTP requests received by node `node01:3000`.
The application exposes latencies as a Prometheus histogram metric named `http_request_total_bucket`.
    - Construct a PromQL query using the `histogram_quantile` function to calculate the `95th` percentile latency on `node01:3000`.
    - Save your query in `/root/quantile.txt`.
- response:
```sh
histogram_quantile(0.95, http_request_total_bucket{instance="node01:3000"})
```

13. The application exposes uploaded bytes as a Prometheus summary metric named `http_upload_bytes`, which reports quantiles via a `quantile` label.
    - Construct a PromQL query to retrieve the `90th` percentile `quantile="0.9"` of uploaded bytes for node `node01:3000`.
    - Save your query in `/root/percentile.txt`.
- response:
```sh
quantile(0.9, http_upload_bytes{instance="node01:3000"})
```