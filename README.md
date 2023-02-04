# Observability
In control theory, observability is defined as a measure of how well the internal states of a system can be inferred from knowledge of its external outputs
# Monitoring
Collecting, processing, aggregating and displaying real-time quantitative data about a system
# Why Monitor?
- Alerting
- Debugging
- Trending

A Google SRE team with 10–12 members typically has one or sometimes two members whose primary assignment is to build and maintain monitoring systems

# The Four Golden Signals
- Latency
- Traffic
- Errors
- Saturation

# Categories of Monitoring
- Profiling
- Tracing
- Logging
- Metrics

# Prometheus
written in Go

2012 at SoundCloud

In 2016 became the second member of the CNCF

# Prometheus Architecture

Image

- Client Libraries
- Exporters
- Service Discovery
- Scraping
  - Pull vs Push
- Storage
- Dashboards
  - expression browser
  - Grafana
- Recording Rules and Alerts
- Alert Management
- Long-Term Storage
  - default retention for Prometheus is 15 days
  
# Prometheus is not
- for storing event logs
- for high cardinality data such as email addresses or username
- for money or billing (small inaccuracies and race conditions are a fact of life)

# Getting Started with Prometheus
prometheus.yml contain the following text:
```yaml
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
        - localhost:9090
```
```bash
$ wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
$ tar -xzf prometheus-*.linux-amd64.tar.gz
$ cd prometheus-*.linux-amd64/
$ ./prometheus
```

## Prometheus ui
open http://localhost:9090/

```promql
up
process_resident_memory_bytes
prometheus_tsdb_head_samples_appended_total
rate(prometheus_tsdb_head_samples_appended_total[1m])
```
## Prometheus metrics
open http://localhost:9090/metrics

# Running the Node Exporter
```bash
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
$ tar -xzf node_exporter-*.linux-amd64.tar.gz
$ cd node_exporter-*.linux-amd64/
$ ./node_exporter
```

## Node Exporter metrics
open http://localhost:9100/metrics

prometheus.yml contain the following text:
```yaml
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
        - localhost:9090
  - job_name: node
    static_configs:
      - targets:
        - localhost:9100
```

# Alerting
prometheus.yml contain the following text:
```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s
rule_files:
  - rules.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
        - localhost:9090
  - job_name: node
    static_configs:
      - targets:
        - localhost:9100
```
rules.yml contain the following text:
```yaml
groups:
  - name: example
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
```

```bash
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
$ tar -xzf alertmanager-*.linux-amd64.tar.gz
$ cd alertmanager-*.linux-amd64/
```
alertmanager.yml sending all alerts to email

```yaml
global:
  smtp_smarthost: 'localhost:25'
  smtp_from: 'youraddress@example.org'
route:
  receiver: example-email
receivers:
  - name: example-email
    email_configs:
      - to: 'youraddress@example.org'
```

```bash
$ ./alertmanager
```

# Types of metrics
- Counter: Counters go up, and reset when the process restarts
- Gauge: Gauges can go up and down
- Summary: Summaries track the size and number of events
- Histogram: Histograms track the size and number of events in buckets
```
# HELP example_gauge An example gauge
# TYPE example_gauge gauge
example_gauge -0.7
# HELP my_counter_total An example counter
# TYPE my_counter_total counter
my_counter_total 14
# HELP my_summary An example summary
# TYPE my_summary summary
my_summary_sum 0.6
my_summary_count 19
# HELP my_histogram An example histogram
# TYPE my_histogram histogram
latency_seconds_bucket{le="0.1"} 7
latency_seconds_bucket{le="0.2"} 18
latency_seconds_bucket{le="0.4"} 24
latency_seconds_bucket{le="0.8"} 28
latency_seconds_bucket{le="+Inf"} 29
latency_seconds_sum 0.6
latency_seconds_count 29
```


## Labels
All metrics can have labels, allowing the grouping of related time series.

```
# HELP my_summary An example summary
# TYPE my_summary summary
my_summary_sum{foo="bar",baz="quu"} 1.8
my_summary_count{foo="bar",baz="quu"} 453
my_summary_sum{foo="blaa",baz=""} 0
my_summary_count{foo="blaa",baz="quu"} 0
```

### Escaping

UTF-8

HELP and label value

Use backslash to scape

For HELP this is line feeds and backslashes

For label values, this is line feeds, backslashes, and double quotes
```
# HELP escaping A newline \\n and backslash \\ escaped
# TYPE escaping gauge
escaping{foo="newline \n backslash \\ double quote \" "} 1
```
### Timestamps
Timestamps for scrapes are usually applied automatically by Prometheus
```
# HELP foo I'm trapped in a client library
# TYPE foo gauge
foo 1 15100992000000
```

## Check metrics
```bash
$ curl http://localhost:8000/metrics | promtool check-metrics
```

# Labels
## What Are Labels?
Labels are key-value pairs associated with time series
```
http_requests_login_total
http_requests_logout_total
http_requests_adduser_total
http_requests_comment_total
http_requests_view_total
```
```
http_requests_total{path="/login"}
http_requests_total{path="/logout"}
http_requests_total{path="/adduser"}
http_requests_total{path="/comment"}
http_requests_total{path="/view"}
```
- instrumentation labels: are about things that are known inside your app
- target labels: identify a specific monitoring target. That is added to the labels of every time series returned from a scrape

### Reserved Labels and \__name__
Label names beginning with a double underscore __ are reserved

metric name is just another label called \__name__
```
up
{__name__="up"}
```
## Metric
```
# HELP latency_seconds Latency in seconds.
# TYPE latency_seconds summary
latency_seconds_sum{path="/foo"} 1.0
latency_seconds_count{path="/foo"} 2.0
latency_seconds_sum{path="/bar"} 3.0
latency_seconds_count{path="/bar"} 4.0

```
- timeseries 
- metric child
- metric family

## Label Patterns

Prometheus only supports 64-bit floating-point numbers as time series values, not any other data types such as strings

Label values are strings and there are certain limited use cases where it is okay to (ab)use them
### Enum
```
# HELP gauge The current state of resources.
# TYPE gauge resource_state
resource_state{resource_state="STARTING",resource="blaa"} 0
resource_state{resource_state="RUNNING",resource="blaa"} 1
resource_state{resource_state="STOPPING",resource="blaa"} 0
resource_state{resource_state="TERMINATED",resource="blaa"} 0
```
### Info
use a gauge with the value 1 and all the strings you’d like to have to annotate the target
```
# HELP python_info Python platform information
# TYPE python_info gauge
python_info{implementation="CPython",major="3",minor="5",patchlevel="2",version="3.5.2"} 1.0
```
add info label to other metrics
```
up
* on (instance, job) group_left(version)
python_info
```

### Cardinality
Cardinality, which in Prometheus is the number of time series you have

It is usually obvious that email addresses, customers, and IP addresses are poor choices for label values

The rule of thumb should be kept below 10 

It is also, okay to a cardinality around 100

also consider using __sample_limit__  as an emergency safety valve

# Node Exporter
Windows users should use the wmi_exporter

only to monitor the machine itself, not individual processes or services

not recommended to run the Node exporter in Docker

configure which categories of metrics it fetches
```
--collector.wifi
--no-collector.wifi
```

Different kernels expose different metrics

## CPU Collector
```
# HELP node_cpu_seconds_total Seconds the cpus spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 48649.88
node_cpu_seconds_total{cpu="0",mode="iowait"} 169.99
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 57.5
node_cpu_seconds_total{cpu="0",mode="softirq"} 8.05
node_cpu_seconds_total{cpu="0",mode="steal"} 0
node_cpu_seconds_total{cpu="0",mode="system"} 1058.32
node_cpu_seconds_total{cpu="0",mode="user"} 4234.94
```

```
avg without(cpu, mode)(rate(node_cpu_seconds_total{mode="idle"}[1m]))
```

## Filesystem Collector
collects metrics about your mounted filesystems, just as you would obtain from the df command
```
--collector.filesystem.ignored-mount-points
--collector.filesystem.ignored-fs-types
```
```
# HELP node_filesystem_size_bytes Filesystem size in bytes.
# TYPE node_filesystem_size_bytes gauge
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",mountpoint="/"} 9e+10
```

```
node_filesystem_avail_bytes/node_filesystem_size_bytes
```

## Diskstats Collector
exposes disk I/O metrics from /proc/diskstats

```
--collector.diskstats.ignored-devices
```

```
# HELP node_disk_io_now The number of I/Os currently in progress.
# TYPE node_disk_io_now gauge
node_disk_io_now{device="sda"} 0
```

## Netdev Collector
exposes metrics about your network devices with the prefix node_network_ and a device label
```
# HELP node_network_receive_bytes_total Network device statistic receive_bytes.
# TYPE node_network_receive_bytes_total counter
node_network_receive_bytes_total{device="lo"} 8.3213967e+07
node_network_receive_bytes_total{device="wlan0"} 7.0854462e+07
```
```promql
rate(node_network_receive_bytes_total[1m])
```

## Meminfo Collector
standard memory-related metrics with a node_memory_ prefix. These all come from your /proc/meminfo

```
# HELP node_memory_MemTotal_bytes Memory information field MemTotal.
# TYPE node_memory_MemTotal_bytes gauge
node_memory_MemTotal_bytes 8.269582336e+09
```

semantics get a bit muddy

## Hwmon Collector
collector provides metrics such as temperature and fan speeds with a node_hwmon_ prefix. This is the same information you can obtain with the sensors command.
```
# HELP node_hwmon_temp_celsius Hardware monitor for temperature (input)
# TYPE node_hwmon_temp_celsius gauge
node_hwmon_temp_celsius{chip="platform_coretemp_0",sensor="temp1"} 42
node_hwmon_temp_celsius{chip="platform_coretemp_0",sensor="temp2"} 42
node_hwmon_temp_celsius{chip="platform_coretemp_0",sensor="temp3"} 41
```

# Service Discovery
support for 
- Azure
- DNS 
- EC2
- OpenStack
- File
- Kubernetes

## Static
```yaml
scrape_configs:
  - job_name: prometheus
     static_configs:
        - targets:
          - localhost:9090
```

```yaml
scrape_configs:
  - job_name: node
    static_configs:
      - targets:
        - host1:9100
      - targets:
        - host2:9100
```

## File
It reads monitoring targets from files you provide on the local filesystem

either JSON or YAML format


filesd.json
```json
[
{
"targets": [ "host1:9100", "host2:9100" ],
"labels": {
"team": "infra",
"job": "node"
}
},
{
"targets": [ "host1:9090" ],
"labels": {
"team": "monitoring",
"job": "prometheus"
}
}
]
```
```yaml
scrape_configs:
  - job_name: file
    file_sd_configs:
      - files:
        - '*.json'
```
```yaml
scrape_configs:
  - job_name: ec2
    ec2_sd_configs:
      - region: <region>
        access_key: <access key>
        secret_key: <secret key>
```

## Relabelling
```yaml
scrape_configs:
  - job_name: file
    file_sd_configs:
      - files:
      - '*.json'
    relabel_configs:
      - source_labels: [team]
        regex: infra
        action: keep
      - source_labels: [team]
        regex: monitoring
        action: keep
```
all of them will be processed in order unless either a keep or drop action drops the target


Using multiple source labels
```yaml
scrape_configs:
  - job_name: file
    file_sd_configs:
      - files:
        - '*.json'
    relabel_configs:
      - source_labels: [job, team]
        regex: prometheus;monitoring
        action: drop
```

## Regex

```
a       The character a
.       Any single character
\.      A single period
.*      Any number of characters
.+      At least one character
a+      One or more a characters
[0-9]   Any single digit, 0-9
\d      Any single digit, 0-9
\d*     Any number of digits
[^0-9]  A single character that is not a digit
ab      The character a followed by the character b
a(b|c*) An a, followed by a single b, or any number of c characters

```

```regex
(.)(\d+)
```
a123

## Replace
```yaml
scrape_configs:
  - job_name: file
    file_sd_configs:
      - files:
        - '*.json'
    relabel_configs:
      - source_labels: [team]
        regex: monitoring
        replacement: monitor
        target_label: team
        action: replace
```

```yaml
scrape_configs:
  - job_name: file
    file_sd_configs:
      - files:
        - '*.json'
    relabel_configs:
      - source_labels: [team]
        regex: '(.*)ing'
        replacement: '${1}'
        target_label: team
        action: replace
```

## Labelmap
is different from the drop, keep, and replace actions

it applies to label names rather than label values
```yaml
scrape_configs:
  - job_name: ec2
    ec2_sd_configs:
    - region: <region>
      access_key: <access key>
      secret_key: <secret key>
    relabel_configs:
      - source_labels: [__meta_ec2_tag_service]
        target_label: job
      - regex: __meta_ec2_public_tag_monitor_(.*)
        replacement: '${1}'
        action: labelmap
```

## Scrape config
```yaml
scrape_configs:
  - job_name: example
    consul_sd_configs:
    - server: 'localhost:8500'
      scrape_timeout: 5s
      metrics_path: /admin/metrics
      params:
        foo: [bar]
      scheme: https
      tls_config:
        insecure_skip_verify: true
      basic_auth:
        username: brian
        password: hunter2
```
```http
admin/metrics?foo=bar
```
## metric_relabel_configs
relabelling applied to the time series scraped from a target

when dropping expensive metrics and when fixing bad metrics

```yaml
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
        - localhost:9090
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: http_request_size_bytes
        action: drop
```

## labeldrop and labelkeep
```yaml
scrape_configs:
  - job_name: misbehaving
    static_configs:
      - targets:
        - localhost:1234
    metric_relabel_configs:
      - regex: 'node_.*'
        action: labeldrop
```

# Containers and Kubernetes

## cAdvisor
In the same way, the Node exporter provides metrics about the machine, cAdvisor is
an exporter that provides metrics about cgroups

Cgroups is a Linux kernel isolation feature that are usually used to implement containers on Linux and are also used by runtime environments

```bash
docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:rw \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
--name=cadvisor \
google/cadvisor:v0.28.3
```

Open url in browser
```
http://localhost:8080/metrics
```

in prometheus.yaml
```yaml
scrape_configs:
  - job_name: cadvisor
    static_configs:
      - targets:
        - localhost:8080
```

### CPU

```
container_cpu_usage_seconds_total
container_cpu_system_seconds_total
container_cpu_user_seconds_total
```

### Memory


Similar to in the Node exporter, the memory usage metrics are less than perfectly clear and require digging through code

```
container_memory_cache
container_memory_rss
```

## Kubernetes

The Kubelet’s own /metrics only contain metrics about the Kubelet itself,
not container-level information. The Kubelet has an embedded cAdvisor on
its /metrics/cadvisor endpoint

Apiserver




## kube-state-metrics
An exporter exposes metrics about your services, pods, deployments, and other resources

```
kube_deployment_spec_replicas
kube_node_status_condition
kube_pod_container_status_restarts_total
```


# Blackbox exporter

Take in the target as a URL parameter
allows you to perform ICMP, TCP, HTTP, and DNS probing

```bash
$ wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.23.0/blackbox_exporter-0.23.0.linux-amd64.tar.gz
$ tar -xzf blackbox_exporter-0.23.0.linux-amd64.tar.gz
cd blackbox_exporter-0.23.0.linux-amd64/
sudo ./blackbox_exporter
```

open url in browser
```
http://localhost:9115/
```

## ICMP

```
http://localhost:9115/probe?module=icmp&target=localhost
```

```
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup
in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.000164439
# HELP probe_duration_seconds Returns how long the probe took to complete
in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.000670403
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

look inside blackbox.yml 
```yaml
icmp:
  prober: icmp
```
other options 
```
payload_size
dont_fragment
```

```
http://localhost:9115/probe?module=icmp&target=www.google.com
```

```
http://localhost:9115/probe?module=icmp&target=www.google.com&debug=true
```

at blackbox.yml 

```yaml
icmp_ipv4:
  prober: icmp
  icmp:
      preferred_ip_protocol: ip4
```

## TCP

```
http://localhost:9115/probe?module=tcp_connect&target=localhost:22
```

at blackbox.yml 

```yaml
tcp_connect:
  prober: tcp
```

```yaml
ssh_banner:
  prober: tcp
  tcp:
    query_response:
      - expect: "^SSH-2.0-"
```


probe_failed_due_to_regex

```yaml
tcp_connect_tls:
  prober: tcp
  tcp:
      tls: true
```

```
http://localhost:9115/probe?module=tcp_connect_tls&target=www.robustperception.io:443
```
```
# HELP probe_ssl_earliest_cert_expiry Returns earliest SSL cert expiry date
# TYPE probe_ssl_earliest_cert_expiry gauge
probe_ssl_earliest_cert_expiry 1.522039491e+09
```

## HTTP

```
http://localhost:9115/probe?module=http_2xx&target=https://www.robustperception.io
```

```
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup
in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.00169128
# HELP probe_duration_seconds Returns how long the probe took to complete in
seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.191706498
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HELP probe_http_content_length Length of http content response
# TYPE probe_http_content_length gauge
probe_http_content_length -1
# HELP probe_http_duration_seconds Duration of http request by phase, summed over
all redirects
# TYPE probe_http_duration_seconds gauge
probe_http_duration_seconds{phase="connect"} 0.018464759
probe_http_duration_seconds{phase="processing"} 0.132312499
probe_http_duration_seconds{phase="resolve"} 0.00169128
probe_http_duration_seconds{phase="tls"} 0.057145526
probe_http_duration_seconds{phase="transfer"} 6.0805e-05
# HELP probe_http_redirects The number of redirects
# TYPE probe_http_redirects gauge
probe_http_redirects 0
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 1
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 200
# HELP probe_http_version Returns the version of HTTP of the probe response
# TYPE probe_http_version gauge
probe_http_version 1.1
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_ssl_earliest_cert_expiry Returns earliest SSL cert expiry in
unixtime
# TYPE probe_ssl_earliest_cert_expiry gauge
probe_ssl_earliest_cert_expiry 1.522039491e+09
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

check body content

```yaml
http_200_ssl_prometheus:
  prober: http
  http:
    valid_status_codes: [200]
    fail_if_not_ssl: true
    fail_if_not_matches_regexp:
      - Prometheus
```

```
http://localhost:9115/probe?module=http_200_ssl_prometheus&target=http://prometheus.io
```

## DNS
Primarily for testing DNS servers

at blackbox.yml
```yaml
dns_tcp:
  prober: dns
  dns:
    transport_protocol: "tcp"
    query_name: "www.prometheus.io"
```

```
http://localhost:9115/probe?module=dns_tcp&target=8.8.8.8
```

```bash
$ dig -tcp @8.8.8.8 www.prometheus.io
```

## Prometheus Configuration

You can take advantage of service discovery, as the
\__param_\<name\> label can be used to provide URL parameters 

```yaml
scrape_configs:
  - job_name: blackbox
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://www.prometheus.io
        - http://www.robustperception.io
        - http://demo.robustperception.io
    relabel_configs:
      - source_labels: [__address__]
      target_label: __param_target
      - source_labels: [__param_target]
      target_label: instance
      - target_label: __address__
      replacement: 127.0.0.1:9115
```

## Blackbox Timeouts
Blackbox prober determines the timeout automatically based on the
scrape_timeout in Prometheus

HTTP header called X-Prometheus-Scrape-Timeout-Seconds

# PromQL
Is the Prometheus Query Language

Labels are a key part of PromQL
## Aggregation Basics
- Gauge: usually when aggregating them you want to take a sum, average, minimum, or maximum
  
   node_filesystem_free_bytes
- Counter: the number or size of events

    that total is of little use to you on its own 

    is usually done using the rate, irate, increase function

    node_network_receive_bytes_total

    The output of the rate is a gauge, so the same aggregations apply as for gauges

- Summary:  usually contain both a _sum and _count
    
    _sum and _count are both counters

- Histogram: track the distribution of the size of events
    
    prometheus_tsdb_compaction_duration_seconds

    ```promql
    histogram_quantile(
    0.90,
    rate(prometheus_tsdb_compaction_duration_seconds_bucket[1d]))
    ```

## Selectors

```promql
process_resident_memory_bytes{job="node"}
```

## Matchers
- = equality matcher

    Empty label, the value is the same as not having that label, you could use foo=" " to specify that the foo label is not be present
- != negative equality matcher
- =~ regular expression matcher
  
  job=~"n.*"
- !~ negative regular expression matcher

```promql  
process_resident_memory_bytes{job="node"}
{__name__="process_resident_memory_bytes",job="node"}
```

The selector {} returns an error

At least one of the matchers in a selector must not match the empty string

## Instant Vector
The most recent samples before the query evaluation time
### timestamp
### staleness

## Range Vector
Can return many samples for each time series

Always used with the rate function

avg_over_time

### Durations
```
Suffix  Meaning
ms      Milliseconds
s       Seconds, which have 1,000 milliseconds
m       Minutes, which have 60 seconds
h       Hours, which have 60 minutes
d       Days, which have 24 hours
w       Weeks, which have 7 days
y       Years, which have 365 days
```
90m is valid but 1h30m and 1.5h are not

## Offset
Allows you to take the evaluation time and put it further back in time

```promql
process_resident_memory_bytes{job="node"} offset 1h
```

```promql
process_resident_memory_bytes{job="node"}
-
process_resident_memory_bytes{job="node"} offset 1h
```

## HTTP API
### query
query endpoint

/api/v1/query
```
http://localhost:9090/api/v1/query?query=process_resident_memory_bytes
```

```
http://localhost:9090/api/v1/query?query=prometheus_tsdb_head_samples_appended_total[1m]
```

```
http://localhost:9090/api/v1/query?query=42
```

### query_range
Is not a range query

Stepwise multiple evaluation single query
```
http://localhost:9090/api/v1/query_range?query=rate(prometheus_tsdb_head_samples_appended_total[5m])&start=1514764800&end=1514765700&step=60
```

# Aggregation Operators
11 aggregation operators

with 2 optional clauses

## Grouping
```
node_filesystem_size_bytes{device="/dev/sda1",fstype="vfat",instance="localhost:9100",job="node",mountpoint="/boot/efi"} 100663296
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",instance="localhost:9100",job="node",mountpoint="/"} 90131324928
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run"} 826961920
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/lock"} 5242880
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/user/1000"} 826961920
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/user/119"} 826961920
```

There are three instrumentation labels: 'device', 'fstype', and 'mountpoint'
There are also two target labels: job and instance.

### without
Aggregating to specify the specific labels you want to remove

```promql
sum without(fstype, mountpoint)(node_filesystem_size_bytes)
```

```
# Group {device="/dev/sda1",instance="localhost:9100",job="node"}
node_filesystem_size_bytes{device="/dev/sda1",fstype="vfat",instance="localhost:9100",job="node",mountpoint="/boot/efi"} 100663296

# Group {device="/dev/sda5",instance="localhost:9100",job="node"}
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",instance="localhost:9100",job="node",mountpoint="/"} 90131324928

# Group {device="tmpfs",instance="localhost:9100",job="node"}
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run"} 826961920
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/lock"} 5242880
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/user/1000"} 826961920
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",instance="localhost:9100",job="node",mountpoint="/run/user/119"} 826961920
```
then apply aggregator
```
{device="/dev/sda1",instance="localhost:9100",job="node"} 100663296
{device="/dev/sda5",instance="localhost:9100",job="node"} 90131324928
{device="tmpfs",instance="localhost:9100",job="node"} 2486128640
```

?
```promql
sum without()(node_filesystem_size_bytes)
```

### by
Where __without__ specifying the labels to remove, __by__ specifying the labels to keep

```promql
sum by(job, instance, device)(node_filesystem_size_bytes)
```
```
{device="/dev/sda1",instance="localhost:9100",job="node"} 100663296
{device="/dev/sda5",instance="localhost:9100",job="node"} 90131324928
{device="tmpfs",instance="localhost:9100",job="node"} 2486128640
```
How many time series have the same metric name?
```promql
sort_desc(count by(__name__)({__name__=~".+"}))
```
How many machines were running each kernel version?
```promql
count by(release)(node_uname_info)
```
?
```promql
sum by()(node_filesystem_size_bytes)
```

## Operators

### sum
When dealing with counters, it is important that you take a rate before aggregation with a sum
```promql
sum without(device)(rate(node_disk_read_bytes_total[5m]))
```
### count
Counts the number of time series in a group

Here it is okay not to use a rate with a counter

```promql
count without(device)(node_disk_read_bytes_total)
```

#### Unique label values
Number of CPUs in each of your machines
```promql
count without(cpu)(count without (mode)(node_cpu_seconds_total))
```
inner count 
```
{cpu="0",instance="localhost:9100",job="node"} 8
{cpu="1",instance="localhost:9100",job="node"} 8
{cpu="2",instance="localhost:9100",job="node"} 8
{cpu="3",instance="localhost:9100",job="node"} 8
```
outer count
```
{instance="localhost:9100",job="node"} 4
```

cardinality
```promql
count(count by(cpu)(node_cpu_seconds_total))
```

### avg
```promql
avg without(cpu)(rate(node_cpu_seconds_total[5m]))
```

### stddev and stdvar
Standard deviation and standard variance

### min and max
```promql
max without(device, fstype, mountpoint)(node_filesystem_size_bytes)
```

### topk and bottomk
- The labels of the time series they return for a group are not the labels of the group
- They can return more than one time series per group
- They take an additional parameter
```promql
topk without(device, fstype, mountpoint)(2, node_filesystem_size_bytes)
```
Returns input time series with all their labels
### quantile
The φ-quantile is the observation value that ranks at the number φ*N among the N observations. Examples for φ-quantiles: The 0.5-quantile is known as the median. 

Take the 0.25 quantile (also known as the 25th percentile, or 1st quartile) -- it defines the value (let's call it x) for a random variable, such that the probability that a random observation of the variable is less than x is 0.25 (25% chance).

```promql
quantile without(cpu)(0.9, rate(node_cpu_seconds_total{mode="system"}[5m]))
```
```
{instance="localhost:9100",job="node",mode="system"} 0.024558620689654007
```
This means that 90% of my CPUs are spending at least 0.02 seconds per second in the system mode 
### count_values
Return more than one time series from a group

Build a frequency histogram of the values of the time series in the group

```
software_version{instance="a",job="j"} 7
software_version{instance="b",job="j"} 4
software_version{instance="c",job="j"} 8
software_version{instance="d",job="j"} 4
software_version{instance="e",job="j"} 7
software_version{instance="f",job="j"} 4
```
```promql
count_values without(instance)("version", software_version)
```
```
{job="j",version="7"} 2
{job="j",version="8"} 1
{job="j",version="4"} 3
```

There is no bucketing involved when the frequency histogram is created; the exact values of the time series are used

# Binary Operators
That take two operands
## Working with Scalars
Scalars are single numbers with no dimensionality
```
0
```
```
{} 0
```
## Arithmetic Operators
```promql
process_resident_memory_bytes / 1024
```
six arithmetic operations
- \+ addition
- \- subtraction
- \* multiplication
- / division
- % modulo
- ^ exponentiation

```promql
(1024 * 1024 * 1024)
```
```promql
1e9 - process_resident_memory_bytes
```
## Comparison Operators
- == equals
- != not equals
- \> greater than
- < less than
- \>= greater than or equal to
- <= less than or equal to
```
process_open_fds{instance="localhost:9090",job="prometheus"} 14
process_open_fds{instance="localhost:9100",job="node"} 7
```
```promql
process_open_fds > 10
```

Be careful about == and != and float numbers

are primarily useful in alert rules

## bool modifier
After the operator instead of time series filtering return boolean value 0 or 1

```promql
process_open_fds > bool 10
```
```
{instance="localhost:9090",job="prometheus"} 1
{instance="localhost:9100",job="node"} 0
```
## Vector Matching
Two instant vectors
vector matching
### One-to-One
```
process_open_fds{instance="localhost:9090",job="prometheus"} 14
process_open_fds{instance="localhost:9100",job="node"} 7
process_max_fds{instance="localhost:9090",job="prometheus"} 1024
process_max_fds{instance="localhost:9100",job="node"} 1024
```

###  for the arithmetic operators

```promql
process_open_fds
/
process_max_fds
```
```
{instance="localhost:9090",job="prometheus"} 0.013671875
{instance="localhost:9100",job="node"} 0.0068359375
```
Samples with exactly the same labels, except for the metric name in the label __name__, were matched together

If a sample on one side had no match on the other side, then it would
not be present in the result

If a binary operator returns an empty instant vector, it is probably because the labels of the samples in the operands don’t match

__ignoring__ clause to ignore certain labels when matching, similar to how __without__

```promql
sum without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))
/ ignoring(mode)
sum without(mode, cpu)(rate(node_cpu_seconds_total[5m]))
```
__on__ clause allows you to consider only the labels you provide, similar to how __by__ works for aggregations
```promql
sum by(instance, job)(rate(node_cpu_seconds_total{mode="idle"}[5m]))
/ on(instance, job)
sum by(instance, job)(rate(node_cpu_seconds_total[5m]))
```
### For the comparison operators
For the comparison operators when there are two instant vectors the value from the left-hand side is returned
```promql
process_open_fds
>
(process_max_fds * .5)
```
```promql
(process_max_fds * .5)
<
process_open_fds
```
## Many-to-One and group_left
```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

```promql
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

Takes all of its labels from samples of your operand on the left hand side

Another use for group_left adding labels from info metrics to other metrics
```promql
up
* on(instance) group_left(version)
prometheus_build_info
```

## Many-to-Many and Logical Operators
Set operators
- or union
- and intersection
- unless set subtract
### or
For each group where the group on the left-hand side has samples, then they are returned; otherwise, the samples in the group on the right-hand side are returned
```promql
node_hwmon_sensor_label
or ignoring(label)
(node_hwmon_temp_celsius * 0 + 1)
```

```promql
node_custom_metric
or
up * 0
```
```promql
node_custom_metric
or
(up{job="node"} == 1) * 0
```
```promql
(a >= b) or b
```

## unless operator
Returns the left-hand group, unless the right-hand group
has members, in which case it returns no samples
```promql
rate(process_cpu_seconds_total[5m])
unless
process_resident_memory_bytes < 100 * 1024 * 1024
```
```promql
up{job="node"} == 1
unless
node_custom_metric
```

```promql
up == 1
unless on (job, instance)
node_custom_metric
```
## and operator
It returns a group from the left-hand operand only if the matching right-hand group

You can think of it as an __if__ operator
```promql
(
rate(http_request_duration_microseconds_sum{job="prometheus"}[5m])
/
rate(http_request_duration_microseconds_count{job="prometheus"}[5m])
) > 1000000
and
rate(http_request_duration_microseconds_count{job="prometheus"}[5m]) > 1
```
## Operator Precedence
1. ^
2. \* / %
3. \+ -
4. == != > < >= <=
5. unless and
6. or

Recommend adding parentheses
# Functions
Almost all PromQL functions return an instant vector

time and scalar return scalar

multiple functions, including rate and avg_over_time, take a range vector
## Changing Type
You will have a vector but need a scalar or vice versa
### Vector
Takes a scalar value, and converts it into an instant vector with one labelless sample and the given value.
```promql
vector(1)
```
```promql
sum(some_gauge) or vector(0)
```
### Scalar
Takes an instant vector with a single sample and converts it to a
scalar with the value of the input sample

When you should use math functions that only work on instant vectors.
```promql
scalar(ln(vector(2)))
```

```promql
year(process_start_time_seconds)
==
scalar(year())
```

## Math
Mathematical operations on instant vectors
### abs
### ln, log2, and log10
```promql
ln(x) / ln(3)
```
### exp
```promql
exp(vector(1))
```
### sqrt
```promql
sqrt(vector(9))
```
### ceil and floor
```promql
ceil(vector(0.1))
```
### round
```promql
round(vector(5.5))
```
```promql
round(vector(2446), 1000)
```
### clamp_max and clamp_min
```promql
clamp_min(process_open_fds, 10)
```
## Time and Date
Prometheus works entirely in UTC
### Time
It returns the evaluation time of the query
```promql
time()
```
```promql
time() - process_start_time_seconds
```
pushgateway scenario
```promql
time() - my_job_last_success_seconds > 3600
```
### minute, hour, day_of_week, day_of_month, days_in_month,month, and year
```
Expression      Result
minute()        {} 39
hour()          {} 13
day_of_week()   {} 3
day_of_month()  {} 14
days_in_month() {} 28
month()         {} 2
year()          {} 2018
```
day_of_week starts with 0 for Sunday

```promql
year(process_start_time_seconds)
```
```promql
sum(
(year(process_start_time_seconds) == bool scalar(year()))
*
(month(process_start_time_seconds) == bool scalar(month()))
)
```
### Timestamp
It looks at the timestamp of the samples in an instant vector rather than the value.
```promql
timestamp(up)
```
## Labels
### label_replace
The arguments to label_replace are the instant vector input, the name of the output label, the replacement, the name of the source label, and the regular expression.
```promql
label_replace(node_disk_read_bytes_total, "dev", "${1}", "device", "(.*)")
```
Will change in the __returned__ timeseries 
### label_join
```promql
label_join(node_disk_read_bytes_total, "combined", "-", "instance", "job")
```
```
node_disk_read_bytes_total{combined="localhost:9100-node",device="sda",instance="localhost:9100",job="node"} 4766359040
```

## Missing Series and Absent
It uses the labels from any equality matcher
```
Expression                                      Result
absent(up)                                      empty instant vector
absent(up{job="prometheus"})                    empty instant vector
absent(up{job="missing"})                       {job="missing"} 1
absent(up{job=~"missing"})                      {} 1
absent(non_existent)                            {} 1
absent(non_existent{job="foo”,env="dev"})       {job="foo”,env="dev"} 1
absent(non_existent{job="foo”,env="dev"} * 0)   {} 1
```

## Sorting with sort and sort_desc
```promql
sort(node_filesystem_size_bytes)
```
NaNs are always sorted to the end

## Histograms with histogram_quantile
```promql
histogram_quantile(
0.90,
rate(prometheus_tsdb_compaction_duration_seconds_bucket[1d]))
```
You must always use rate first for buckets.

histogram_quantile needs gauges to work on.

## Counters
Not just the counter metric, but also the _sum, _count, and _bucket
time series from summary and histogram 

### Rate
Primary function you will use with counters
```promql
rate(process_cpu_seconds_total[1m])
```
Automatically handles counter resets, and any decrease in a counter is considered to be a counter reset

[5,10,4,6] 

[5,10,14,16]

vector that is at least four times your scrape interval

for a 1-minute scrape interval you might use a 4-minute rate, but usually that is rounded up to a 5-min

you should aim to have the same range used on all your rate functions

### increase
```promql
increase(x_total[5m])
```
```promql
rate(x_total[5m]) * 300
```

### irate
It only looks at the last two samples of the range vector it is passed

more volatile

It is not advisable to use irate in alerts due to it being sensitive to
brief spikes and dips; use rate instead

### resets
Returns how many times each time series in a range vector has reset
```promql
resets(process_cpu_seconds_total[1h])
```

## Changing Gauges
### changes
Counts how many times a gauge has changed
```promql
changes(process_start_time_seconds[1h])
```

### deriv
```
x - x offset 1h
```
```promql
deriv(process_resident_memory_bytes[1h])
```
least-squares regression to estimate the slope of each of the time series in a range vector

### predict_linear
```promql
predict_linear(node_filesystem_free_bytes{job="node"}[1h], 4 * 3600)
```
```promql
deriv(node_filesystem_free_bytes{job="node"}[1h]) * 4 * 3600
+
node_filesystem_free_bytes{job="node"}[1h]
```
is useful for resource limit alerts

### delta
similar to __increase__, but without the counter reset handling

### idelta

### holt_winters
Implements Holt-Winters double exponential smoothing
```promql
holt_winters(process_resident_memory_bytes[1h], 0.1, 0.5)
```
with a smoothing factor of 0.1 and a trend factor of 0.5

Both factors must be between 0 and 1

## Aggregation Over Time
Similar to __avg__ apply the same logic, but across the values of a time series in a range vector

- sum_over_time
- count_over_time
- avg_over_time
- stddev_over_time
- stdvar_over_time
- min_over_time
- max_over_time
- quantile_over_time

```promql
max_over_time(process_resident_memory_bytes[1h])
```
# Recording Rules
Evaluate PromQL expressions regularly and ingest their results

To speed up your dashboards

Provide aggregated results for use elsewhere
## Using Recording Rules
rule_files at prometheus.yml
```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s
rule_files:
  - rules.yml
```
reload config
```bash
kill -HUP <pid>
```
```bash
curl -X POST http://localhost:9090/-/reload
```
enable reload api
```
--web.enable-lifecycle
```

```yaml
groups:
  - name: example
    rules:
      - record: job:process_cpu_seconds:rate5m
        expr: sum without(instance)(rate(process_cpu_seconds_total[5m]))
      - record: job:process_open_fds:max
        expr: max without(instance)(process_open_fds)
```
__name__: must be unique within a rule file

__expr__: is the PromQL expression to be evaluated

__record__: output into the metric name specified by record

Each rule in a group is evaluated in turn

Different groups will be run at different times just as different targets are scraped at diffrent times

http://localhost:9090/rules

## When to Use Recording Rules
### Reducing Cardinality
```promql
sum without(instance)(rate(process_cpu_seconds_total{job="node"}[5m]))
```
```yaml
groups:
  - name: node
    rules:
      - record: job:process_cpu_seconds:rate5m
        expr: sum without(instance)(rate(process_cpu_seconds_total{job="node"}[5m]))
```
### Composing Range Vector Functions
```promql
max_over_time(sum without(instance)(rate(x_total[5m]))[1h])
```
```yaml
groups:
  - name: j_job_rules
    rules:
      - record: job:x:rate5m
        expr: sum without(instance)(rate(x_total{job="j"}[5m]))
      - record: job:x:max_over_time1h_rate5m
        expr: max_over_time(job:x:rate5m{job="j"}[1h])
```
## Naming of Recording Rules
Colons are valid characters to have in metric names but are to be avoided in instrumentation.
```
job_device:node_disk_read_bytes:rate5m
```
it has __job__ and __device__ labels

it is based off is __node_disk_read_bytes__

rate(node_disk_read_bytes_total[5m])

__level:metric:operations__
# Alerting
## Prometheus and Alertmanager architecture

## Alerting Rules
It is normal to have all the rules and alerts for a job in one group.
```yaml
groups:
  - name: node_rules
    rules:
      - record: job:up:avg
        expr: avg without(instance)(up{job="node"})
      - alert: ManyInstancesDown
        expr: job:up:avg{job="node"} < 0.5
```
```yaml
- alert: InstanceDown
  expr: up{job="node"} == 0
```

```promql
ALERTS{alertstate="firing"}
```
Include metric labels and does not include the metric name label \__name__

Include an alertname label with the name of the __alertname__

```yaml
- alert: ManyInstancesDown
  expr: >
        (
        avg without(instance)(up{job="node"}) < 0.5
        and on()
        hour() > 9 < 17
        )
```

```yaml
- alert: BatchJobNoRecentSuccess
  expr: >
        time() - my_batch_job_last_success_time_seconds{job="batch"} > 86400*2
```
## for
metrics involve many race conditions

- scrape may timeout
- rule evaluation could be delayed
- target could have falctuation
  
```yaml
groups:
  - name: node_rules
    rules:
      - record: job:up:avg
        expr: avg without(instance)(up{job="node"})
      - alert: ManyInstancesDown
        expr: avg without(instance)(up{job="node"}) < 0.5
        for: 5m
```
Given alert must be returned for at least this long before it starts firing

Until the __for__ condition is met is considered to be pending


recommend using a form of at least 5 minutes for all of your alert rules

http://localhost:9090/alerts


for requires that your alerting rule returns the same time series for a period of time

for the state can be reset if a single rule evaluation does not contain a given time series



```yaml
- alert: FDsNearLimit
  expr: >
        process_open_fds > process_max_fds * .8
  for: 5m
```
you can use the _over_time function

```yaml
- alert: FDsNearLimit
  expr:
    (
    max_over_time(process_open_fds[5m])
    >
    max_over_time(process_max_fds[5m]) * 0.9
    )
  for: 5m
```

```yaml
- alert: ProbeFailing
  expr: up{job="blackbox"} == 0 or probe_success{job="blackbox"} == 0
  for: 5m
```

## Alert Labels
you can specify labels for an alerting

```yaml
- alert: InstanceDown
  expr: up{job="node"} == 0
  for: 1h
  labels:
    severity: ticket
- alert: ManyInstancesDown
  expr: job:up:avg{job="node"} < 0.5
  for: 5m
  labels:
    severity: page
```

## External Labels
at prometheus.yml

```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s
  external_labels:
    region: eu-west-1
    env: prod
    team: frontend
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['localhost:9093']
```

Labels applied as defaults when your Prometheus talks to other systems, such as the Alertmanager, federation, remote read and remote write but not
the HTTP query API
