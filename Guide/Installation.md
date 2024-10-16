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

