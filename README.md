# ELK Stack (Elasticsearch, Logstash, Kibana) Setup on Ubuntu

This guide provides a complete step-by-step process to set up the **ELK Stack** (Elasticsearch, Logstash, Kibana) on **Ubuntu**, enabling **secure Kibana access** and **remote log collection** without installing additional software on client servers.

## 📌 Prerequisites
- Ubuntu 20.04 or later
- Root or sudo privileges
- At least 4GB RAM (Recommended)

## 🚀 Step 1: Install Java (Required for ELK)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-11-jdk -y
java -version
```

## 📊 Step 2: Install and Configure Elasticsearch

### 1. Add Elasticsearch Repository:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" > /etc/apt/sources.list.d/elastic-8.x.list'
sudo apt update
```

### 2. Install Elasticsearch:

```bash
sudo apt install elasticsearch -y
```

### 3. Enable Security in Elasticsearch:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Add the following line:

```yaml
xpack.security.enabled: true
```

Save and exit: `CTRL + X` → `Y` → `Enter`

### 4. Start and Enable Elasticsearch:

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

### 5. Set Up User Passwords:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

Set passwords for the following users:

- **elastic**: AdminPass123
- **kibana_system**: KibanaPass456

## 📈 Step 3: Install and Configure Kibana

### 1. Install Kibana:

```bash
sudo apt install kibana -y
```

### 2. Configure Kibana Authentication:

```bash
sudo nano /etc/kibana/kibana.yml
```

Add the following lines:

```yaml
elasticsearch.username: "kibana_system"
elasticsearch.password: "KibanaPass456"
xpack.security.enabled: true
server.host: "0.0.0.0"
```

Save and exit: `CTRL + X` → `Y` → `Enter`

### 3. Start and Enable Kibana:

```bash
sudo systemctl enable kibana
sudo systemctl restart kibana
```

## 📊 Step 4: Install and Configure Logstash

### 1. Install Logstash:

```bash
sudo apt install logstash -y
```

### 2. Configure Logstash for UDP Syslog Input:

```bash
sudo nano /etc/logstash/conf.d/syslog.conf
```

Add the following configuration:

```yaml
input {
  udp {
    port => 514
    type => "syslog"
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    user => "elastic"
    password => "AdminPass123"
    index => "syslog-%{+YYYY.MM.dd}"
  }
}
```

Save and exit: `CTRL + X` → `Y` → `Enter`

### 3. Start and Enable Logstash:

```bash
sudo systemctl enable logstash
sudo systemctl restart logstash
```

Check the status to verify:

```bash
sudo systemctl status logstash
```

## 📥 Step 5: Configure Remote Log Sending (Client Servers)

### 1. Edit `rsyslog` Configuration on Client Server:

```bash
sudo nano /etc/rsyslog.conf
```

Find and add this line at the end:

```bash
*.* @ELK-SERVER-IP:514
```

Replace `ELK-SERVER-IP` with the IP address of your ELK server.

### 2. Restart `rsyslog` on Client Server:

```bash
sudo systemctl restart rsyslog
```

## 🔍 Step 6: Access Kibana and View Logs

1. Open your browser and visit:

```
http://YOUR_ELK_SERVER_IP:5601
```

2. Log in using the credentials:

- **Username:** elastic
- **Password:** AdminPass123

### Create an Index Pattern:

1. Navigate to **Management** > **Index Patterns**.
2. Create a new index pattern: `syslog-*`
3. Confirm and save.

## 🔒 Step 7: Ensure Services Start on Boot

Enable all services to start automatically:

```bash
sudo systemctl enable elasticsearch kibana logstash
```

## ✅ Final Setup Summary

- **ELK Stack** is fully installed and secured with username and password.
- **Kibana** is protected and accessible via login.
- **Client servers** send logs without installing additional software.

Enjoy real-time monitoring and log analysis! 🎉

