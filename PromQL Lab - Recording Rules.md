1. Create a recording rule to track the rate at which a node is receiving traffic. Find below more details:
  (a) Create a file called node-rules.yaml under /etc/prometheus directory.
  (b) Update this file to:
        (i) Create a group called node, this group should have all the rules for node_exporters.
        (ii) Set the interval for the rules to run every 15s.
        (iii) Add a record called node_network_receive_bytes_rate.
        (iv) The expression should be rate(node_network_receive_bytes_total{job="nodes"}[2m])
  (c) Finally, update the prometheus.yml file to import rules from node-rules.yaml file and restart the prometheus service.

Solution:
```yaml
# /etc/prometheus/node-rules.yaml
groups:
  - name: node
    interval: 15s
    rules:
      - record: node_network_receive_bytes_rate
        expr: rate(node_network_receive_bytes_total{job="nodes"}[2m])
```
```yaml
# /etc/prometheus/prometheus.yml    
...
rule_files:
    - node-rules.yaml
...
```
```sh
systemctl restart prometheus
```

2. Update the node group you created earlier to add another rule to get the average rate of received packets across all interfaces on a node, as each node has 2 interfaces. Please find below more details:
  (a) The record name should be node_network_receive_bytes_rate_avg.
  (b) The expression should be:
```sh
avg by(instance) (node_network_receive_bytes_rate)
```

Solution:
```yaml
# /etc/prometheus/node-rules.yaml
groups:
  - name: node
    interval: 15s
    rules:
      - record: node_network_receive_bytes_rate
        expr: rate(node_network_receive_bytes_total{job="nodes"}[2m])
      - record: node_network_receive_bytes_rate_avg                     # Newly added
        expr: avg by(instance) (node_network_receive_bytes_rate)        # Newly added
```

3. Update the node group to add another rule to track the filesystem free space percentage. Find below more details:
   (a) The record name should be node_filesystem_free_percent.
   (b) The expression should be 100 * node_filesystem_free_bytes{job="nodes"} / node_filesystem_size_bytes{job="nodes"}
   (c) Restart the prometheus service.

Solution:
```yaml
# /etc/prometheus/node-rules.yaml
groups:
  - name: node
    interval: 15s
    rules:
      - record: node_network_receive_bytes_rate
        expr: rate(node_network_receive_bytes_total{job="nodes"}[2m])
      - record: node_network_receive_bytes_rate_avg
        expr: avg by(instance) (node_network_receive_bytes_rate)
      - record: node_filesystem_free_percent
        expr: 100 * node_filesystem_free_bytes{job="nodes"} / node_filesystem_size_bytes{job="nodes"}
```
```sh
systemctl restart prometheus
```

4. Create a new file called api-rules.yaml under /etc/prometheus/ directory. Update it to configure below rule(s).
  (a) Add a group called api.
  (b) Set the interval to 15s
  (c) Add a rule that will track the average latency for past 2 minutes:
        (i) The record name should be avg_latency_2m.
        (ii) The expression should be rate(http_request_total_sum{job="api"}[2m]) /
        rate(http_request_total_count{job="api"}[2m])
  (d) Update the prometheus.yml file to include the new rules file i.e api-rules.yaml.
  (e) Restart the Prometheus service.

Solution:
```yaml
# /etc/prometheus/api-rules.yaml
groups:
  - name: api
    interval: 15s
    rules:
      - record: avg_latency_2m
        expr: rate(http_request_total_sum{job="api"}[2m]) / rate(http_request_total_count{job="api"}[2m])
```
```yaml
# /etc/prometheus/prometheus.yml
...
rule_files:
    - node-rules.yaml
    - api-rules.yaml    # Newly added
...
```
```sh
systemctl restart prometheus
```

