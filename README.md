# Task 1: Scan Local Network for Open Ports
*Internship:* Elevate Labs - Cyber Security Internship
*Tools:* Nmap 7.99 (Zenmap GUI), Wireshark 4.6.8
*Date:* 05-11-2026

## Objective
Discover open ports on a device in my local network, identify the services behind them, and understand the security risks of network exposure.

## What I Did
1. Installed Nmap (Zenmap) and Wireshark on my Windows laptop.
2. Found my IP address using ipconfig (10.131.10.190).
3. Ran a TCP SYN scan: nmap -sS 10.131.10.190
4. Ran service/version detection: nmap -sV 10.131.10.190
5. Captured the scan traffic in Wireshark (loopback adapter, because I scanned my own machine).
6. Researched the services found and listed their risks.
7. Saved the results as text and screenshots.

> Only my own device was scanned. No third-party systems were touched.

## Scan Results

| IP Address | Port | State | Service | Version |
|---|---|---|---|---|
| 10.131.10.190 | 135/tcp | open | msrpc | Microsoft Windows RPC |
| 10.131.10.190 | 139/tcp | open | netbios-ssn | Microsoft Windows netbios-ssn |
| 10.131.10.190 | 445/tcp | open | microsoft-ds | SMB (version not identified) |

997 other ports were closed (reset). OS detected: Windows.

## Wireshark Analysis
Filter used: tcp.port == 445

| Packet | Info | Meaning |
|---|---|---|
| 51 | 59118 -> 445 [SYN] | Nmap sends a SYN probe |
| 52 | 445 -> 59118 [SYN, ACK] | Port replies, so it is *open* |
| 53 | 59118 -> 445 [RST] | Nmap resets instead of completing the handshake |

Nmap never sends the final ACK, so the connection is not completed. This is why a SYN scan is called a half-open (stealth) scan. A closed port (for example port 199) replies with RST, ACK directly instead of SYN-ACK.

Total packets captured: 2003 (about 1000 ports x probe + reply).

## Services and Security Risks

| Port | Service | Risk | Mitigation |
|---|---|---|---|
| 135 | MS RPC | Attackers can enumerate Windows services; historic DCOM/RPC exploits | Block at firewall, do not expose outside the host |
| 139 | NetBIOS | Leaks computer name and workgroup, useful for reconnaissance | Disable NetBIOS over TCP/IP if not needed |
| 445 | SMB | Most dangerous: exploited by WannaCry/EternalBlue, ransomware and lateral movement, brute force on weak passwords | Patch Windows, disable SMBv1, turn off file sharing, set network profile to Public, restrict with firewall |

## Interview Questions

1. *What is an open port?*
   A port where a service is listening and accepting connections.
2. *How does Nmap perform a TCP SYN scan?*
   It sends a SYN packet. SYN-ACK means open, RST means closed. It then sends RST instead of ACK, so the handshake is never completed.
3. *What risks are associated with open ports?*
   Unpatched services can be exploited, weak authentication can be brute-forced, and exposed services leak information.
4. *Difference between TCP and UDP scanning?*
   TCP scans rely on the handshake and give clear replies. UDP has no handshake, replies are rare, so scans are slower and less certain (open|filtered).
5. *How can open ports be secured?*
   Close unused services, apply patches, use strong authentication, and restrict access with a firewall.
6. *What is a firewall's role regarding ports?*
   It allows or blocks traffic to ports based on rules (IP, port, protocol, direction).
7. *What is a port scan and why do attackers perform it?*
   Probing a host to find open ports. Attackers use it for reconnaissance to find vulnerable services to target.
8. *How does Wireshark complement port scanning?*
   Nmap shows which ports are open; Wireshark shows the actual packets (SYN, SYN-ACK, RST) so the scan behaviour can be verified and understood.

## Repository Contents
- scan_results.txt - Nmap output
- nmap_svan.pcapng - Wireshark capture
- screenshots/ - Zenmap output, Wireshark filters (ports 445, 139, 199)
- README.md - this file

## Conclusion
The scan showed three open Windows ports (135, 139, 445), all tied to file/RPC sharing. SMB (445) is the biggest concern. Closing unused services and enabling the firewall greatly reduces the attack surface.
