# ELK Skript

## Elasticsearch Setup:
- Der öffentliche Schlüssel von Elasticsearch wird heruntergeladen und zu den apt-Schlüsseln hinzugefügt. 
- Elasticsearch-Apt-Repository wird zur sources.list hinzugefügt. 
- [Elasticsearch](tech_daten/elasticsearch_P1.md), [Kibana](tech_daten/kibana_P1.md), [Filebeat](tech_daten/filebeat_P1.md) und [Auditbeat](tech_daten/auditbeat_P1.md) werden installiert. 
- Ein Cron-Job für Elasticsearch Curator wird erstellt, um ältere Elasticsearch-Indizes zu löschen.

```bash
# Herunterladen des GPG-Schlüssels und Hinzufügen zum apt-Schlüsselring
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Hinzufügen des Elasticsearch-Apt-Repository zur sources.list
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
echo "deb [arch=amd64] https://packages.elastic.co/curator/5/debian stable main" | sudo tee -a /etc/apt/sources.list.d/curator-5.list

# Aktualisieren des Paketmanagers und Installieren der ELK-Software
apt-get -qq update
apt-get -qq install elasticsearch -y # 1st install elasticseatch to get JDK
export JAVA_HOME=/usr/share/elasticsearch/jdk && echo export JAVA_HOME=/usr/share/elasticsearch/jdk >>/etc/bash.bashrc
apt-get -qq install kibana filebeat auditbeat elasticsearch-curator -y

# Einrichten des Cron-Jobs für Elasticsearch Curator
cat >/etc/cron.daily/curator <<EOF
#!/bin/sh
curator_cli --host 192.168.38.105 delete_indices --filter_list '{"filtertype": "age", "source": "name", "timestring": "%Y.%m.%d", "unit": "days", "unit_count": 1, "direction": "older"}'  > /dev/null 2>&1
EOF
chmod +x /etc/cron.daily/curator
```

## Elasticsearch Konfiguration:
- Ein Elasticsearch-Keystore wird erstellt, und ein Bootstrap-Passwort wird festgelegt. 
- Ein Elasticsearch-Benutzer "vagrant" mit dem Passwort "vagrant" und der Rolle "superuser" wird erstellt. 
- Die Elasticsearch-Konfiguration wird erstellt, wobei der Netzwerkhost, der Clustername und andere Einstellungen festgelegt werden.

```bash
# Setzen des Bootstrap-Passworts und Erstellen des Benutzers "vagrant"
printf vagrant | /usr/share/elasticsearch/bin/elasticsearch-keystore add -x "bootstrap.password" -f
/usr/share/elasticsearch/bin/elasticsearch-users useradd vagrant -p vagrant -r superuser

# Erstellen der Elasticsearch-Konfiguration
cat >/etc/elasticsearch/elasticsearch.yml <<EOF
network.host: _eth1:ipv4_
discovery.type: single-node
cluster.name: cydef-es-cluster
node.name: \${HOSTNAME}
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
xpack.security.enabled: true
xpack.security.authc:
        api_key.enabled: true
        anonymous:
                username: anonymous
                roles: superuser
                authz_exception: false
EOF

cat >/etc/default/elasticsearch <<EOF
ES_PATH_CONF=/etc/elasticsearch
ES_STARTUP_SLEEP_TIME=5
MAX_OPEN_FILES=65536
MAX_LOCKED_MEMORY=unlimited
EOF

mkdir /etc/systemd/system/elasticsearch.service.d/
cat >/etc/systemd/system/elasticsearch.service.d/override.conf <<EOF
[Service]
LimitMEMLOCK=infinity
EOFcat >/etc/default/elasticsearch <<EOF
ES_PATH_CONF=/etc/elasticsearch
ES_STARTUP_SLEEP_TIME=5
MAX_OPEN_FILES=65536
MAX_LOCKED_MEMORY=unlimited
EOF

mkdir /etc/systemd/system/elasticsearch.service.d/
cat >/etc/systemd/system/elasticsearch.service.d/override.conf <<EOF
[Service]
LimitMEMLOCK=infinity
EOF
```

## Systemkonfiguration:
- Limits für Elasticsearch werden in der limits.conf und in den systemd-Service-Overrides festgelegt. 
- Elasticsearch wird als systemd-Dienst gestartet und aktiviert.

```bash
# Festlegen der Limits für Elasticsearch
cat >/etc/security/limits.conf <<EOF
elasticsearch soft nofile 65536
elasticsearch hard nofile 65536
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
EOF

# Reload des systemd-Dienst-Managers und Starten von Elasticsearch als systemd-Dienst
/bin/systemctl daemon-reload
/bin/systemctl enable elasticsearch.service
/bin/systemctl start elasticsearch.service
```

## Kibana Setup:
- Das Kibana-Log-Verzeichnis wird erstellt und konfiguriert. 
- Die Kibana-Konfiguration wird erstellt, wobei der Serverhost und die Elasticsearch-Hosts festgelegt werden. 
- Kibana wird als systemd-Dienst gestartet und aktiviert.

```bash
# Erstellen des Kibana-Log-Verzeichnisses
touch /var/log/kibana.log
chown kibana:kibana /var/log/kibana.log

# Erstellen der Kibana-Konfiguration
cat >/etc/kibana/kibana.yml <<EOF
server.host: "192.168.38.105"
elasticsearch.hosts: ["http://192.168.38.105:9200"]
logging.dest: "/var/log/kibana.log"
kibana.defaultAppId: "discover"
telemetry.enabled: false
telemetry.optIn: false
newsfeed.enabled: false
xpack.security.enabled: true
xpack.ingestManager.fleet.tlsCheckDisabled: true
xpack.encryptedSavedObjects.encryptionKey: 'fhjskloppd678ehkdfdlliverpoolfcr'
EOF

# Enable und Starten des Kibana-Dienstes
/bin/systemctl enable kibana.service
/bin/systemctl start kibana.service
```

## Filebeat Setup:
- Die Filebeat-Konfiguration wird erstellt, wobei Pfade zu Log-Dateien und Verbindungsdaten zu Elasticsearch festgelegt sind. 
- Filebeat-Module für osquery, Zeek und Suricata werden aktiviert. 
- Filebeat wird als systemd-Dienst gestartet und aktiviert.

```bash
# Erstellen der Filebeat-Konfiguration
cat >/etc/filebeat/filebeat.yml <<EOF
filebeat.inputs:
- type: log
  enabled: false
  paths:
    - /var/log/auth.log
    - /var/log/syslog

filebeat.config.modules:
  path: \${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s

setup.kibana:
  host: "192.168.38.105:5601"
  username: vagrant
  password: vagrant

setup.dashboards.enabled: true
setup.ilm.enabled: false

output.elasticsearch:
  hosts: ["192.168.38.105:9200"]
EOF

# Aktivieren des osquery-Moduls in Filebeat
cat >/etc/filebeat/modules.d/osquery.yml.disabled <<EOF
- module: osquery
  result:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/var/log/kolide/osquery_result"]
EOF
filebeat --path.config /etc/filebeat modules enable osquery

# Enable und Starten des Filebeat-Dienstes
/bin/systemctl enable filebeat.service
/bin/systemctl start filebeat.service
```

## Auditbeat Setup:
- Die Auditbeat-Konfiguration wird erstellt, wobei Pfade zu Modulen und Verbindungsdaten zu Elasticsearch festgelegt sind. 
- Auditbeat-Module für auditd, file_integrity und system werden aktiviert. 
- Auditbeat wird als systemd-Dienst gestartet und aktiviert.

````bash
# Erstellen der Auditbeat-Konfiguration
cat >/etc/auditbeat/auditbeat.yml <<EOF
auditbeat.config.modules:
  path: \${path.config}/modules.d/*.yml
  reload.period: 10s
  reload.enabled: true
auditbeat.max_start_delay: 10s

auditbeat.modules:
- module: auditd
  audit_rule_files: [ '\${path.config}/audit.rules.d/*.conf' ]
  audit_rules: |
- module: file_integrity
  paths:
  - /bin
  - /usr/bin
  - /sbin
  - /usr/sbin
  - /etc
- module: system
  state.period: 12h
  user.detect_password_changes: true
  login.wtmp_file_pattern: /var/log/wtmp*
  login.btmp_file_pattern: /var/log/btmp*
setup.template.settings:
  index.number_of_shards: 1
setup.kibana:
  host: "192.168.38.105:5601"
  username: vagrant
  password: vagrant

setup.dashboards.enabled: true
setup.ilm.enabled: false

output.elasticsearch:
  hosts: ["192.168.38.105:9200"]
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
EOF

# Aktivieren des auditbeat-Moduls in Filebeat
mv /etc/auditbeat/audit.rules.d/sample-rules.conf.disabled /etc/auditbeat/audit.rules.d/sample-rules.conf

# Enable und Starten des Auditbeat-Dienstes
/bin/systemctl enable auditbeat.service
/bin/systemctl start auditbeat.service
````

## Weitere Konfigurationen:
- Ein Symbolischer Link für Bro-Logs wird erstellt. 
- Die SIEM-Regeln werden in Kibana geladen. 
- Elasticsearch-Trial wird (auskommentiert) aktiviert.

````bash
# Erstellen eines symbolischen Links für Bro-Logs
mkdir /var/log/bro/
ln -s /opt/zeek/logs/current/ /var/log/bro/current

# Warten auf die Verfügbarkeit von Kibana
echo "Waiting for Kibana to be up..."
while true; do
  result=$(curl -uvagrant:vagrant --silent 192.168.38.105:5601/api/status)
  if echo $result | grep -q logger; then break; fi
  sleep 1
done

# Laden der SIEM-Regeln in Kibana
echo "Load SIEM prebuilt rules"
curl -s -uvagrant:vagrant -XPOST "192.168.38.105:5601/api/detection_engine/index" -H 'kbn-xsrf: true' -H 'Content-Type: application/json'
curl -s -uvagrant:vagrant -XPUT "192.168.38.105:5601/api/detection_engine/rules/prepackaged" -H 'kbn-xsrf: true' -H 'Content-Type: application/json'

# Aktivieren des Elasticsearch-Trial (auskommentiert)
# echo "Enable elastic trial version"
# curl -s -XPOST "192.168.38.105:9200/_license/start_trial?acknowledge=true&pretty"

````

Das Skript ist darauf ausgelegt, eine umfassende ELK-Stack-Überwachungsumgebung bereitzustellen, einschliesslich Log-Management, Visualisierung und Sicherheitsinformationen.