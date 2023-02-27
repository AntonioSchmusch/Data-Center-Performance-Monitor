# Data Center Performance Monitor

Diese Dokumentation hat zum Ziel, Sie beim Monitoring mit Prometheus/Grafana zu unterstützen. Sie entstand im Rahmen eines Moduls zu Projektmanagement/Softwareprojekt in einer Kooperation der HTW Berlin und dem Öko-Institu e.V. und wurde beispielhaft für das Monitoring eines Rechenzentrums geschrieben.

Wir erheben keinen Anspruch auf Vollständigkeit und sind offen für Ergänzungen.


## Grundsätzliche Fragen beantworten

- Sind die nötigen Zugänge zu den Komponenten im Rechenzentrum zugänglich?
- Wie ist das Rechenzentrum strukturiert?
- Was soll überwacht werden?
- Welche Daten sollen gespeichert werden und wie lange?
- Welche Berechnungen sollen ausgeführt werden?

Für bestimmte Berechnungen werden eventuell folgende Daten benötigt:
- Gesamtstromverbrauch des Rechenzentrums oder der Server
- die Elektrische Leistungsaufnahme von: Server, Netzwerk und Speicher


## Download der Tools

- [ ] [Prometheus Download](https://prometheus.io/download/)
- [ ] [Node_Exporter Download](https://github.com/prometheus/node_exporter/releases)
- [ ] [Grafana Download](https://grafana.com/grafana/download)


## Installation der Tools, wo genau installiert man Node, Prom, Graf?

Das Monitoring wird auf einer VM eingerichtet, von welcher aus die einzelnen Targets über das Netzwerk erreichbar sind. Targets können hierbei einzelne Server, aber auch z.B. Strommessgeräte sein. Auf den Servern ist die Installation von Node-Exporter nötig, um die Daten zu erheben.

Auf der Monitoring-VM werden die Tools Prometheus, Grafana und eventuell ein MQTT-Broker installiert. Prometheus greift dabei die einzelnen Daten der Targets ab und speichert diese. Der Stromverbrauch lässt sich von den einzelnen Messgeräten über einen MQTT-Broker erfassen und in eine Datenbank speichern.

Installation Node_Expoerter:
Download:
```
wget \https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
```
Node-Exporter User und Verzeichnisse erstellen:
```
sudo groupadd -f node_exporter
sudo useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
sudo mkdir /etc/node_exporter
sudo chown node_exporter:node_exporter /etc/node_exporter
```
Entpacken:
```
tar -xvf node_exporter-1.0.1.linux-amd64.tar.gz
mv node_exporter-1.0.1.linux-amd64 node_exporter-files
```
Installation:
```
sudo cp node_exporter-files/node_exporter /usr/bin/
sudo chown node_exporter:node_exporter /usr/bin/node_exporter
```
Setup Node-Expoerter:
```
sudo nano /usr/lib/systemd/system/node_exporter.service
```
Beispielkonfiguration:
```
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/bin/node_exporter \
  --web.listen-address=:9200

[Install]
WantedBy=multi-user.target
```
```
sudo chmod 664 /usr/lib/systemd/system/node_exporter.service
```
System neu starten und Node-Exporter starten:
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```
Status überprüfen:

```
sudo systemctl status node_exporter
```
Node-Exporter mit Systemstart starten:
```
sudo systemctl enable node_exporter.service
```
Aufräumen:
```
rm -rf node_exporter-1.0.1.linux-amd64.tar.gz node_exporter-files
```
## Node_Exporter Metriken finden und auswählen

Ist Node_Exporter einmal installiert, so liefert es permanent Messdaten seines Hosts. Über die Konsole (Bash, Powershell, Command-Line) eines Servers auf dem Prometheus installiert und mit Node_Exporter verbunden ist, lässt sich mit dem Befehl

``` curl http://localhost:9100/metrics ```

eine Liste ausgeben, die alle Metriken enthält, die von Node_Exporter angeboten werden. # HELP erläutert kurz die Metrik, nach #TYPE folgt der Name der Metrik sowie der Datentyp (z.B. counter). Dananch folgen die Metriken und aktuell ermittelte Werte, da Prometheus permanent misst. Bei einigen Metriken werden auch verschiedene Modi angezeigt, bspw. die verschiedenen Modi, in denen sich ein CPU befinden kann. 

Auszug aus der Liste der Metriken:

```
    # HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
    # TYPE node_cpu_seconds_total counter
    node_cpu_seconds_total{cpu="0",mode="idle"} 4.19954657e+06
    node_cpu_seconds_total{cpu="0",mode="iowait"} 1306.24
    node_cpu_seconds_total{cpu="0",mode="irq"} 0
    node_cpu_seconds_total{cpu="0",mode="nice"} 3.06237605e+06
    node_cpu_seconds_total{cpu="0",mode="softirq"} 126.9
    node_cpu_seconds_total{cpu="0",mode="steal"} 0
    node_cpu_seconds_total{cpu="0",mode="system"} 19429.74
    node_cpu_seconds_total{cpu="0",mode="user"} 208544.95
    # HELP node_filesystem_free_bytes Filesystem free space in bytes.
    # TYPE node_filesystem_free_bytes gauge
    node_filesystem_free_bytes{device="/dev/mapper/ubuntu--vg-ubuntu--lv",fstype="ext4",mountpoint="/"} 9.0620461056e+10
    node_filesystem_free_bytes{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"} 1.119490048e+09
    # HELP node_network_transmit_bytes_total Network device statistic transmit_bytes.
    # TYPE node_network_transmit_bytes_total counter
    node_network_transmit_bytes_total{device="eno1"} 4.665219706e+09
    node_network_transmit_bytes_total{device="eno2"} 4.0108397e+07
```

## Konfiguration

Prometheus ist sehr gut dokumentiert. Die meisten Fragen zu Konfiguration, Installation werden hier beantwortet.
- [ ] [Prometheus documentation](https://prometheus.io/docs/prometheus/latest/getting_started/)


## Festlegung der Monitoring-Dauer (Aufbewahrung)

(Alphalpha bitte ? wo in der Config Datei hatten wir die 500 Tage eingestellt?)


## Beispiel für prometheus.yml-Datei

Die prometheus.yml ist standardmäßig vorhanden und kann konfiguriert werden. Hier wird definiert, welche "targets" gescraped werden sollen und in welchem Abstand. Unter "rules" ist ein Verweis auf Berechnungsvorschriften, falls vorhanden, hinterlegt. Außerdem können Metriken benannt werden, die behalten (keep) oder nicht behalten (drop) werden sollen um die Datensparsamkeit zu fördern.

Unter "files" ist eine vollständige prometheus.yml-Datei des Uni-Projekts vorhanden. Nachfolgend ein Auszug aus prometheus.yml mit 60 Sekunden Scrape Intervall und dem Verweis auf die rules-Datei "first_rules.yml".

```
# Sample config for Prometheus.

global:
  scrape_interval:     60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 60s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "first_rules.yml"
```


## Beispiel für Berechnungsvorschriften

Dies ist ein allgemeines Beispiel für Berechnungsvorschriften, also Formeln, die in einer rules-Datei hinterlegt werden, die von Prometheus aufgerufen und ausgeführt wird. Die unter "files" angehängte Datei ist sehr ähnlich, enthält jedoch Namen, die speziell für das Uni-Projekt ausgewählt wurden.

Im Beispiel wird berechnet:

- CPU Auslastung in %
- Lese-Schreibvolumen im Speicher sowie Speicherauslastung
- Übertragene Datenmenge im Netzwerk

Es bietet sich an die "records" so zu benennen, dass ersichtlich ist, welche Berechnung dahinter steckt.

first_rules.yml

```
groups:
  - name: Server
    rules:
    - record: cpu_auslastung_in_prozent_sum_by_instance
      expr: ((sum by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[1h]))) / (sum by (instance) (rate(node_cpu_seconds_total[1h]))))*100 
  - name: Speichersysteme
    rules:
    - record: lese_schreib_volumen_gigabyte_average_by_instance
      expr: avg by (instance) (rate(node_disk_read_bytes_total[1h]) + rate(node_disk_written_bytes_total[1h])) * 9.31e-10
    - record: speicherauslastung_terabyte_sum_by_instance
      expr: sum by (instance) (node_filesystem_size_bytes - min_over_time(node_filesystem_free_bytes[1d])) * 1E-12
  - name: Netzwerk
    rules:
    - record: uebertragene_datenmenge_gigabyte_sum_by_instance
      expr: sum by (instance) (rate(node_network_transmit_bytes_total[1h]) + rate(node_network_receive_bytes_total[1h])) * 9.31e-10
```

## Anmerkungen

* 9.31e-10 = Ausgabe des Ergebnisses in Gigabyte
* 1E-12 = Umrechnung des Ergebnisses in Terabyte






