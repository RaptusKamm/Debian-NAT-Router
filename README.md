# Debian-NAT-Router

In diesem Post wird erklärt wie man einen Debian-Server als NAT-Router konfiguriert.

 

Benötigte Maschinen:

Debian-Maschine mit 2 Netzwerkadaptern (Netzwerkbrücke, intern) (Hier wird die Maschine als Master bezeichnet)
Debian-Maschine mit 1 Netzwerkadapter (Intern) (Hier wird die Maschine als Node bezeichnet)
 

Vorbereitung

Zuerst muss ein neues Netz innerhalb des Masters erstellt werden. Hierfür bearbeitet man folgende Datei: interfaces

```nano /etc/network/interfaces```
 

Unter enp0s3 fügt man folgenden Eintrag ein:

```
auto enp0s8
iface enp0s8 inet static
		address 192.168.5.1
		netmask 255.255.255.0
```
(Die 5 kann hier beliebig geändert werden wie z.B. 10 = 192.168.10.0)

Somit hat man nun ein extra Netz für die Nodes erstellt. Jetzt muss nur noch der Service "networking" neugestartet werden.
```
service networking restart
```

Damit der Traffic an die Nodes gehen kann muss noch ein Wert geändert werden. Dafür bearbeitet man die Datei: sysctl.conf
```
nano /etc/sysctl.conf
```

Dort entfernen wir ein # von folgender Zeile:
```
Vorher:
#net.ipv4.ip_forward = 1

Nachher:
net.ipv4.ip_forward = 1
```

Somit wird der Traffic nun über den Router weitergeleitet. Damit das ganze nun greift, wird der Master einmal neugestartet.

reboot
 

Arbeiten mit iptables

Damit die Nodes über den Router ins WAN kommen, muss man nun einige NAT-Regeln in die iptables eintragen. Falls keine iptables vorhanden sind muss man diese vorher installieren:
```
apt-get install iptables
 ```

Folgende Regeln müssen eingetragen werden:
```
iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE
```
```
iptables --append FORWARD --in-interface enp0s8 -j ACCEPT
```
Somit wurden erfolgreich die Regeln eingetragen.

 

Zum Schluss kann nun nach belieben die IP-Table Konfiguration gespeichert werden.

Dies funktioniert indem das Paket "iptables-persistent" mit:
```
apt-get install -y iptables-persistent
```
installiet wird.

 

In der Installation wird beretis gefragt, ob die IP Konfguration von IPv4 und IPv6 gespeichert werden soll, was entweder mit ja oder nein beantworten kann.

 

Konfiguration der Nodes

Die Nodes müssen noch auf das erstellte Netz vom Master eingestellt werden. Dafür bearbeitet man die Datei: interfaces
```
nano /etc/network/interfaces
```

Hier muss man den bereits vorhanden Eintrag von enp0s3 bearbeiten:
```
auto enp0s3
iface enp0s3 inet static
		address 192.168.5.10
		gateway 192.168.5.1
		netmask 255.255.255.0
```

Nachdem bearbeiten des Interfaces muss nun der Service "networking" neugestartet werden.
```
service networking restart
```

Nun ist der Node über die interne Schnittstelle mit dem Router verbunden und hat Zugriff auf das Internet über das Netzwerk 192.168.5.0.

Somit hat man nun einen Debian-Server als NAT-Router konfiguriert.

 

Optional kann nun eine Netzwerkroute unter Windows angelegt werden, um auch die Node, beispielsweise, mit SSH erreichen zu können.

Hierfür wird der Folgende Befehl benötigt:
```
route add <Netz-ID> MASK <Netzmaske> <Gateway>
```

Beispiel:
```
route add 192.168.10.0 MASK 255.255.255.0 192.168.10.1
```

Nun ist auch die Node erreichebar.
