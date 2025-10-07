# Cybersecurity SOC Homelab

Een beknopte maar complete handleiding voor het opzetten van een eigen **SOC-homelab** met **VMware Workstation**, Windows (Sysmon + Splunk) en Kali Linux. Ontworpen om veilig te oefenen met loganalyse, detecties en netwerkgedrag zonder risico voor je host of netwerk.

---

## Doelstelling

Het project richt zich op het creëren van een gecontroleerde omgeving voor:

* Het verzamelen en analyseren van logbestanden in **Splunk**.
* Het veilig genereren van **telemetrie** (processen, netwerkverkeer, poortscans).
* Het leren interpreteren van Sysmon- en Splunk-events voor detectie en incidentanalyse.

---

## Vaardigheden

Tijdens dit project ontwikkel je:

* Inzicht in SIEM-principes (logverzameling, indexering, correlatie).
* Vaardigheid in het opzetten van virtuele netwerken en snapshots in **VMware**.
* Ervaring met **Sysmon** en **Windows Event Logging**.
* Basiskennis van detectieregels en SPL-query’s in **Splunk**.

---

## Tools

* **VMware Workstation Pro / Player** – virtualisatieplatform.
* **Windows 10/11 VM** – endpoint met Sysmon en Splunk.
* **Kali Linux VM** – aanvaller/testmachine.
* **Splunk Enterprise/Free** – loganalyse.
* **Splunk Add-on for Sysmon** – automatische veldextracties.

---

## Architectuur

```
[Host PC]
 └── VMware Workstation
      ├── VM1: Windows (Sysmon + Splunk)
      └── VM2: Kali Linux (nmap, benign tests)

Netwerkopties:
- NAT / Bridged → voor beginners met internettoegang.
- LAN Segment → geïsoleerd, veilig voor testscenario’s.
```

<img width="729" height="567" alt="image" src="https://github.com/user-attachments/assets/9f145dd1-5c41-4abb-98d1-483fc3cd3e3a" />


---

## Stappenplan

### 1️⃣ VMware installeren

1. Download **VMware Workstation Pro** of **Player**.
2. Installeer met standaardinstellingen.
3. Controleer of virtualisatie in de BIOS aanstaat.


### 2️⃣ Windows VM aanmaken

* Naam: `WIN-SOC`
* 2–4 vCPU’s, 4–8 GB RAM, 50 GB disk.
* Koppel de Windows ISO via *CD/DVD (SATA)*.
* Installeer **VMware Tools** na installatie voor betere integratie.
  <img width="577" height="795" alt="image" src="https://github.com/user-attachments/assets/103ea3c8-bbc5-42fc-8048-3c88f7a9fd8b" />


### 3️⃣ Kali Linux importeren

* Download de officiële **Kali VMware image (7z)**.
* Pak uit met 7‑Zip en open `.vmx` in VMware.
* Login: `kali` / `kali`.
  <img width="1277" height="840" alt="image" src="https://github.com/user-attachments/assets/99388a8c-cc67-4e16-b153-7226067722a4" />


### 4️⃣ Netwerk instellen

Gebruik een **LAN Segment** of **Host‑Only Network**.

* Windows: `192.168.20.10/24`
* Kali: `192.168.1.2/24`
* Geen gateway of DNS nodig.


Test verbinding:

```cmd
ping 192.168.1.2
```

(werkt alleen als Windows-firewall ICMP toelaat).
*Ref 08 → images/08_ping_test.png*

### 5️⃣ Snapshot maken

Voordat je verder gaat: VMware → **Snapshot → Take Snapshot → Baseline**.
*Ref 09 → images/09_snapshot.png*

### 6️⃣ Splunk + Sysmon op Windows

1. Installeer **Splunk** en start de webinterface (`http://127.0.0.1:8000`).
2. Maak index `endpoint` aan.
3. Installeer **Sysmon** met een gangbare configuratie (bijv. SwiftOnSecurity).
4. Voeg toe aan `inputs.conf`:

   ```ini
   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   index = endpoint
   renderXml = true
   disabled = 0
   ```
5. Herstart Splunk-service en controleer of Sysmon‑logs zichtbaar zijn.
   *Refs 10–12 → images/10_splunk_index.png, 11_inputs_conf.png, 12_sysmon_events.png*

---

## Telemetrie genereren (veilig)

### 🧩 Netwerkscan met nmap (Kali)

```bash
nmap -A -Pn 192.168.20.10
```

Gebruik dit alleen binnen je eigen lab.
*Ref 13 → images/13_nmap_results.png*

### 🧩 Procesactiviteit (Windows)

In CMD of PowerShell:

```
whoami
ipconfig /all
net user
net localgroup administrators
```

Splunk zal Sysmon Event ID 1 (Process Create) registreren.
*Ref 15 → images/15_splunk_process_tree.png*

---

## Splunk-query’s

**Procescreatie:**

```spl
index=endpoint EventCode=1
| stats values(CommandLine) by Image, ParentImage, user
```

**Netwerkactiviteit:**

```spl
index=endpoint | stats count by dest_port, Computer
```

*Ref 14 → images/14_splunk_search_net.png*

---

## Screenshot‑checklist

1. Architectuurdiagram (01_architectuur.png)
2. VMware installatie (02_vmware_setup.png)
3. Windows VM (03_vmware_win_vm.png)
4. Kali import (04_vmware_kali_import.png)
5. LAN Segment (05_network_lansegment.png)
6. Windows IPv4 (06_win_static_ip.png)
7. Kali IPv4 (07_kali_static_ip.png)
8. Ping test (08_ping_test.png)
9. Snapshot (09_snapshot.png)
10. Splunk + Sysmon resultaten (10–15)

---

## Next Steps

* Experimenteer met **Atomic Red Team** of **MITRE CALDERA** (alleen benign tests).
* Breid Splunk uit met **Sigma‑regels** of **Zeek** voor netwerklogging.
* Voeg documentatie of rapportage toe in PDF of Markdown.

---

**Veiligheidstip:**  voer nooit echte malware uit. Alles gebeurt binnen een afgesloten, niet‑verbonden netwerk. Gebruik snapshots om je omgeving snel te herstellen.
