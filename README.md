# ELK Stack Installation and Configuration Guide

## Prerequisites

- A Linux-based server (Ubuntu 20.04 recommended)
- At least 4GB RAM
- OpenJDK 11 installed

## Step 1: Install Elasticsearch

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.2-amd64.deb
sudo dpkg -i elasticsearch-8.17.2-amd64.deb
sudo systemctl enable --now elasticsearch
```

### Configure Elasticsearch

Edit the configuration file:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Set the following parameters:

```
network.host: 0.0.0.0
http.port: 9200
xpack.security.enabled: true
```

Restart Elasticsearch:

```bash
sudo systemctl restart elasticsearch
```

Set a password for `elastic` user:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

## Step 2: Install Kibana

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.17.2-amd64.deb
sudo dpkg -i kibana-8.17.2-amd64.deb
sudo systemctl enable --now kibana
```

Edit Kibana configuration:

```bash
sudo nano /etc/kibana/kibana.yml
```

Set:

```
server.host: "0.0.0.0"
elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.username: "elastic"
elasticsearch.password: "your_password_here"
```

Restart Kibana:

```bash
sudo systemctl restart kibana
```

## Step 3: Install Logstash

```bash
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.17.2-amd64.deb
sudo dpkg -i logstash-8.17.2-amd64.deb
```

Edit Logstash configuration:

```bash
sudo nano /etc/logstash/conf.d/logstash.conf
```

Example configuration:

```
input {
  http {
    port => 5044
  }
}
output {
  elasticsearch {
    hosts => ["https://localhost:9200"]
    user => "elastic"
    password => "your_password_here"
    ssl_certificate_verification => false
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

Restart Logstash:

```bash
sudo systemctl restart logstash
```

## Step 4: Test Logstash Input

Send a test log using `curl`:

```bash
curl -X POST "http://localhost:5044" -H "Content-Type: application/json" -d '{"message": "Test log from curl", "level": "INFO", "timestamp": "2025-03-02T12:00:00Z"}'
```

## Step 5: Verify Logs in Elasticsearch

```bash
curl -X GET -u elastic:your_password_here "https://localhost:9200/logstash-*/_search?pretty=true" -k
```

If everything is set up correctly, you should see your test log in the response.

## Step 6: Access Kibana

- Open a browser and go to `http://your_server_ip:5601`
- Login using `elastic` and the password you set earlier.
- Navigate to **Discover** to check logs.

