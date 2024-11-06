# University research project - OWELK (Opnsense, Wazuh, Elasticsearch, Logstash, Kibana)
This is the official repository for the research project as part of the master's programme. This project is concerned with the implementation of a holistic network monitoring tool using various security technologies so that micro-enterprises can monitor their network with as little effort and at low cost as possible. The security technologies used are: 

| Tool | Beschreibung |
| ----------- | ----------- |
| [Opnsense](https://opnsense.org/)      | An open-source firewall and routing platform that offers security features such as VPN, IDS/IPS and web filtering. |
| [Wazuh](https://wazuh.com/)         | An open-source security detection, monitoring and response platform that provides log analysis, threat detection and integrity checking. |
| [Elasticsearch](https://www.elastic.co/de/elasticsearch) | A distributed search and analysis engine for storing and quickly retrieving large amounts of data.     |
| [Logstash](https://www.elastic.co/de/logstash)      | A data processing pipeline tool that collects and processes logs and other data sources and sends them to various outputs, often to Elasticsearch. |
| [Kibana](https://www.elastic.co/de/kibana)        | A visualisation tool for Elasticsearch that creates dashboards and visualises data in real time. | 

---

## Reference implementation 
This repository contains a reference implementation of OWELK. By following the steps in the file `Installation.md`

**Agent Security Events**
![Agent Security Events](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/security-events.jpeg?raw=true)

**Agent Metrics**
![Agent Metrics](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/metrics.jpeg?raw=true)

**Agent Details**
![Agent Details](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/host-details.jpg?raw=true)

**Agent Details**
![Agent Details](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/wazuh-agent.jpeg?raw=true)

**OPNsense Dashboard**
![OPNsense Dashboard](https://github.com/cryptocean22/forschungsprojekt/blob/main/Guide/pictures/opnsensedashboard.png?raw=true)
