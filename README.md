# Data Center Performance Monitor

Diese Dokumentation hat zum Ziel, Sie beim Monitoring mit Prometheus/Grafana zu unterstützen. Sie entstand im Rahmen eines Moduls zu Projektmanagement/Softwareprojekt in einer Kooperation der HTW Berlin und dem Öko-Institu e.V. und wurde beispielhaft für das Monitoring eines Rechenzentrums geschrieben.

Wir erheben keinen Anspruch auf Vollständigkeit und sind offen für Ergänzungen.


## Grundsätzliche Fragen beantworten

- Haben Sie die nötigen Zugänge zu den Komponenten im Rechenzentrum?
- Wie ist Ihr Rechenzentrum strukturiert?
- Was soll überwacht werden?
- Welche Daten sollen gespeichert werden und wie lange?
- Welche Berechnungen sollen ausgeführt werden?

Für bestimmte Berechnungen werden Sie eventuell folgende Daten benötigen:
- Gesamtstromverbrauch des Rechenzentrums oder der Server
- die Elektrische Leistungsaufnahme von: Server, Netzwerk und Speicher


## Download der Tools

- [ ] [Prometheus Download](https://prometheus.io/download/)
- [ ] [Node_Exporter Download](https://github.com/prometheus/node_exporter/releases)
- [ ] [Grafana Download](https://grafana.com/grafana/download)


## Installation der Tools, wo genau installiert man Node, Prom, Graf?

(Antonio bitte)

```
irgendwelche coolen Konsolenbefehle vielleicht?
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






