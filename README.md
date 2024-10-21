# Forschungsprojekt - OWELK (Opnsense, Wazuh, Elasticsearch, Logstash, Kibana)
Dies ist das offizielle Repository für das Forschungsprojekt im Rahmen des Masterstudiengangs. Dieses Projekt beschäftigt sich mit der Implementierung eines ganzheitlichen Netzwerküberwachungs-Tools unter Verwendung verschiedener Sicherheitstechnologien, damit Kleinstunternehmen ihr Netzwerk mit möglichst wenig Aufwand und geringen Kosten überwachen können. Die verwendeten Sicherheitstechnologien sind: 

| Tool | Beschreibung |
| ----------- | ----------- |
| [Opnsense](https://opnsense.org/)      | Eine Open-Source-Firewall- und Routing-Plattform, die Sicherheitsfunktionen wie VPN, IDS/IPS und Webfilter bietet. |
| [Wazuh](https://wazuh.com/)         | Eine Open-Source-Plattform für Sicherheitserkennung, -überwachung und -reaktion, die Log-Analyse, Bedrohungserkennung und Integritätsprüfung bietet. |
| [Elasticsearch](https://www.elastic.co/de/elasticsearch) | Eine verteilte Such- und Analyse-Engine zur Speicherung und schnellen Abfrage großer Datenmengen.     |
| [Logstash](https://www.elastic.co/de/logstash)      | Ein Datenverarbeitungs-Pipeline-Tool, das Logs und andere Datenquellen sammelt, verarbeitet und an verschiedene Ausgaben sendet, oft an Elasticsearch. |
| [Kibana](https://www.elastic.co/de/kibana)        | Ein Visualisierungstool für Elasticsearch, das Dashboards erstellt und Daten in Echtzeit visualisiert. | 

---

## Referenzimplementierung 
Dieses Repository enthält eine Referenzimplementierung von OWELK. Durch das Befolgen der Schritte in der Datei `Installation.md` kann das System in einem Netzwerk implementiert werden.

![Elasticsearch Agents](https://private-user-images.githubusercontent.com/175680852/378340208-c77c3261-fcc8-4947-8a00-e19c414075dd.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3Mjk1MDAyNjQsIm5iZiI6MTcyOTQ5OTk2NCwicGF0aCI6Ii8xNzU2ODA4NTIvMzc4MzQwMjA4LWM3N2MzMjYxLWZjYzgtNDk0Ny04YTAwLWUxOWM0MTQwNzVkZC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQxMDIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MTAyMVQwODM5MjRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mNDY0NTNlZDJlMGQ2NzkyNmQ0OWJjMGU5Mzc2NDBkOTQ4ZDA4YjI4MWYyMDA4MmEwMWExNzNlZDAwODg5NWFmJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.Jb41i1a3mG-ccfNecUGEPv-30AhOQYilq3iR5oIUfH0)

![Wazuh Logs](https://private-user-images.githubusercontent.com/175680852/378340234-b5f20b9f-2ef7-4a79-a7d2-eca9112201c0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3Mjk1MDAyNjQsIm5iZiI6MTcyOTQ5OTk2NCwicGF0aCI6Ii8xNzU2ODA4NTIvMzc4MzQwMjM0LWI1ZjIwYjlmLTJlZjctNGE3OS1hN2QyLWVjYTkxMTIyMDFjMC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQxMDIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MTAyMVQwODM5MjRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0xNDIxOWE1ZjJkZjdmNzhjMGQ0OWIzYjBkODQzNzcxYmIzMTEzNzNkZTQ5OWQ2MjNkMjliNjM3ODlmZmNmYTc1JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.sxVxoiwrpL2v3trrh1oFOwCxPMzIo3dMs5u5PE5Vy_Q)
