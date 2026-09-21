# Apex Planet Internship - Task 2: Network Security & Scanning

This folder contains the evidence and documentation for Apex Planet Internship Task 2. The practical combined private-lab network setup, reconnaissance, Nmap enumeration, Nessus vulnerability assessment, Wireshark traffic analysis, controlled SYN analysis, and host firewall testing.

## Scope

### Controlled target - Metasploitable2
- IP: `192.168.56.101`
- Network: VirtualBox Host-Only
- Used for: active reconnaissance, Nmap, Nessus, Wireshark, hping3, and iptables

### Public training target - VulnWeb
- Hostname: `testaspnet.vulnweb.com`
- Observed IP: `44.238.29.244`
- Used for: WHOIS, Nslookup, Google Dorking, Shodan, reachability, banner grabbing, and Nmap

## Why two targets were used

Metasploitable2 was isolated on a private Host-Only network, so public passive-recon sources such as WHOIS, public search indexing, and Shodan were not representative for the private IP. VulnWeb therefore supplied the public Internet-facing target needed for passive reconnaissance, while Metasploitable2 supplied a controlled target for active and defensive exercises.

## Repository structure

```text
Task-2-Network-Security/
├── README.md
├── nmap/
│   ├── metasploitable-tcp.txt
│   ├── metasploitable-udp.txt
│   ├── metasploitable-services.txt
│   ├── metasploitable-os.txt
│   ├── vulnweb-tcp.txt
│   ├── vulnweb-udp.txt
│   ├── vulnweb-services.txt
│   └── vulnweb-os.txt
├── wireshark/
│   ├── HTTP evidence
│   ├── FTP evidence
│   ├── DNS evidence
│   └── PCAP/PCAPNG evidence
├── report/
│   └── Apex_Task2_Network_Security_Detailed_Report.pdf
└── screenshots/
    ├── Nessus
    ├── Nmap
    ├── Recon
    ├── hping3
    └── iptables
```

> The final five-minute demo video may be submitted separately through the internship portal or linked here after upload.

## Practical workflow

1. Restore the VirtualBox Host-Only network.
2. Assign and verify Kali `192.168.56.102/24`.
3. Assign and verify Metasploitable2 `192.168.56.101/24`.
4. Verify connectivity with ping and a private-subnet ping sweep.
5. Perform passive reconnaissance against VulnWeb.
6. Perform active reconnaissance and banner grabbing.
7. Run Nmap `-sS`, `-sU`, `-sV`, and `-O` against both targets.
8. Preserve raw Nmap results.
9. Install/start Nessus and assess Metasploitable2.
10. Capture HTTP, FTP, DNS, and SYN traffic in Wireshark.
11. Demonstrate plaintext FTP authentication exposure in the lab.
12. Apply iptables allow/deny rules and verify the effect with Nmap.
13. Package evidence into the report and five-minute demonstration.

## Key results

### Metasploitable2
- Private-lab address: `192.168.56.101`
- 23 open TCP services were observed in the default 1,000-port scan.
- Service detection identified, among others, vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, BIND 9.4.2, Samba, MySQL 5.0.51a, PostgreSQL 8.3.x, VNC, UnrealIRCd, AJP, and Tomcat.
- OS fingerprinting reported Linux 2.6.x (details 2.6.9-2.6.33).
- Telnet on TCP/23 accepted the supplied lab account and returned a remote Linux shell. This is documented as authorized service authentication with known credentials, not an authentication bypass exploit.
- Nessus Essentials reported 59 vulnerabilities on the completed Metasploitable2 scan dashboard.

### VulnWeb
- `testaspnet.vulnweb.com` resolved to `44.238.29.244`.
- WHOIS identified Gandi SAS as registrar and AWS DNS name servers for the parent domain.
- The IP WHOIS result showed Amazon.com / Amazon Web Services allocation information.
- HTTP banner evidence identified Microsoft IIS 8.5 and `X-Powered-By: ASP.NET`.
- Nmap identified TCP/80 as open; other common TCP ports appeared filtered.
- UDP scanning reported the 1,000 common ports as `open|filtered` due to no response.
- OS detection was explicitly inconclusive; its guesses are not treated as confirmed OS facts.

## Nessus severity note

Representative findings reported from the completed dashboard were:

| Severity | Finding | CVSS |
|---|---|---:|
| Critical | VNC Server Password Vulnerability | 10.0 |
| Critical | Canonical Ubuntu Linux issues | 10.0 |
| Critical | SSL Version 2/3 and Bind Shell Backdoor | 9.8 |
| High | NFS Shares World Readable | 7.5 |
| High | rlogin Service Detection | 7.5 |
| Medium | TLS Version 1.0 Protocol | 6.5 |
| Low | Exact finding/count not transcribed in retained session log | - |

The Nessus UI presented an upgrade requirement when export was attempted, so screenshots of the completed scan/results are the primary evidence.

## Wireshark evidence

### HTTP
Generated with:
```bash
curl http://192.168.56.101
```

### FTP
Generated with:
```bash
ftp 192.168.56.101
```

The lab `msfadmin` account authenticated successfully. FTP authentication packets were filtered with:
```text
ftp.request.command == "USER" || ftp.request.command == "PASS"
```

Do not publish credential-bearing screenshots or PCAP/PCAPNG files publicly without redaction/approval.

### DNS
```bash
dig @192.168.56.101 metasploitable.localdomain
dig @192.168.56.101 localhost A
```

The first query returned `SERVFAIL`; the second returned `NOERROR` with `localhost -> 127.0.0.1`.

## SYN analysis

```bash
hping3 --version
sudo hping3 -S -p 80 -c 20 -i u100000 192.168.56.101
```

The bounded test sent 20 SYN probes and received 20 SYN-ACK responses with 0% packet loss. This demonstrates SYN-handshake behavior; it is not claimed as proof of a successful denial-of-service condition.

## Firewall demonstration

Baseline:
```bash
sudo nmap -p 80 192.168.56.101
```
Result: `80/tcp open http`

Deny:
```bash
iptables -I INPUT -p tcp --dport 80 -j DROP
sudo nmap -p 80 192.168.56.101
```
Result: `80/tcp filtered http`

Allow SSH:
```bash
iptables -I INPUT -p tcp --dport 22 -j ACCEPT
sudo nmap -p 22,80 192.168.56.101
```
Result: `22/tcp open ssh`, `80/tcp filtered http`.

## Important troubleshooting lessons

- `ip addr replace` assigns an address but does not create a persistent NetworkManager profile; the later `nmcli connection add ...` created the persistent lab profile.
- `nmap 44.238.29.244 80` scans two targets. The correct port-specific form is `nmap -p 80 44.238.29.244`.
- Netcat against Telnet may show protocol-negotiation bytes; the Telnet client makes the banner readable.
- A DNS `SERVFAIL` still demonstrates DNS request/response traffic; a later `NOERROR` query provided the successful case for the report.
- `open|filtered` in UDP results is not equivalent to “open”.
- The missing `nessusd.service` unit did not prevent Nessus from running; the installed Nessus service binary successfully started the daemon and exposed TCP/8834.
- The `zsh: corrupt history file` warning was unrelated to the network-security tests.

## Final deliverables

- Detailed PDF report in `report/`
- Eight raw Nmap result files in `nmap/`
- Wireshark PNG/PCAP evidence in `wireshark/`
- Nessus and other screenshots in `screenshots/`
- Five-minute demonstration video submitted separately or linked here

## References

- Nmap: https://nmap.org/book/
- Nmap Port Scanning Basics: https://nmap.org/book/man-port-scanning-basics.html
- Nmap Port Scanning Techniques: https://nmap.org/book/man-port-scanning-techniques.html
- Tenable Nessus 10.12: https://docs.tenable.com/nessus/Content/GettingStarted.htm
- Tenable Nessus Severity: https://docs.tenable.com/nessus/10_12/Content/Severity.htm
- Packt Active Reconnaissance: https://subscription.packtpub.com/book/security/9781837630639/9/ch09lvl1sec51/active-reconnaissance
