# Installation of WELK
## Installing and Configuring Elasticsearch and Kbiana
### Step 1 - Preparation
- First, we need to prepare the system for the deployment of the elastic stack:

```bash
sudo apt update; sudo apt upgrade -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
sudo apt install unzip
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

### Step 2 - Firewall Configuration
- Now we open the necessary ports on the server that are required for the Elastic Stack to work properly

```bash
ufw enable
ufw allow 9200
ufw allow 5601
ufw allow 8220
```

---

### Step 3 - Installation of Elasticsearch and Kibana
- Now we install both elasticsearch and kibana
- **IMPORTANT:** Once elasticsearch has been installed, the terminal will display the **credentials** for the elasticsearch user. Save them! 

```bash
sudo apt-get update && sudo apt-get install elasticsearch
sudo apt-get update && sudo apt-get install kibana
```

---

### Step 4 - Starting Elasticsearch for the first time 
- Now we need to configure Elasticsearch
- The Elasticsearch configuration files are located at `/etc/elasticsearch/`
- We need to configure the following file: `elasticsearch.yml`
- Add the following fields to your configuration file (**IMPORTANT:** replace the IP address with the IP address of your server):

```YAML
cluster.name: ELK
network.host: 192.168.1.42
http.port: 9200
```

- After making the changes and saving the file, start elasticsearch: `sudo systemctl start elasticsearch`
- Check the availability of elasticsearch by visiting: https://192.168.1.42:9200 (replace with the server's IP-Address)
- Ignore the warning and continue
- Enter your elastic user credentials (they have been automatically generated earlier. See Step 4) 

---

### Step 5 - Certificate Deployment 
- Stop elasticsearch: `sudo systemctl stop elasticsearch`
- Now we deploy the SSL certificates so that the connection to our elastic instance is encrypted.
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

- Open the `elasticsearch.yml` again and add the following fields to your configuration file:

```YAML
certificate: /etc/elasticsearch/certs/elastic/elastic.crt
key: /etc/elasticsearch/certs/elastic/elastic.key
certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt
``` 
- After making the changes and saving the file, start elasticsearch: `sudo systemctl start elasticsearch`
- Visit elasticsearch via the browser again: https://192.168.1.42:9200 (replace with the server's IP-Address)
- And check the ssl-certificate. It should be deployed now. 


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




