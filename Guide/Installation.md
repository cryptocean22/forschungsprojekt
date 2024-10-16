# Installation of WELK
## Installing and Configuring Elasticsearch and Kbiana
### Step 1 - Preparation
```bash
sudo apt update; sudo apt upgrade -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

### Step 2 - Firewall Configuration
```bash
ufw enable
ufw allow 9200
ufw allow 5601
ufw allow 8220
```

---

### Step 3 - Installation of Elasticsearch and Kibana

```bash
sudo apt-get update && sudo apt-get install elasticsearch
sudo apt-get update && sudo apt-get install kibana
```

---

### Step 4 - Starting Elasticsearch for the first time 
- Make changes to the YAML (see below) and start elasticsearch: `sudo systemctl start elasticsearch`


```
YAML ohne SSL 
```

---

### Step 5 - Certificate Deployment 
- Install certificates:

```bash
cd /etc/elasticsearch/certs/
/usr/share/elasticsearch/bin/elasticsearch-certutil ca --pem --out /etc/elasticsearch/certs/ca.zip
unzip ca.zip
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --out /etc/elasticsearch/certs/elastic.zip --name elastic --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 192.168.100.10 --pem
cd /etc/elasticsearch/certs/
unzip elastic.zip
chown -R elasticsearch:elasticsearch .
sudo systemctl daemon-reload
```


---

### Step 6 - Make Changes to the Elasticsearch YAML 
- Make changes to the elasticsearch YAML 

```
YAML mit SSL 
```

- Restart elasticsearch

---

### Step 7 - Certificate Deployment for Kibana 
```bash
cd /etc/kibana
mkdir certs/
cd certs/
mkdir elastic
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --out /etc/kibana/certs/kibana.zip --name kibana --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 192.168.100.10 --pem
unzip kibana.zip
cp /etc/elasticsearch/certs/ca/* /etc/kibana/certs/elastic/
cd /etc/elasticsearch/
chown -R elasticsearch:elasticsearch .
cd /etc/kibana/
chown -R kibana:kibana ./
```

---

### Step 8 - Kibana Keystore

```bash
/usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana_token
cd /etc/elasticsearch/
chown -R elasticsearch:elasticsearch service_tokens

/usr/share/kibana/bin/kibana-keystore add elasticsearch.serviceAccountToken
cd /etc/kibana/ 
chown -R kibana:kibana ./
```

---

### Step 9 - Configure Kibana 
- Make changes to the YAML (see below) 

```

```
- Start Kibana

---




