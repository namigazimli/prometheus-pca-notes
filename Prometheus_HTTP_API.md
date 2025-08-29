**HTTP API**

Prometheus has an HTTP API that can be used to execute queries as well as gather information on alert, rules, and service-discovery related configs. This is useful for situations where using the built-in web gui isn't an option.
- Building custom tools
- 3rd party integrations
To run a query through the API send a POST request to:
```sh
http://<prometheus-ip>/api/v1/query
```
The query is then passed into the query parameter. For example, we have a query like following:
```sh
node_arp_entries{instance="192.168.1.100:9100"}}
```
To send this query throug the API:
```sh
curl <prometheus-ip>:9090/api/v1/query --data 'query=node_arp_entries{instance="192.168.1.100:9100"}'
``` 
If you want to perform query at a specific time, you have to add `--data 'time=1670380680.123'` option to the command.
If you want to return a range vector, you have to write command like this:
```sh
curl <prometheus-ip>:9090/api/v1/query_range --data 'query=node_arp_entries{instance="192.168.1.100:9100"}[5m]' --data 'start=1670380680.123
```
