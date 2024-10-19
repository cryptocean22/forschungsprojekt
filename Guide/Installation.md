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
- Now that elasticsearch is up and running, we will repeat this process for kibana.
- First, stop elasticsearch: `sudo systemctl stop elasticsearch`
- Then, run the following commands (**replace the IP with the IP of your server**):
 
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
- Now we create the kibana keystore for secure authentication between elasticsearch and kibana (SAVE THE TOKEN):

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
- The kibana configuration files are located at: `/etc/kibana/`
- Add the following fields to the `kibana.yml`

```YAML
server.port: 5601
server.host: "0.0.0.0"
server.publicBaseUrl: "https://192.168.1.42:5601"

server.ssl.enabled: true
server.ssl.certificateAuthorities: ["/etc/kibana/certs/elastic/ca.crt"]
server.ssl.certificate: /etc/kibana/certs/kibana/kibana.crt
server.ssl.key: /etc/kibana/certs/kibana/kibana.key
elasticsearch.hosts: ["https://192.168.1.42:9200"]

elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/elastic/ca.crt" ]
elasticsearch.ssl.verificationMode: full
```
- Start elasticsearch: `sudo systemctl start elasticsearch`
- Start kibana: `sudo systemctl start kibana`

---

### Step 10 - Installing the Fleet Server

- First we need to create new SSL-Certificates for the Fleet-Server:

```bash
cd /etc/
mkdir certs/
cd certs/
mkdir elastic/ 
cp /etc/elasticsearch/certs/ca/ca.crt elastic/
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --out /etc/certs/fleet.zip --name fleet --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --ip 192.168.0.87 --pem
cd /etc/certs/
unzip fleet.zip
```

- Then, follow the next steps:

1. Go to the Kibana Dashboard -> Open Menu -> Fleet -> Settings -> Outputs -> Edit
2. Change Hosts: `https://192.168.0.87:9200`
3. Advanced YAML Configuration: `ssl.certificate_authorities: ["/etc/certs/elastic/ca.crt"]`
4. Save and apply settings
5. Go to Agents
6. Click on _Add Fleet Server_ -> _Advanced_ -> _Create policy_ -> _Production_ -> Name: fleet -> URL: https://192.168.178.128:8220 -> Add host -> Generate Service Token
7. We then get a command and we save it

- We then run the following commands:

```bash
cd /root/ 
touch install.sh 
chmod 755 install.sh 
```

- We then open this new file with `nano install.sh` and paste the command: 

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.15.3-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.15.3-linux-x86_64.tar.gz
cd elastic-agent-8.15.3-linux-x86_64
sudo ./elastic-agent install --url=https://192.168.1.42:8220 \
  --fleet-server-es=https://192.168.1.42:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3MjkzMjU4NDM3NTk6S1k3TTg2U2tUNzJWUTdGMGI2bjQ3UQ \
  --fleet-server-policy=fleet-server-policy \
  --certificate-authorities=/etc/certs/elastic/ca.crt \
  --fleet-server-es-ca=/etc/certs/elastic/ca.crt \
  --fleet-server-cert=/etc/certs/fleet/fleet.crt \
  --fleet-server-cert-key=/etc/certs/fleet/fleet.key \
  --fleet-server-port=8220
```

---

### Step 11 
- Now we will install the kibana encrytion keys: `/usr/share/kibana/bin/kibana-encryption-keys`
- We will get an output in the terminal.
- We will paste these encryption keys into the `kibana.yml`

```YAML
xpack.encryptedSavedObjects.encryptionKey: 26b693e4a6bde560069207dabe49a865
xpack.reporting.encryptionKey: 409f2b25ec999c70dcc64a441a1436ec
xpack.security.encryptionKey: 0784cd5158afe89ec71e684972625062
```

- Then we restart kibana: `systemctl restart kibana`
- Both Elasticsearch and Kibana are now ready to use. The installation is complete.
