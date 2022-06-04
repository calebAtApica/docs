# Filebeat

Filebeat is a lightweight shipper for **forwarding and centralizing log data**. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.

LOGIQ supports data ingestion from Filebeat. The following section describes the steps for configuring log forwarding from Filebeat to LOGIQ by modifying the Filebeat configuration file.&#x20;

## Configuring Filebeat to forward logs to LOGIQ

To configure Filebeat, you need to configure the Filebeat configuration file. The location of your Filebeat configuration file can vary depending on your platform. Locate and access the configuration file and modify it as described below.

### Ingest Ports

LOGIQ.AI hosts the lumberjack protocol at port **25224 (non-TLS)**

The ports are configurable and can be changed if needed.

### Understanding Filebeats configuration

Filebeats configuration consists of 2 key sections _**input**_ and _**output.**_

The input section describes what files to process. The output section describes how/where to send the collected data. In addition, the input section contains _**processors**_ that allow data modification before it is sent to the output. We will look at each of these below.

### INPUT

The input section will typically contain the list of file paths that Filebeat should monitor.

```
inputs:
  - enabled: true
    paths:
    - /var/log/*.log
    type: filestream
```

### OUTPUT

The output section describes how to send data to LOGIQ. The LOGIQ layer supports the lumberjack protocols v1 and v2 for ingesting data and is compatible with _logstash_. Use output type _logstash_

```
output:
  logstash:
    hosts:["Logiq-endpoint>:25224"]
```

### PROCESSORS

Under the `input` section, including the `processors` the section in addition to your existing Filebeat configuration along with **`namespace`**, **`application`**, and **`cluster_id`**. Doing so ensures that the logs you forward to LOGIQ can be identified and partitioned easily

We strongly recommend partitioning your data for better organization as well as query performance. Each Cluster, Namespace, and Application tuple becomes a partition in LOGIQ and can be used to organize data that is logically related

```
  inputs:
  - enabled: true
    paths:
    - <path to your log>
    type: filestream
    processors:
      - add_fields:
          target: logiq
          fields:
            namespace: myproject
            application: test
            cluster_id: foo
```

### Example&#x20;

The following is a complete _Filebeat_ configuration for forwarding logs to LOGIQ without SSL.&#x20;

```
output.logstash:
  hosts: ["logs.mycompany.com:5044"]
filebeat:
  inputs:
  - enabled: true
    paths:
    - <path to your log>
    type: filestream
    processors:
      - add_fields:
          target: logiq
          fields:
            namespace: myproject
            application: test
            cluster_id: foo

output:
#  console:
#   pretty: true
  logstash:
    hosts:["Logiq-endpoint>:25224"]
processors:
- add_host_metadata:
    when:
      not:
        contains:
          tags: forwarded
- add_cloud_metadata: null
- add_docker_metadata: null
- add_kubernetes_metadata: null
setup:
  kibana: null
  template:
    settings:
      index:
        number_of_shards: 1
```

###
