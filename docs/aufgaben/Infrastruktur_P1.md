# 1. LB2 - Infrastruktur aufbauen/verstehen und dokumentieren (10%)

## 1.1 Bewertungskriterien in der Dokumentation#

Infrastruktur-Lab 0-4P:

 - Netzwerk-Diagramm mit IP-Adressen der VMs [Netzwerk Diagramm](../tech_daten/Netzwerk_Diagramm_P1.md)
 - Beschreibung der beiden Windows-VMs: [Windows VM's](../tech_daten/WindowsVMs.md)
     - Login
     - Installierte Dienste ("Welche Software wurde auf der VM installiert?")
     - Zugriffe auf die Applikationen ("Wie greife ich auf die verschiedenen Softwaren zu?")
 - Beschreibung der ELK-VM [Linux VM](../tech_daten/LinuxVM.md)
     - Login
     - Installierte Software ("Welche Software wurde auf der VM installiert?")
     - Zugriffe auf die Applikationen ("Wie greife ich auf die verschiedenen Softwaren zu?")

Infrastruktur-Setup 0-4P:

- Sinn und Zweck des Vagrantfile [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)
- Sinn und Zweck Provisioning-Befehle [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)
- Liste aller Scripts für die Windows-Images die bei einer Installation ausgeführt werden [Skript Liste](../tech_daten/skripts_P1.md)
- Beschreibung des bootstrap.sh-Files für das Aufsetzen der Logger-VM [Bootstrap Skript](../tech_daten/bootstrap_P1.md)
- Beschreibung des ELK.sh-Files für das Aufsetzen der Logger-VM [ELK Skript](../tech_daten/elk_P1.md)

## 1.2 Vorbereitungen#

Die Lehrpersonen verteilt Ihnen 3(!) VMs. Damit diese VMs miteinander kommunizieren können, müssen bei den VMs die Netzwerk-Adapter etwas speziell konfiguriert werden.

Wir verwenden eine etwas abgewandelte Version von diesem Repository - die (angepasste) Installation finden Sie hier

Interface 1:
Besitzt einen Standardgateway und ist über NAT angeschlossen
Interface 2:
Ist ein Host-Only Network und besitzt keinen Gateway

## 1.3 Aufträge#

Schritt 1 : Installation der Boxen

- Kopieren Sie die VMs auf Ihren Laptop
- Starten Sie die VMs (je nach Laptop: nacheinander)
- Kontrollieren Sie Netzwerk-Konnektivität

Schritt 2 : Recherche (Vagrant / Vagrantfile)

 - Informieren Sie sich zum Produkt Vagrant und Konfigurationsdatei Vagrantfile [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)
 - Lesen und interpretieren Sie das Vagrantfile für das Detection-LAB, ELK-Version [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)

**Beispielfragen die Sie danach verstehen sollten: **

 - Wie kommt es das ELK auf der Logger-VM installiert wird? [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)
 - Wie kommt es das auf dem DC eine Domäne installiert wird? [Vagrant Dokumentation](../tech_daten/Vagrant_P1.md)
 ...

Schritt 3 : Beschreibung der Infrastruktur

Beschreiben und dokumentieren Sie, nach den gegebenen Kriterien, die resultierende Umgebung.
