**Console templates**

Console templates allow you to create your own custom html pages using Go templating language. Prometheus metrics, queries, and charts can be embedded in the templates.

1. Create a new console template called node-stats.html and add the standard header and footer templates.
Find more details below:
   (a) Console templates are located under /etc/prometheus/consoles/ directory.
   (b) Between the prom_content_head and prom_content_tail templates add an H1 tag with the text Node Stats to give this page a title.

- Solution: 
```shell
touch /etc/prometheus/consoles/node-stats.html
```
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
{{ template "prom_content_tail" }}
```

2. Update /etc/prometheus/consoles/node-stats.html template and add a section to include memory stats for the nodes. See below for additional details.
   (a) First, add an H3 header (under Node Stats) with the text Memory. This is where we will store all memory related stats
       <h3>Memory</h3>
   (b) Add a strong tag with the text Memory utilization: under Memory
      <strong>Memory utilization:</strong>
   (c) Next to the strong tag, add a prom_query_drilldown template to report the memory utilization of the nodes.
      The query for this is: 100- (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
{{ template "prom_content_tail" }}
```

3. Below the Memory utilization stats add a <br/> tag and then add a strong tag with the text Memory Size:.
To the right of it place another prom_query_drilldown template and render out the max/total memory the server has in Mb. The expression for this is:
```html
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
```

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
{{ template "prom_content_tail" }}
```

4. Let’s add another section for cpu based stats below memory stats:
   (a) Add an H3 tag with the text CPU.
     <h3>CPU</h3>
   (b) Next, add a strong tag with the text CPU Count: and then add a prom_query_drilldown to calculate the number of cpus on a node. The expression for this is: `count(node_cpu_seconds_total{mode='idle'})`

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
<h3>CPU</h3>
<strong>CPU Count:</strong> {{template "prom_query_drilldown" (args "count(node_cpu_seconds_total{mode='idle'})") }}
{{ template "prom_content_tail" }}
```

5. Let’s add one more cpu metric to calculate cpu utilization:
   (a) Add a break, i.e., <br/> after the previous metric.
   (b) Add a strong tag with the text CPU Utilization:
      <strong>CPU Utilization:</strong>
   (c) Add a prom_query_drilldown template to render out cpu utilization. The expression that you should use is:
       sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/8
Expression explanation: The expression will take the current rate of all cpu modes except idle because idle means cpu isn’t being used. It will then sum them up and multiply them by 100 to give a percentage. This final number is divided by 8 (if this server/node has 8 CPUs, we want to get the utilisation per CPU, so adjust this value as needed).

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
<h3>CPU</h3>
<strong>CPU Count:</strong> {{template "prom_query_drilldown" (args "count(node_cpu_seconds_total{mode='idle'})") }}
<br/>
<strong>CPU Utilization:</strong> {{template "prom_query_drilldown" (args "sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/16" "%") }}
{{ template "prom_content_tail" }}
```

6. Using the Graph library, let's graph out the cpu utilization on the nodes:
   (a) At the end of the template file, create a new div and give it an ID called cpu:
    <div id="cpu"></div>
   (b) Below the div, add a script tag so the javascript code can be executed
      <script>
      </script>
   (c) Within the script tag, create a new PromConsole.Graph object. You will need to pass in an object with two properties

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
<h3>CPU</h3>
<strong>CPU Count:</strong> {{template "prom_query_drilldown" (args "count(node_cpu_seconds_total{mode='idle'})") }}
<br/>
<strong>CPU Utilization:</strong> {{template "prom_query_drilldown" (args "sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/16" "%") }}
{{ template "prom_content_tail" }}
<div id="cpu"></div>
<script>
new PromConsole.Graph({
    node: document.querySelector("#cpu"),
    expr: "sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/2",
});
</script>
```

7. Next, you’ll add a new section for Network related stats and add a graph to track received bytes on all interfaces:
   (a) Add a new H3 tag with the text Network.
   (b) Below the H3 tag, add a div with an id called network
   (c) Within the script tag, create a new PromConsole.Graph object. You will need to pass in an object with two properties

- Solution:
```html
{{ template "prom_content_head" }}
<h1>Node Stats</h1>
<h3>Memory</h3>
<strong>Memory utilization:</strong>
{{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100)") }}
<br/>
    <strong>Memory Size:</strong> {{template "prom_query_drilldown" (args
    "node_memory_MemTotal_bytes/1000000" "Mb") }}
<h3>CPU</h3>
<strong>CPU Count:</strong> {{template "prom_query_drilldown" (args "count(node_cpu_seconds_total{mode='idle'})") }}
<br/>
<strong>CPU Utilization:</strong> {{template "prom_query_drilldown" (args "sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/16" "%") }}
{{ template "prom_content_tail" }}
<div id="cpu"></div>
<script>
new PromConsole.Graph({
    node: document.querySelector("#cpu"),
    expr: "sum(rate(node_cpu_seconds_total{mode!='idle'}[2m]))*100/2",
});
</script>
<h3>Network</h3>
<div id="network"></div>
<script>
new PromConsole.Graph({
    node: document.querySelector("#network"),
    expr: "rate(node_network_receive_bytes_total[2m])",
});
</script>
```