# Aufbau der Testumbegung
Suricata hat, je nach Anwendungsfall und Einsatzzweck unterschiedlichste  
Anforderungen bezüglich der Rechenleistung, des Arbeitsspeichers und des  
Speicherplatzes. Da wir in unserem PoC kein Live-Monitoring aller Datenpakete  
inklusive Speicherung vorsehen, greifen wir auf Entwicklerlaptops zurück, die  
mit 32 GB Arbeitsspeicher und einem Core i7 Prozessor ausgestattet sind.  
  
![](suricata_testaufbau.png) 
  
  
Letztendlich haben wir unsere finale PoC Installation auf einem ubuntu 22.04 LTS  
server durchgeführt. Nach dem Aufsetzten des 22.04 ubuntu servers unter  
Virtual Box erfolgte die Einbindung der Personal Package Archives (PPAs):  
  
## Installation von Suricata  

```sudo add-apt-repository ppa:oisf/suricata-stable –yes```

Aktualisierung des repositories mit: 

```sudo apt update```

Installation von suricata mit dem Paketmanager 

```apt-cache policy suricata```  
```sudo apt install suricata jq```  
  
Test der erfolgreichen Installation über die Ausgabe der Verstionsnummer. 

```sudo suricata --build-info```

Die Suricata Regeln sind unter:  

```/etc/suricata/rules``` abgelegt.  

Die Konfigurationsdatei von Suricata liegt unter: 

```/etc/suricata/suricata.yaml```

Für die initiale Konfiguration ist der Quickstart guide sehr empfehlenswert.  
(Quelle: <https://suricata.readthedocs.io/en/latest/quickstart.html>)  

Nach Abschluss der Konfiguration kann diese mit folgendem Befehl syntaktisch überprüft werden:

```sudo suricata -T -c /etc/suricata/suricata.yaml –v```

Wenn die Konfigurationsprüfung ohne Fehlermeldung zurückkehrt,  
kann suricata mit folgendem Befehl gestartet werden: 

```sudo systemctl enable --now suricata```

Der aktuelle Status lässt sich anschließend mit:  

```sudo systemctl status suricata``` ermitteln.  

![](suricata_systemctl_status.png)  


Als Test-Case soll ein SYN-flood mit Hping3 erzeugt werden, welche idealerweise  
von suricata erkannt wird. Zur Vorbereitung des Tests wird die Packet-Offload-Funktionen
der Netzwerkschnittstelle, auf der Suricata lauscht mit: 

```ethtool -K enp0s3 gro off lro off``` deaktiviert  

  
Suricata wird jetzt im PCAP Modus gestartet und monitort den Netzwerkcerkehr der Schnittstelle: enp0s3  
Die SYN-flood Attacke wird gestartet mit: 

```hping3 -S -p 80 --flood --rand-source 192.178.162.15 -I enp0s3 -c 150```
  
Suricata schreibt alle Ausgaben der DDoS-Attacke in das Logfile: ```/var/log/suricata/fast.log```
  
Im Logfile finden sich die folgenden Warnungen:  
```
05/28/2022 … ET DROP Spamhaus DROP Listed Traffic Inbound group 26 ...  
[Classification: Misc Attack] [Priority: 2] {TCP} 192.178.162.16:17846 -> 192.178.162.15:80
05/28/2022 … ET DROP Spamhaus DROP Listed Traffic Inbound group 16 ...  
[Classification: Misc Attack] [Priority: 2] {TCP} 192.178.162.17:17899 -> 192.178.162.15:80  
05/28/2022 … ET DROP Spamhaus DROP Listed Traffic Inbound group 2 ... 
[Classification: Misc Attack] [Priority: 2] {TCP} 192.178.162.18:20024 -> 192.178.162.15:80
```

  

![](biberbande_testui.png)
