# Optional: Installation of Opnsense
- Opnsense is an open source firewall
- The installation file can be downloaded from the official Opnsense page (https://opnsense.org/download/)

## Step 1 - Create network topology
- Before using Opnsense, we first need to determine how the network is structured and where the firewall is located in this network.
- This reference implementation uses the following network topology:

![Network Topology](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/network-topology.png?raw=true)


## Step 2 - Determine the Opnsense implementation 
- Before installing Opnsense, we need to determine how we want to install and operate Opnsense.
- In this reference implementation, we use a suitable [Hardware](https://www.amazon.de/gp/product/B09QM69CJ8/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&th=1), um Opnsense zu installieren. 

![Hardware for Firewall](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/firewall1.png?raw=true)
![Hardware for Firewall 2](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/firewall2.png?raw=true)

## Step 3 - Create an Opnsense boot-stick
- We can use the [official instructions](https://docs.opnsense.org/manual/install.html) as a guide


---

# 1. Installation of Wazuh 
## Step 1 - Preparation
- First, the system is updated and prepared for Docker operation:
- The following file is edited first: 'nano /etc/sysctl.conf'
- Then the following is added: `vm.max_map_count=262144`
- The file is saved and closed. The following commands are executed:

```bash
apt update; sudo apt upgrade -y
apt install docker
apt install docker-compose
systemctl start docker
```

- Then the firewall is configured:

```bash
apt install ufw
ufw enable
ufw allow 1514
ufw allow 1515
ufw allow 1515
ufw allow 9200
ufw allow 55000
```

---

## Step 2 - Installation of Wazuh 
- Now we install Wazuh:
 
```bash
cd /opt/
git clone https://github.com/wazuh/wazuh-docker.git -b v4.9.1
cd wazuh-docker/single-node/
docker-compose -f generate-indexer-certs.yml run --rm generator
docker-compose up -d
```

- After installation, we can access the dashboard at: `https://127.0.0.1:443` (the server's IP address on port 443)

---

## Step 3 - Cronjobs for log forwarding to Wazuh

```bash
apt install cron
systemctl enable cron
systemctl start cron
cd /root/
mkdir logs
```

- Enter the following command: `crontab -e`
- Add the following to crontab: `*/5 * * * * docker cp single-node_wazuh.manager_1:/var/ossec/logs/alerts/alerts.json /root/logs/`
- Then: `systemctl restart cron`

---
# 2. Installation of Logstash
## Step 1 - Installing Logstash

```bash
cd /etc/
mkdir /etc/sysconfig
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install logstash
cd /etc/logstash/
mkdir certs
```

---

## Step 2 - Configuration of Logstash 
- Now we copy the Elasticsearch CA into the following directory and set the following permissions: `chmod -R 755 CA.crt` und ändern die Berechtigungen: `chown logstash:logstash CA.crt` (sowie den Ordner, in dem sich die Datei befindet `chown logstash:logstash /etc/logstash/certs`)
- We then execute the following commands:

```bash
sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-elasticsearch
sudo chmod -R 755 /etc/logstash/certs/elastic-ca.crt
sudo mkdir /etc/logstash/templates
sudo curl -o /etc/logstash/templates/wazuh.json https://packages.wazuh.com/integrations/elastic/4.x-8.x/dashboards/wz-es-4.x-8.x-template.json
echo 'LOGSTASH_KEYSTORE_PASS="k3txcvSJGNsdjfkn#asd#"'| sudo tee /etc/sysconfig/logstash
export LOGSTASH_KEYSTORE_PASS=k3txcvSJGNsdjfkn#asd#
sudo chown root /etc/sysconfig/logstash
sudo chmod 600 /etc/sysconfig/logstash
sudo systemctl start logstash
```

- In the next step, we enter the elasticsearch username and password:

```bash
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash create
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ELASTICSEARCH_USERNAME
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ELASTICSEARCH_PASSWORD
```

- Then we continue with the following commands:

```bash
sudo touch /etc/logstash/conf.d/wazuh-elasticsearch.conf
sudo systemctl stop logstash
sudo systemctl enable logstash.service
sudo systemctl start logstash.service
```

---

# 3. Installation of Elasticsearch and Kibana
## Installation and Configuration of Elasticsearch and Kbiana
### Step 1 - Preparation
- First, we need to prepare the system for the Elastic Stack:

```bash
sudo apt update; sudo apt upgrade -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
sudo apt install unzip
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

### Step 2 - Firewall Configuration
- Now we will open the necessary ports on the server that are required for the Elastic Stack to function correctly:

```bash
ufw enable
ufw allow 9200
ufw allow 5601
ufw allow 8220
```

---

### Step 3 - Installation of Elasticsearch and Kibana
- Now we install both elasticsearch and kibana
- **IMPORTANT:** Once elasticsearch is installed, the terminal will display the **login data** for the elasticsearch user. Save it!

```bash
sudo apt-get update && sudo apt-get install elasticsearch
sudo apt-get update && sudo apt-get install kibana
```

---

### Step 4 - Starting Elasticsearch for the first time
- Now we need to configure Elasticsearch
- The Elasticsearch configuration files are located at `/etc/elasticsearch/`
- We need to configure the following file: `elasticsearch.yml`
- Add the following fields to the configuration file (**IMPORTANT:** replace the IP address with the server's IP address):

```YAML
cluster.name: ELK
network.host: 192.168.100.10
http.port: 9200
```

- After we have made the changes and saved the file, we start elasticsearch: `sudo systemctl start elasticsearch`.
- Check the availability of elasticsearch at: https://192.168.100.10:9200 (replace this with the server's IP address)
- Ignore the warning and continue
- Enter the elastic user data (it was automatically generated earlier. See step 4) 

---

### Step 5 - Provision of Certificates 
- Stop Elasticsearch: `sudo systemctl stop elasticsearch`
- Now we will install the SSL certificates so that the connection to our Elastic instance is encrypted.
- Install the certificates:

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

- Open the `elasticsearch.yml` again and add the following fields:

```YAML
certificate: /etc/elasticsearch/certs/elastic/elastic.crt
key: /etc/elasticsearch/certs/elastic/elastic.key
certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt
```

- After we have made the changes and saved the file, we start elasticsearch: `sudo systemctl start elasticsearch`
- Access elasticsearch via the browser: `https://192.168.100.10:9200` (replace with the server's IP address)
- And check the SSL certificate. It should now be provided.

---

### Step 6 - Providing certificates to KibanaKibana 
- Now that elasticsearch is running, let's repeat this process for kibana.
- First, stop elasticsearch: sudo systemctl stop elasticsearch
- Then run the following commands (**replace the IP with the server's IP**):
 
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

### Step 7 - Kibana Keystore
- Now we will create the Kibana keystore for secure authentication between elasticsearch and Kibana (SAVE THE TOKEN):

```bash
/usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana_token
cd /etc/elasticsearch/
chown -R elasticsearch:elasticsearch service_tokens

/usr/share/kibana/bin/kibana-keystore add elasticsearch.serviceAccountToken
cd /etc/kibana/ 
chown -R kibana:kibana ./
```

---

### Step 8 - Configuration of Kibana 
- The Kibana configuration files are in the following directory: `/etc/kibana/`
- Insert the following lines into the file `kibana.yml`:

```YAML
server.port: 5601
server.host: "0.0.0.0"
server.publicBaseUrl: "https://192.168.100.10:5601"

server.ssl.enabled: true
server.ssl.certificateAuthorities: ["/etc/kibana/certs/elastic/ca.crt"]
server.ssl.certificate: /etc/kibana/certs/kibana/kibana.crt
server.ssl.key: /etc/kibana/certs/kibana/kibana.key
elasticsearch.hosts: ["https://192.168.100.10:9200"]

elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/elastic/ca.crt" ]
elasticsearch.ssl.verificationMode: full
```
- Start elasticsearch: `sudo systemctl start elasticsearch`
- Start kibana: `sudo systemctl start kibana`

---

### Schritt 9 - Installation of Fleet Servers
- First, we need to create new SSL certificates for the fleet server:

```bash
cd /etc/
mkdir certs/
cd certs/
mkdir elastic/ 
cp /etc/elasticsearch/certs/ca/ca.crt elastic/
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --out /etc/certs/fleet.zip --name fleet --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 192.168.100.10 --pem
cd /etc/certs/
unzip fleet.zip
```

- Then we perform the following steps:

1. Go to the Kibana Dashboard -> Open Menu -> Fleet -> Settings -> Outputs -> Edit
2. Change Hosts: `https://192.168.100.10:9200`
3. Advanced YAML Configuration: `ssl.certificate_authorities: ["/etc/certs/elastic/ca.crt"]`
4. Save and apply settings
5. Go to Agents
6. Click on _Add Fleet Server_ -> _Advanced_ -> _Create policy_ -> _Production_ -> Name: fleet -> URL: https://192.168.100.10:8220 -> Add host -> Generate Service Token
7. We then get a command and we save it

- We then execute the following commands:

```bash
cd /root/ 
touch install.sh 
chmod 755 install.sh 
```

- We open this new file with `nano install.sh` and copy the installation command: 

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.15.3-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.15.3-linux-x86_64.tar.gz
cd elastic-agent-8.15.3-linux-x86_64
sudo ./elastic-agent install --url=https://192.168.100.10:8220 \
  --fleet-server-es=https://192.168.100.10:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3MjkzMjU4NDM3NTk6S1k3TTg2U2tUNzJWUTdGMGI2bjQ3UQ \
  --fleet-server-policy=fleet-server-policy \
  --certificate-authorities=/etc/certs/elastic/ca.crt \
  --fleet-server-es-ca=/etc/certs/elastic/ca.crt \
  --fleet-server-cert=/etc/certs/fleet/fleet.crt \
  --fleet-server-cert-key=/etc/certs/fleet/fleet.key \
  --fleet-server-port=8220
```

---

### Step 10 - Kibana encryption keys
- Now we will install the Kibana encryption keys: `/usr/share/kibana/bin/kibana-encryption-keys`
- We receive an output in the terminal
- We add these keys to the file `kibana.yml`:

```YAML
xpack.encryptedSavedObjects.encryptionKey: 26b693e4a6bde560069207dabe49a865
xpack.reporting.encryptionKey: 409f2b25ec999c70dcc64a441a1436ec
xpack.security.encryptionKey: 0784cd5158afe89ec71e684972625062
```

- Then we start Kibana: `systemctl restart kibana`
- Both Elasticsearch and Kibana are now ready for use. The installation is complete.
