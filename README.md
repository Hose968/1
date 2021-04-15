# Unified logging system with proven logs on blockchain
## Posted by CodeDefenders team
----
[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Our solution would:
- Provide logging of data of different systems 
of different nations in a single format.
- Does not require additional hardware, and 
easily integrates with existing systems.
- Allows you to detect the fact of logs forging 
- Compiles events from logs of different 
systems in a single sequence in time and place.
---
## The solutions we use

- Fluentd(td-agent)
- Elasticsearch
- Kibana
- ProuvenDB
---
## Overview

Elasticsearch, Fluentd, and Kibana (EFK) allow you to collect, index, search, and visualize log data. This is a great alternative to the proprietary software Splunk, which lets you get started for free, but requires a paid license once the data volume increases.

---
## Prerequisites

- Droplet with Ubuntu 14.04
 - User with sudo privileges
---
## Installing and Configuring Elasticsearch
### Getting Java
Elasticsearch requires Java, so the first step is to install Java.

```sh
$ sudo apt-get update && sudo apt-get install default-jre -y
```
Check that Java was indeed installed. Run:
```sh
$ java -version
```
### Getting Elasticsearch
Next, download and install Elasticsearch’s deb package as follows.
```sh
$ sudo wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.2.2.deb
$ sudo dpkg -i elasticsearch-1.2.2.deb
```
### Starting Elasticsearch

Start running Elasticsearch with the following command.
```sh
$ sudo service elasticsearch start
```
---
## Installing and Configuring Kibana

### Getting Kibana

Move to your home directory:
```sh
$ cd ~
```
We will download Kibana as follows:

```sh
$ curl -L https://download.elasticsearch.org/kibana/kibana/kibana-3.1.0.tar.gz | tar xzf -
$ sudo cp -r kibana-3.1.0 /usr/share/
$ sudo -i service kibana start
$ service kibana status
```
---
## Installing and Configuring Fluentd
Finally, let’s install Fluentd. We will use td-agent, the packaged version of Fluentd, built and maintained by Treasure Data.

### Installing Fluentd via the td-agent package
Install Fluentd with the following commands:
```sh
$ wget http://packages.treasuredata.com/2/ubuntu/trusty/pool/contrib/t/td-agent/td-agent_2.0.4-0_amd64.deb
$ sudo dpkg -i td-agent_2.0.4-0_amd64.deb
```
---
### Installing Plugins
We need a couple of plugins:

-    out_elasticsearch: this plugin lets Fluentd to stream data to Elasticsearch.
 -   outrecordreformer: this plugin lets us process data into a more useful format.

The following commands install both plugins (the first apt-get is for out_elasticsearch: it requires ``make`` and ``libcurl``)
```sh
$ sudo apt-get install make libcurl4-gnutls-dev --yes
$ sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-elasticsearch
$ sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-record-reformer
```
Next, we configure Fluentd to listen to syslog messages and send them to Elasticsearch. Open ``/etc/td-agent/td-agent.conf`` and add the following lines at the top of the file:
```
<source>
 @type syslog
 port 5140
 tag  system
</source>
<match system.*.*>
 @type record_reformer
 tag elasticsearch
 facility ${tag_parts[1]}
 severity ${tag_parts[2]}
</match>
<match elasticsearch>
 @type copy
 <store>
   @type stdout
 </store>
 <store>
 @type elasticsearch
 logstash_format true
 flush_interval 5s #debug
 </store>
</match>
```
### Starting Fluentd
Start Fluentd with the following command:
```
$ sudo service td-agent start
```
------
## Forwarding rsyslog Traffic to Fluentd
Ubuntu 14.04 ships with rsyslogd. It needs to be reconfigured to forward syslog events to the port Fluentd listens to (port 5140 in this example).

Open ``/etc/rsyslog.conf`` (you need to ``sudo``) and add the following line at the top

```
*.* @127.0.0.1:5140
```
After saving and exiting the editor, restart rsyslogd as follows:
```
$ sudo service rsyslog restart
```
-----
## Setting Up Kibana Dashboard Panels
Kibana’s default panels are very generic, so it’s recommended to customize them. Here, we show two methods.
### Method 1: Using a Template
The Fluentd team offers an alternative Kibana configuration that works with this setup better than the default one. To use this alternative configuration, run the following command:

```
$ wget -O default.json https://assets.digitalocean.com/articles/fluentd/default.json
$ sudo cp default.json /usr/share/kibana-3.1.0/app/dashboards/default.json
```
Note: The original configuration file is from the author’s GitHub gist.

If you refresh your Kibana dashboard home page at your server’s URL, Kibana should now be configured to show histograms by syslog severity and facility, as well as recent log lines in a table.
### Method 2: Manually Configuring
Go to your server’s IP address or domain to view the Kibana dashboard.
![img1](https://assets.digitalocean.com/articles/fluentd/kibana_welcome.png)
There are a couple of starter templates, but let’s choose the blank one called Blank Dashboard: I’m comfortable configuring on my own, shown at the bottom of the welcome text.
![img2](https://assets.digitalocean.com/articles/fluentd/kibana_blank.png)
Next, click on the + ADD A ROW button on the right side of the dashboard. A configuration screen for a new row (a row consists of one or more panels) should show up. Enter a title, press the Create Row button, followed by Save. This creates a row.
![img3](https://assets.digitalocean.com/articles/fluentd/kibana_row.png)
When an empty row is created, Kibana shows the prompt Add panel to empty row on the left. Click this button. It takes you to the configuration screen to add a new panel. Choose histogram from the dropdown menu. A histogram is a time chart; for more information, see Kibana’s documentation.
![img4](https://assets.digitalocean.com/articles/fluentd/kibana_histogram.png)
There are many parameters to configure for a new histogram, but you can just scroll down and press the Save button. This creates a new panel.
![img5](https://assets.digitalocean.com/articles/fluentd/kibana_histogram_details.png)
---------------------
## More Information
- Fluentd Doc`s :  https://docs.fluentd.org/
- Elastic (Elasticsearch and Kibana) Doc`s : https://www.elastic.co/guide/index.html
- ProvenDB usage git : https://github.com/SouthbankSoftware/provenlogs
---
