# ELK Stack Installation Guide on Ubuntu

This guide will walk you through installing and configuring the ELK Stack (Elasticsearch, Kibana, and Logstash) on a Linux server.

---

## Prerequisites

- Operating System: Ubuntu 20.04 (or similar versions)
- Root or sudo user access
- Internet access to download packages

---

## Installation Steps

### 1. Update System

Start by updating your system:

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Java (OpenJDK 11)

ELK requires Java. Install OpenJDK 11:

```bash
sudo apt install openjdk-11-jdk -y
java -version
```

### 3. Install Elasticsearch

To install Elasticsearch, add the repository and install the package:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
sudo apt update
sudo apt install elasticsearch -y
```

Then, enable and start Elasticsearch:

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

To check if Elasticsearch is running, use:

```bash
curl -X GET "localhost:9200/"
```

### 4. Install Kibana

To install Kibana:

```bash
sudo apt install kibana -y
sudo systemctl enable kibana
sudo systemctl start kibana
```

Configure Kibana for external access:

```bash
sudo nano /etc/kibana/kibana.yml
```

Find and change the following line:

```yaml
server.host: "0.0.0.0"
```

Then restart Kibana:

```bash
sudo systemctl restart kibana
```

### 5. Install Logstash

To install Logstash:

```bash
sudo apt install logstash -y
sudo systemctl enable logstash
sudo systemctl start logstash
```

Configure Logstash to receive data over HTTP:

```bash
sudo nano /etc/logstash/conf.d/logstash-http.conf
```

Add the following content:

```bash
input {
  http {
    host => "0.0.0.0"
    port => 5044
    codec => "json"
  }
}

filter {

}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

Then restart Logstash:

```bash
sudo systemctl restart logstash
```

### 6. Enable X-Pack Security

To enable X-Pack Security for Elasticsearch and Kibana, do the following:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Add the following lines to enable security:

```yaml
xpack.security.enabled: true
xpack.security.authc.realms.file.file1.enabled: true
```

Then restart Elasticsearch:

```bash
sudo systemctl restart elasticsearch
```

Run the setup password command for Elasticsearch:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

Next, configure Kibana to use the credentials:

```bash
sudo nano /etc/kibana/kibana.yml
```

Add the following lines:

```yaml
elasticsearch.username: "elastic"
elasticsearch.password: "your_password_here"
xpack.security.enabled: true
```

Restart Kibana:

```bash
sudo systemctl restart kibana
```

### 7. Configure Logstash for Authentication

Configure Logstash to authenticate when sending logs to Elasticsearch:

```bash
sudo nano /etc/logstash/conf.d/your-logstash-config.conf
```

Use the following configuration for Logstash:

```bash
output {
  elasticsearch {
    hosts => ["https://your-elasticsearch-ip:9200"]
    user => "elastic"
    password => "your_password_here"
    ssl => true
    cacert => '/etc/elasticsearch/certs/your-certificate.pem'
  }
}
```

Alternatively, you can use:

```bash
output {
  elasticsearch {
    hosts => ["http://172.16.20.31:9200"]
    user => "elastic"
    password => "your_password_here"
  }
}
```

### 8. Test Log Sending

To test sending a log to Elasticsearch using curl, run:

```bash
curl -u elastic:your_password_here -X POST "https://172.16.20.31:9200/logs/_doc/" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2025-02-24T12:30:00",
  "message": "This is a test log message",
  "host": "log-server",
  "severity": "info"
}'
```

Alternatively:

```bash
curl -u elastic:your_password_here -X POST "http://172.16.20.31:9200/logs/_doc/" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2025-02-24T12:30:00",
  "message": "This is a test log message",
  "host": "log-server",
  "severity": "info"
}'
```

