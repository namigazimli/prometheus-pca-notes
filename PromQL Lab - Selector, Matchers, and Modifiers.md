1. Construct and run a query that will return all instances that are in up state and that are part of the web job. Save the query in /root/query2.txt file.
- response: up{job="web"}

2. node_memory_MemAvailable_bytes query returns the amount of the memory available on the target(s). Construct a query to return the available memory on all target(s) under loadbalancer job. Save the query in /root/query3.txt file.
- response: node_memory_MemAvailable_bytes{job="loadbalancer"}

3. Which of the following queries will return the number of arp entries for node01:9100 node?
- response: node_arp_entries{instance="node01:9100"}

4. Which of the following queries will return the number of bytes received on interface eth1 on instance node01:9100?
- response: node_network_receive_bytes_total{instance="node01:9100", interface="eth1"}

5. Each host includes a loopback interface i.e lo. Construct a query to return the number of bytes received on all interfaces except interface lo on instance node02:9100. Save the query in /root/query6.txt file.
- response: node_network_receive_bytes_total{instance="node02:9100", device!="lo"}

6. Using node_memory_MemAvailable_bytes metric, construct a query to return the available memory bytes for past 5 minutes on node01:9100. Save the query in /root/query7.txt file.
- response: node_memory_MemAvailable_bytes{instance="node01:9100"}[5m]

7. Which of the following queries will return the 1h ago available memory bytes on node01:9100 host?
- response: node_memory_MemAvailable_bytes{instance="node01:9100"} offset 1h

8. Which of the following queries will return the value of the metric on timestamp 1654920221 i.e Jun 11, 2022 4:03:41 AM GMT for instance node02:9100?
- response: node_context_switches_total{instance="node02:9100"}@1654920221

9. Which of the following queries will return the value of the metric 30 minutes before Jun 11, 2022 4:03:41 AM GMT for instance node01:9100? The unix timestamp value for this time is 1654920221.
- response: node_context_switches_total{instance="node01:9100"}@1654920221 offset 30

10. The node_cpu_seconds_total metric has two labels. One is the cpu, which denotes for which cpu the time series is for and the other one is mode, which denotes the cpu operating mode. Construct a query that will return a metric for all instances in the web job for cpu 0 only and the cpu mode can be either user or system. To match user or system, a regular expression like user|system will be used.

- response: node_cpu_seconds_total{job="web", cpu="0", mode=~"user|system"}