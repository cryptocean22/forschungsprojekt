# Example Configuration
## Directory of elasticsearch 
This is the structure `/etc/elasticsearch/`:

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

## Directory of kibana
This is the structure `/etc/kibana/`:

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
