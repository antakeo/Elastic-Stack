# Logstash Introduction
* open source data collection engine
* normalizes data
* enriches and transforms data through plugins <br>
<br>
<img src="https://www.elastic.co/guide/en/logstash/current/static/images/logstash.png">

## Installation
Skip if environment is given

Java (JVM) Version
* Java 8
* Java 11
* Java 15

Installing Logstash
* From Downloaded Binaary

https://www.elastic.co/downloads/logstash

* Installing from APT package repo
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install logstash
* Installing from YUM package repo <br>
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

* Edit /etc/yum.repos.d/ and create file logstash.repo with the contents
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-mdsudo yum install logstash
* From Docker

docker pull docker.elastic.co/logstash/logstash:7.16.3

## Logstash Pipeline

Logstash pipeline has two requirements, Input and Output.  The Filter is optional. <br>

<img src="https://www.elastic.co/guide/en/logstash/current/static/images/basic_logstash_pipeline.png">

# Basic Logstash pipeline config file

* Default location for configuration files is /etc/logstash/conf.d/
* Can be changed in pipelines.yml

*Example myFirstPipeline.conf*
input {
    stdin {}
}

# The filter part of this file is optional.

filter {
}

output {
    stdout {}
}
### Input
You use inputs to get data into Logstash. Some of the more commonly-used inputs are:
* **file**: reads from a file on the filesystem, much like the UNIX command
* **syslog**: listens on the well-known port 514 for syslog messages and parses according to the RFC3164 format
* **redis**: reads from a redis server, using both redis channels and redis lists. Redis is often used as a "broker" in a centralized Logstash installation, which queues Logstash events from remote Logstash "shippers"
* **beats**: processes events sent by Beats
* **http**: Receives events over HTTP or HTTPS

Full list of input plugins
https://www.elastic.co/guide/en/logstash/current/input-plugins.html


### Input Examples
Files
input {
    file {
        path => "/home/datadmin/mylogfiles.csv"
        }
    }
Syslog to open port on 514 tcp/udp
input {
    tcp {
        type => "syslog"
        port => 514
        }
    }

input {
  syslog {
    port => 12345
    codec => cef
    syslog_field => "syslog"
  }
}
### Codec

A codec is the name of Logstash codec used to represent the data. Codecs can be used in both inputs and outputs.

Input codecs provide a convenient way to decode your data before it enters the input. Output codecs provide a convenient way to encode your data before it leaves the output. Using an input or output codec eliminates the need for a separate filter in your Logstash pipeline.

https://www.elastic.co/guide/en/logstash/8.0/codec-plugins.html

Common codecs:
* Avro
* cef
* csv
* json

### Codecs
Codecs are basically stream filters that can operate as part of an input or output. Codecs enable you to easily separatethe transport of your messages from the serialization process. Popular codecs include json, msgpack, and plain (text).

* json: encode or decode data in the JSON format.
* multiline: merge multiple-line text events such as java exception and stacktrace messages into a single event.
input {
    stdin {
        codec => cef {
          ecs_compatibility => v1
        }
      }
    }
### Filter
Filters are intermediary processing devices in the Logstash pipeline. You can combine filters with conditionals to perform an action on an event if it meets certain criteria. Some useful filters include:

* **grok**: parse and structure arbitrary text. Grok is currently the best way in Logstash to parse unstructured log data into something structured and queryable. With 120 patterns built-in to Logstash, it’s more than likely you’ll find one that meets your needs!
* **mutate**: perform general transformations on event fields. You can rename, remove, replace, and modify fields in your events.
* **drop**: drop an event completely, for example, debug events.
* **clone**: make a copy of an event, possibly adding or removing fields.
* **geoip**: add information about geographical location of IP addresses (also displays amazing charts in Kibana!)

Full list of filter plugins https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

Grok filters


```python

```

Grok premade filters & grok debugger

https://github.com/logstash-plugins/logstash-patterns-core/tree/main/patterns/ecs-v1


```python

```

### Output
Outputs are the final phase of the Logstash pipeline. An event can pass through multiple outputs, but once all output processing is complete, the event has finished its execution. Some commonly used outputs include:

* elasticsearch: send event data to Elasticsearch. If you’re planning to save your data in an efficient, convenient, and easily queryable format Elasticsearch is the way to go. Period. Yes, we’re biased :)
* file: write event data to a file on disk.
* graphite: send event data to graphite, a popular open source tool for storing and graphing metrics. http://graphite.readthedocs.io/en/latest/
* statsd: send event data to statsd, a service that "listens for statistics, like counters and timers, sent over UDP and sends aggregates to one or more pluggable backend services". If you’re already using statsd, this could be useful for you!

Full list of output plugins https://www.elastic.co/guide/en/logstash/current/output-plugins.html


Output to commandline, used to debug or test
output {
    stdout { codec => rubydebug }
}
Output to file
output {
 file {
   path => ...
 }
}
Output to elasticsearch
output {
    elasticsearch {
        hosts => ["IP Address 1:port1", "IP Address 2:port2", "IP Address 3"]
        index => "my_index"
    }
}
# Lab 1
**Task** : Learn basics of Input/Output by starting logstash with stdin and stdout plugins

1. SSH into your server with elkadmin/elkadmin
2. Run the following command.
* -e flag allows logstash to accept configurations through command-line.
* It may take a minute for logstash to start, when it says "Pipelines running.." then its ready.
sudo /usr/share/logstash/bin/logstash -e "input { stdin { } } output { stdout { codec => rubydebug } }"
3. Type in "I heart CTC" in the terminal and review the output
{
       "message" => "I heart CTC",
      "@version" => "1",
    "@timestamp" => 2022-02-01T16:10:35.685Z,
          "host" => "elkstack1"
}
4. Now lets enter a http log into the terminal. Copy/paste the following and review the output.
83.149.9.216 - - [04/Jan/2015:05:13:44 +0000] "GET /presentations/logstash-monitorama-2013/plugin/highlight/highlight.js HTTP/1.1" 200 26185 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"{
       "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/lgstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplee.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 0_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
      "@version" => "1",
    "@timestamp" => 2022-02-01T16:10:53.958Z,
          "host" => "elkstack1"
}
5. As you can see we were able to successfully send data into the pipeline.  Altough very simple, Next we will apply some filtering
6. Exit/close (Ctrl-C) out of your logstash pipeline.

# Lab 2

Task:  Apply Grok filtering to your mypipeline.conf
1. We will use the http log from lab 1 to apply a grok filter to structure the data.
2. Since http logs are common, logstash includes a grok filters to parse http logs
3. Create a pipeline file in /etc/logstash/conf.d/ and call it mypipeline.conf
4. Edit the file with the output below.
input {
    stdin { }
}
    
filter {
    grok {
        match => { "message" => "%{HTTPD_COMBINEDLOG}"}
    }
}

output {
    stdout { 
        codec => rubydebug
        }
}
5. Run Logstash against the .conf file with the following.
* -f flag specifies to run logstash with a specific conf file.
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/mypipeline.conf
6. Once the pipeline is running, copy and paste the http log into the terminal and observe the output.
83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"{
           "verb" => "GET",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
      "timestamp" => "04/Jan/2015:05:13:42 +0000",
        "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
     "@timestamp" => 2022-02-01T16:49:05.441Z,
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
    "httpversion" => "1.1",
          "bytes" => "203023",
           "auth" => "-",
       "@version" => "1",
       "clientip" => "83.149.9.216",
           "host" => "elkstack1",
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
       "response" => "200",
          "ident" => "-"
}
7. Our grok filter successfully parsed our http log into its perspective fields.
8. Lets use some more filters to remove the "message" field and change the "verb" field to method.
9. Close your current pipeline and Edit the your mypipeline.conf to add the mutate options to rename and remove a field.
input {
    stdin { }
}
    
filter {
    grok {
        match => { "message" => "%{HTTPD_COMBINEDLOG}"} 
    }
    mutate {
        rename => { "verb" => "method"}
        remove_field => ["message"]
    }
}

output {
    stdout { 
        codec => rubydebug
        }
}
10. Run the pipeline again and enter the http log again from step 6. Your output should now have changed to the following.
{
          "ident" => "-",
    "httpversion" => "1.1",
           "host" => "elkstack1",
     "@timestamp" => 2022-02-01T17:06:29.722Z,
       "clientip" => "83.149.9.216",
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
      "timestamp" => "04/Jan/2015:05:13:42 +0000",
       "response" => "200",
         "method" => "GET",
           "auth" => "-",
          "bytes" => "203023",
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
       "@version" => "1",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\""
}
# Lab 3
#### Task 1: using mypipeline.conf, change the output to send data to elasticsearch.
1. Edit your mypipeline.conf to send to elasticsearch to the http-logs index.
2. Rerun the logstash pipeline from the previous lab against your conf file.

<details><summary>Hint</summary>

```
Check the Output learning section from the beginning.
```
</details>

<details><summary>Solution</summary>

```
output {
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "http-logs"
    }
}

sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/mypipeline.conf
```
</details>

#### Task 2: Use Console in Dev tools in Kibana to query for all logs in http-logs index

Connect to kibana with your favorite web browser at <server_ip>:5601

<details><summary>Solution</summary>

```
GET /http-logs/_search
{
  "query": {
    "match_all": {}
  }
}
```
</details>

# Lab 4
#### Task 1: You have been given a sample of a linux log.  Using the Grok debugger in Kibana, determine the proper Grok pattern from github https://github.com/logstash-plugins/logstash-patterns-core/tree/main/patterns/ecs-v1. 

##### Log Sample
```
Jul  9 22:53:19 combo ftpd[24085]: connection from 206.196.21.129 (host129.206.196.21.maximumasp.com) at Sat Jul  9 22:53:19 2005
```

<details><summary>Hint</summary>

```
https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/linux-syslog
```
</details>

<details><summary>Solution</summary>

```
%{SYSLOGLINE}
```
</details>

#### Task 2: Create a new .conf file called "linux.conf" and input the proper fields to ingest the data in a stdin, apply the proper grok pattern, and send it to elasticsearch in linux-logs index.

#### Task 3: Run Logstash with the new .conf file.

<details><summary>Solution</summary>

```

input {
    stdin { }
}
    
filter {
    grok {
        match => { "message" => "%{SYSLOGLINE}"} 
    }
}

output {
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "linux-logs"
    }
}



sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/linux.conf
```
</details>

#### Task 4: In the Console in Dev tools in Kibana, search the linux-logs index for your data.

<details><summary>Solution</summary>

```
GET /linux-logs/_search
{
  "query": {
    "match_all": {}
  }
}
```
</details>


```python

```

# Breaking changes in 8.0
Here are the breaking changes for 8.0.

## Secure communication with Elasticsearch
Logstash must establish a Secure Sockets Layer (SSL) connection before it can transfer data to an on-premise Elasticsearch cluster. Logstash must have a copy of the Elasticsearch CA that signed the cluster’s certificates. When a new Elasticsearch cluster is started up without dedicated certificates, it generates its own default self-signed Certificate Authority at startup.

Our hosted Elasticsearch Service simplifies safe, secure communication between Logstash and Elasticsearch. Elasticsearch Service uses certificates signed by standard publicly trusted certificate authorities, and therefore setting a cacert is not necessary.

For more information, see Elasticsearch security on by default.

## Java 11 minimum
Logstash requires Java 11 or later. By default, Logstash will run with the bundled JDK, which has been verified to work with each specific version of Logstash, and generally provides the best performance and reliability.

## Support for JAVA_HOME removed
We’ve removed support for using JAVA_HOME to override the path to the JDK. Users who need to use a version other than the bundled JDK should set the value of LS_JAVA_HOME to the path of their preferred JDK. The value of JAVA_HOME will be ignored.

## ECS compatibility is now on by default
Many plugins can now be run in a mode that avoids implicit conflict with the Elastic Common Schema. This mode is controlled individually with each plugin’s ecs_compatibility option, which defaults to the value of the Logstash pipeline.ecs_compatibility setting. In Logstash 8, this compatibility mode will be on-by-default for all pipelines. #11623

If you wish to lock in a pipeline’s behaviour from Logstash 7.x before upgrading to Logstash 8, you can set pipeline.ecs_compatibility: disabled to its definition in pipelines.yml (or globally in logstash.yml).

## Ruby Execution Engine removed
The Java Execution Engine has been the default engine since Logstash 7.0, and works with plugins written in either Ruby or Java. Removal of the Ruby Execution Engine will not affect the ability to run existing pipelines. #12517

## Support for UTF-16
We have added support for UTF-16 and other multi-byte-character when reading log files. #9702

## Field Reference parser configuration setting removed
The Field Reference parser interprets references to fields in your pipelines and plugins. Its behavior was configurable in 6.x, and 7.x allowed only a single option: strict. 8.0 no longer recognizes the setting, but maintains the same behavior as the strict setting. Logstash rejects ambiguous and illegal inputs as standard behavior.


```python

```
