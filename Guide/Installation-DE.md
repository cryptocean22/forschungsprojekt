# Optional: Installation von Opnsense
- Opnsense ist eine Open-Source Firewall
- Die Installationsdatei kann über die offizielle [Opnsense-Seite](https://opnsense.org/download/) heruntergeladen werden

## Schritt 1 - Netzwerktopologie erstellen
- Vor dem Einsatz von Opnsense muss zunächst festgelegt werden, wie das Netzwerk aufgebaut ist und wo sich die Firewall in diesem Netzwerk befindet.
- Diese Referenzimplementierung verwendet die folgende Netzwerktopologie:

![Network Topology](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/network-topology.png?raw=true)


## Schritt 2 - Opnsense Implementierung bestimmen 
- Vor der Installation von Opnsense muss festgelegt werden, wie Opnsense installiert und betrieben werden soll.
- In dieser Referenzimplementierung verwenden wir eine geeignete [Hardware](https://www.amazon.de/gp/product/B09QM69CJ8/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&th=1), um Opnsense zu installieren. 

![Hardware for Firewall](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/firewall1.png?raw=true)
![Hardware for Firewall 2](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/firewall2.png?raw=true)

## Schritt 3 - Opnsense Installations-Stick erstellen
- Wir können die [offizielle Anleitung](https://docs.opnsense.org/manual/install.html) als Orientierung verwenden


---

# 1. Installation von Wazuh 
## Schritt 1 - Vorbereitung
- Zunächst wird das System aktualisiert und auf den Betrieb von Docker vorbereitet:
- Folgende Datei wird zunächst bearbeitet: `nano /etc/sysctl.conf`
- Anschließend wird folgendes hinzugefügt: `vm.max_map_count=262144`
- Die Datei wird gespeichert und geschlossen. Die folgenden Befehle werden ausgeführt:

```bash
apt update; sudo apt upgrade -y
apt install docker
apt install docker-compose
systemctl start docker
```

- Anschließend wird die Firewall konfiguriert: 
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

## Schritt 2 - Installation von Wazuh 
- Jetzt installieren wir Wazuh:
 
```bash
cd /opt/
git clone https://github.com/wazuh/wazuh-docker.git -b v4.9.1
cd wazuh-docker/single-node/
docker-compose -f generate-indexer-certs.yml run --rm generator
docker-compose up -d
```

- Nach der Installation können wir auf das Dashboard zugreifen unter: https://127.0.0.1:443 (die IP-Adresse des Servers auf Port 443)

---

## Schritt 3 - Cronjobs für Log-Forwarding an Wazuh 

```bash
apt install cron
systemctl enable cron
systemctl start cron
cd /root/
mkdir logs
```

- Folgenden Befehl eingeben: `crontab -e`
- Folgendes in crontab hinzufügen: `*/5 * * * * docker cp single-node_wazuh.manager_1:/var/ossec/logs/alerts/alerts.json /root/logs/`
- Danach: `systemctl restart cron`

---
# 2. Installation von Logstash
## Schritt 1 - Logstash installieren

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

## Schritt 2 - Konfiguration von Logstash 
- Jetzt kopieren wir die Elasticsearch-CA in das folgende Verzeichnis und setzen die folgenden Berechtigungen: `chmod -R 755 CA.crt` und ändern die Berechtigungen: `chown logstash:logstash CA.crt` (sowie den Ordner, in dem sich die Datei befindet `chown logstash:logstash /etc/logstash/certs`)
- Wir führen dann die folgenden Befehle aus:

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

- Im nächsten Schritt geben wir den elasticsearch-Benutzernamen und das elasticsearch-Passwort ein:

```bash
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash create
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ELASTICSEARCH_USERNAME
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ELASTICSEARCH_PASSWORD
```

- Dann fahren wir mit den folgenden Befehlen fort:

```bash
sudo touch /etc/logstash/conf.d/wazuh-elasticsearch.conf
sudo systemctl stop logstash
sudo systemctl enable logstash.service
sudo systemctl start logstash.service
```

---

# 3. Installation von Elasticsearch and Kibana
## Installation und Konfiguration von Elasticsearch and Kbiana
### Schritt 1 - Vorbereitung
- Zunächst müssen wir das System für den Einsatz des Elastic Stack vorbereiten:

```bash
sudo apt update; sudo apt upgrade -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
sudo apt install unzip
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

### Schritt 2 - Firewall Konfiguration
- Nun öffnen wir die notwendigen Ports auf dem Server, die für die korrekte Funktion des Elastic Stack erforderlich sind:

```bash
ufw enable
ufw allow 9200
ufw allow 5601
ufw allow 8220
```

---

### Schritt 3 - Installation von Elasticsearch und Kibana
- Jetzt installieren wir sowohl elasticsearch als auch kibana
- **WICHTIG:** Sobald elasticsearch installiert ist, zeigt das Terminal die **Zugangsdaten** für den elasticsearch-Benutzer an. Speicher diese! 

```bash
sudo apt-get update && sudo apt-get install elasticsearch
sudo apt-get update && sudo apt-get install kibana
```

---

### Schritt 4 - Elasticsearch zum ersten Mal starten 
- Jetzt müssen wir Elasticsearch konfigurieren
- Die Elasticsearch-Konfigurationsdateien befinden sich unter `/etc/elasticsearch/`
- Wir müssen die folgende Datei konfigurieren: `elasticsearch.yml`
- Füge die folgenden Felder zu der Konfigurationsdatei hinzu (**WICHTIG:** ersetze die IP-Adresse durch die IP-Adresse des Servers):

```YAML
cluster.name: ELK
network.host: 192.168.100.10
http.port: 9200
```

- Nachdem wir die Änderungen vorgenommen und die Datei gespeichert haben, starten wir elasticsearch: `sudo systemctl start elasticsearch`.
- Überprüfe die Verfügbarkeit von elasticsearch unter: https://192.168.100.10:9200 (ersetze diese durch die IP-Adresse des Servers)
- Ignoriere die Warnung und fahre fort
- Gib die elastic-Benutzerdaten ein (sie wurden zuvor automatisch generiert. Siehe Schritt 4) 

---

### Schritt 5 - Bereitstellung von Zertifikaten 
- Elasticsearch anhalten an: `sudo systemctl stop elasticsearch`
- Jetzt installieren wir die SSL-Zertifikate, damit die Verbindung zu unserer Elastic-Instanz verschlüsselt ist.
- Installiere die Zertifikate:

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

- Öffne die `elasticsearch.yml` erneut und füge die folgenden Felder hinzu:

```YAML
certificate: /etc/elasticsearch/certs/elastic/elastic.crt
key: /etc/elasticsearch/certs/elastic/elastic.key
certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt
```

- Nachdem wir die Änderungen vorgenommen und die Datei gespeichert haben, starten wir elasticsearch: `sudo systemctl start elasticsearch`
- Rufe elasticsearch über den Browser auf: https://192.168.100.10:9200 (durch die IP-Adresse des Servers ersetzen)
- Und überprüfe das SSL-Zertifikat. Es sollte jetzt bereitgestellt werden.

---

### Schritt 6 - Zertifikatsbereitstellung für Kibana 
- Jetzt, wo elasticsearch läuft, wiederholen wir diesen Vorgang für kibana.
- Zuerst stoppen wir elasticsearch: sudo systemctl stop elasticsearch
- Führe dann die folgenden Befehle aus (**Ersetze die IP durch die IP des Servers**):
 
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

### Schritt 7 - Kibana Keystore
- Nun erstellen wir den Kibana-Keystore für die sichere Authentifizierung zwischen elasticsearch und Kibana (SAVE THE TOKEN):

```bash
/usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana_token
cd /etc/elasticsearch/
chown -R elasticsearch:elasticsearch service_tokens

/usr/share/kibana/bin/kibana-keystore add elasticsearch.serviceAccountToken
cd /etc/kibana/ 
chown -R kibana:kibana ./
```

---

### Schritt 8 - Konfiguration von Kibana 
- Die Kibana Konfigurationsdateien sind im folgenden Verzeichnis: `/etc/kibana/`
- Füge die folgenden Zeilen in die Datei `kibana.yml` ein:

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
- Starte elasticsearch: `sudo systemctl start elasticsearch`
- Starte kibana: `sudo systemctl start kibana`

---

### Schritt 9 - Installation des Fleet Servers
- Zuerst müssen wir neue SSL-Zertifikate für den Fleet-Server erstellen:

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

- Dann führen wir die folgenden Schritte aus:

1. Go to the Kibana Dashboard -> Open Menu -> Fleet -> Settings -> Outputs -> Edit
2. Change Hosts: `https://192.168.100.10:9200`
3. Advanced YAML Configuration: `ssl.certificate_authorities: ["/etc/certs/elastic/ca.crt"]`
4. Save and apply settings
5. Go to Agents
6. Click on _Add Fleet Server_ -> _Advanced_ -> _Create policy_ -> _Production_ -> Name: fleet -> URL: https://192.168.100.10:8220 -> Add host -> Generate Service Token
7. We then get a command and we save it

- Wir führen dann die folgenden Befehle aus:

```bash
cd /root/ 
touch install.sh 
chmod 755 install.sh 
```

- Wir öffnen diese neue Datei mit `nano install.sh` und kopieren den Installationsbefehl: 

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

### Schritt 10 - Kibana Verschlüsselungsschlüssel
- Jetzt werden wir die Kibana-Verschlüsselungsschlüssel installieren: `/usr/share/kibana/bin/kibana-encryption-keys`
- Wir erhalten eine Ausgabe im Terminal.
- Wir fügen diese Schlüssel in die Datei `kibana.yml`:

```YAML
xpack.encryptedSavedObjects.encryptionKey: 26b693e4a6bde560069207dabe49a865
xpack.reporting.encryptionKey: 409f2b25ec999c70dcc64a441a1436ec
xpack.security.encryptionKey: 0784cd5158afe89ec71e684972625062
```

- Dann starten wir Kibana: `systemctl restart kibana`
- Sowohl Elasticsearch als auch Kibana sind nun einsatzbereit. Die Installation ist abgeschlossen.
