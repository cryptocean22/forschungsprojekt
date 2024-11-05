# Beispiel Konfigurationen
## Verzeichnis von elasticsearch 
Dies ist das Verzeichnis von `/etc/elasticsearch/`:

```bash
.
├── certs
│   ├── ca
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   └── ca.srl
│   ├── ca.zip
│   ├── elastic
│   │   ├── elastic.crt
│   │   ├── elastic.csr
│   │   └── elastic.key
│   ├── elastic.zip
│   ├── http_ca.crt
│   ├── http.p12
│   └── transport.p12
├── elasticsearch.keystore
├── elasticsearch-plugins.example.yml
├── elasticsearch.yml
├── jvm.options
├── jvm.options.d
├── log4j2.properties
├── role_mapping.yml
├── roles.yml
├── service_tokens
├── users
└── users_roles
```

## Verzeichnis von kibana
Dies ist das Verzeichnis von `/etc/kibana/`:

```bash
/etc/kibana/
├── certs
│   ├── elastic
│   │   └── ca.crt
│   ├── kibana
│   │   ├── kibana.crt
│   │   ├── kibana.csr
│   │   └── kibana.key
│   └── kibana.zip
├── kibana.keystore
├── kibana.yml
└── node.options
```

## Verzeichnis von wazuh 
Dies ist das Verzeichnis von `/opt/wazuh-docker/single-node/`

```bash
wazuh-docker/single-node/
├── config
│   ├── certs.yml
│   ├── wazuh_cluster
│   │   └── wazuh_manager.conf
│   ├── wazuh_dashboard
│   │   ├── opensearch_dashboards.yml
│   │   └── wazuh.yml
│   ├── wazuh_indexer
│   │   ├── internal_users.yml
│   │   └── wazuh.indexer.yml
│   └── wazuh_indexer_ssl_certs  [error opening dir]
├── docker-compose.yml
├── generate-indexer-certs.yml
└── README.md

6 directories, 9 file

```
