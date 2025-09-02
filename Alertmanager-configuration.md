```yml
# alertmanager.yml
global:
  smtp_smarthost: "mail.example.com:25"
  smtp_from: "test@example.com"
route:
  receiver: staff
  group_by: ['alertname','job']
  routes:
  - match_re: 
      job: (node|windows)
    receiver: infra-email
  - matchers:
      job: kubernetes
    receiver: k8s-slack
receivers:
- name: "k8s-slack"
  slack_configs:
  - channel: "#alerts"
    text: "https://exampl.com/alerts/{{ .GroupLabels.app }}/
```
Global section applies global configuration across all sections which can be overwritten. The route section provides a set of **rules** to determine what alerts get matched up with which receivers. A receiver contains one or more notifiers to forward alerts to users.

```yml
route:
  receiver: staff
  group_by: ['alertname','job']
  routes:
  - match_re: 
      job: (node|windows)
    receiver: infra-email
  - matchers:
      job: kubernetes
    receiver: k8s-slack
```
At the top level there is a fallback/default route. Any alerts that don't match any of the other routes will match the default route. Any alerts that don't match any of the other routes will match the default route. These alerts will be grouped by labels `alertname`, and `job`. These alerts will go to the receiver `staff`.

```yml
route:
  routes:
  - match_re: 
      job: (node|windows)
    receiver: infra-email
  - matchers:
      job: kubernetes
      severity: ticket
    receiver: k8s-slack
```
Every route has a list match statement and a receiver for alerts that match. In this example, all alerts with the `job=kubernetes` and `severity=ticket` will go to the receiver `k8s-slack`. If the match_re keyword is used instead of matchers, then a list of regular expressions can be provided to match specific labels. In this example, the rule will match any alert with a label of `job=node` or `job=windows`.

```yml
route:
  routes:
  - matcher: 
      job: kubernetes
    receiver: k8s-email
    routes:
    - matchers:
        severity: pager
      receiver: k8s-pager
```
All alerts with label of `job=kubernetes` will be send to receiver `k8s-email`. However we can configure sub-routes for further route matching. If the alert has a label `severity=pager` then it'll be sent to the receiver `k8s-pager`. All other alerts with label `job=kubernetes` will match the main route and be sent to the receiver `k8s-email`.

As with Prometheus, Alertmanager doesn't automatically pickup changes to its config file. One of the following must be performed, for changes to take effect:
- restart Alertmanager
- send a SIGHUP (sudo killall -HUP alertmanager)
- HTTP post to /-/reload endpoint

```yml
route:
  routes:
  - receiver: alert-logs
    continue: true
  - matcher: 
      job: kubernetes
    receiver: k8s-email
```
What if we want an alert to match two routes. In this example, we want all alerts to be sent to the receiver `alert-logs` and then if it has the label `job=kubernetes` it should also match the following line and be sent to `k8s-email` receiver. By default, the first matching route wins. So a specific alert would stop going down the route tree after the first match, to continue processing down the tree we can add `continue: true`.