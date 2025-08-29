1. Construct a query to return all filesystems that have over 1000 bytes available on all instances under web job. Save the query in /root/query1.txt file.
- response: node_filesystem_avail_bytes{job="web"} > 1000

2. Which of the following queries you will use for loadbalancer:9100 host to return all the interfaces that have received less than or equal to 10000 bytes of traffic?
- response: node_network_receive_bytes_total{instance="loadbalancer:9100"} <= 10000

3. node_filesystem_files tracks the filesystem's total file nodes. Construct a query that only returns time series greater than 500000 and less than 10000000 across all jobs. Save the same in /root/query3.txt file.
- response: node_filesystem_files > 500000 and node_filesystem_files < 10000000

4. Which of the following queries can be used to track the total number of seconds cpu has spent in user + system mode for instance loadbalancer:9100?
- response: node_cpu_seconds_total{instance="loadbalancer:9100", mode="user"} + ignoring(mode) node_cpu_seconds_total{instance="loadbalancer:9100", mode="system"}

5. On loadbalancer:9100 instance, calculate the sum of the size of all filesystems. The metric to get filesystem size is node_filesystem_size_bytes. Save the query in /root/query6.txt file.
- response: sum(node_filesystem_size_bytes{instance="loadbalancer:9100"})

6. Your task is to construct a PromQL query that returns the number of CPU cores available on the instance loadbalancer:9100. Utilize the node_cpu_seconds_total metric and apply a filter for a specific mode (mode="idle") to ensure that each CPU core is counted only once. Once you have created this query, save it to the file located at /root/query7.txt file.
- response: count(node_cpu_seconds_total{instance="loadbalancer:9100", mode="idle"})

7. Construct a PromQL query that shows the number of CPUs on each instance across all jobs using the node_cpu_seconds_total metric. To count each CPU only once per instance, filter using a single mode (mode="idle"). Save this query in /root/query8.txt file.
- response: count(node_cpu_seconds_total{mode="idle"}) by (instance)

8. Use the node_network_receive_bytes_total metric to calculate the sum of the total received bytes across all interfaces on per instance basis. Save the query in /root/query9.txt file.
- response: sum(node_network_receive_bytes_total) by (instance)

9. Which of the following queries will be used to calculate the average packet size for each instance?
- response: sum by(instance) (node_network_receive_bytes_total)/ sum by(instance) (node_network_receive_packets_total)

10. To calculate the percentage in mode user, get the total seconds spent in mode user and divide that by the sum of the time spent across all modes. Further, multiply that result by 100 to get a percentage.
- response: node_cpu_seconds_total{mode="user"}*100 /ignoring(mode, job) sum by(instance, cpu) (node_cpu_seconds_total)

11. The api job collects metrics on an API used for uploading files. The API has 3 endpoints /images /videos, and /songs, which are used to upload respective file types. The API provides 2 metrics to track
http_uploaded_bytes_total - tracks the number of uploaded bytes.
http_upload_failed_bytes_total - tracks the number of bytes failed to upload.
Construct a query to calculate the percentage of bytes that failed for each endpoint. The formula for the same is http_upload_failed_bytes_total*100 / http_uploaded_bytes_total.

- response: http_upload_failed_bytes_total*100 / ignoring(error) http_uploaded_bytes_total