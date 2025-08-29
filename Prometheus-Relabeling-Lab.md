1. For the demo job, configure re-label configs to scrape only targets with env="prod" label and drop all other targets. After making all required changes, restart the prometheus service using below command.
```sh
systemctl restart prometheus.service
```

- Solution:
```yml
scrape_configs:
- job_name: demo
  relabel_configs:
  - source_labels: env
    regex: prod
    action: keep
```

2. We have decided to scrape the metrics from targets that have the following labels only:
- team=api
- env=prod
Make the required changes for demo job. After making all required changes, restart the prometheus service using below command.
```sh
systemctl restart prometheus.service
```

- Solution:
```yml
scrape_configs:
- job_name: demo
  relabel_configs:
  - source_labels: [env, team]
    regex: prod;api
    action: keep    
```

3. Currently, there is a label that follows the format `team=<team-name>`:
- team=`api`
- team=`database`
Re-label this label so the label name changes to the organization and the value gets prepended with org- text.
Example:
- organization=`org-api`
- organization=`org-database`
Make the required changes for demo job.

- Solution:
```yml
scrape_configs:
- job_name: demo
  relabel_configs:
  - source_labels: [team]
    regex: .*
    target_label: organization
    replacement: org-$1
    action: replace
```

4. The `type` label is no longer needed, set up a **relabel policy** for the `demo` job to drop this label.

- Solution:
```yml
scrape_configs:
- job_name: demo
  relabel_configs:
  - regex: type
    action: labeldrop
```

5. By default, the labels that start with `__` will get dropped after the re-labeling process. Now, we want to keep all of those labels as well and change the name so it removes the `__meta_` text from the name.
For example change:
- __meta_os__=centos ⇒ os=centos
- __meta_mem__=8000mb ⇒ mem=8000mb
Use the `labelmap` action to assign these `discovered` labels as `target` labels. Make the required changes under `demo` job.

- Solution:
```yml
scrape_configs:
- job_name: demo
  relabel_configs:
  - regex: __meta_(.*)
    action: labelmap
    replacement: $1
```

6. The Prometheus metric `node_network_transmit_drop_total` is no longer needed for monitoring. Your task:
Configure Prometheus so that under the job with `job_name: "nodes"`, add a metric relabeling policy to drop the `node_network_transmit_drop_total` metric.

- Solution:
```yml
scrape_configs:
- job_name: "nodes"
  metric_relabel_configs:
  - source_labels: [__name__]
    regex: node_network_transmit_drop_total 
    action: drop
```

7. Update the Prometheus configuration to rename the metric `node_network_receive_bytes_total` to `node_network_rx_bytes_total` for all targets under the `nodes` job. Under the job with `job_name: "nodes"`, add a `metric_relabel_configs` rule to rename `node_network_receive_bytes_total` to node_network_rx_bytes_total.

- Solution:
```yml
scrape_configs:
- job_name: "nodes"
  metric_relabel_configs:
  - source_labels: [__name__]
    regex: node_network_receive_bytes_total
    target_label: __name__
    replacement: node_network_rx_bytes_total
    action: replace
```

8. In the Prometheus configuration, remove the `fstype` label from the `node_filesystem_files` metric for all targets in the nodes scrape job. In the scrape job with `job_name: "nodes"`, add a metric relabel configuration to drop the `fstype` label from the `node_filesystem_files` metric.

- Solution:
```yml
scrape_configs:
- job_name: "nodes"
  metric_relabel_configs:
  - regex: fstype
    action: labeldrop
```

9. For all network-related metrics such as `node_network_receive_bytes_total`, the `device` label indicates the network interface (e.g., eth0, enp0s3). In the scrape job with `job_name: "nodes"`, you are required to:
- Rename the `device` label to `interface`.
- Remove the original device label, ensuring only the new interface label is present.

- Solution:
```yml
scrape_configs:
- job_name: "nodes"
  metric_relabel_configs:
  - source_labels: [device]
    regex: (.*)
    target_label: interface
    action: replace
    replacement: $1
  - regex: device
    action: labeldrop
```