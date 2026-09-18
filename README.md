# networkwalks-w2-B083--footprinting-scanning--

PENETRATION TESTING REPORT
FOOTPRINTING & NETWORK SCANNING PHASES
W2-PM-FINAL  |  CYBERSECURITY  |  NETWORKWALKS

Week 2 pentest report — footprinting networkwalks.com and Zenmap network scanning

|||
|---|---|
| **Pentester Name** | Ademola Oduola (Cybersecurity Professional) |
| **Program / Batch** | B083 - Networkwalks |
| **Date** | 18 September 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools)<br>W2-PM5 (Zenmap Scanning) |
| **Client / Target** | 1. Networkwalks (secured written permission already)<br>2. My own local LAN network |
| **Permission secured from client?** | Yes |
| **Phases covered** | Phase 1: Reconnaissance & Footprinting<br>Phase 2: Scanning & Network Discovery<br>Phase 3-5: In Progress |

1. Liability Disclaimer:
I have performed these activities only on the systems and devices where I had secured written permission, or on devices and systems that I own myself. All of this material is for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job, and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

2. Introduction:
This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning my own local network with Zenmap (W2-PM5). One module covers the footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering publicly available information to mapping the live hosts on a network. This is the Week 2 part of my ongoing Cybersecurity & Ethical Hacking internship at Networkwalks.

The footprinting commands were run in Kali Linux 2026.3 inside Oracle VirtualBox. The scanning activity was performed on a Windows 10 PC with Nmap 7.991 and Zenmap installed. Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker’s point of view.

3. Tools Used: 

The table below lists each tool used in this report and its purpose.

| Tool | Purpose |
|---|---|
| Kali Linux & Windows 10 | Operating systems used for the reconnaissance and scanning activities. |
| WHOIS | Find domain registration details (registrar, dates, name servers, status codes). |
| WhatWeb | Fingerprint web technologies (server, CMS, plugins, IP address). |
| nslookup | Resolve the domain name to its IP address using DNS. |
| curl -I | Read the HTTP response headers of the website. |
| wafw00f | Detect whether a Web Application Firewall protects the site. |
| dnsrecon | Enumerate all DNS records (SOA, NS, MX, A, SPF, TXT, SRV). |
| Zenmap (Nmap GUI) | Scan the local subnet to find live hosts, IP and MAC addresses. |
| Windows CMD (ipconfig) | Identify the local IP address and LAN subnet. |

4. Activities Performed:
   
4.1  Footprinting & Reconnaissance (W2-PM1)
   
I performed reconnaissance against the networkwalks.com domain using six Kali Linux tools: WHOIS, WhatWeb, nslookup, curl, wafw00f and dnsrecon. Each tool was used to collect a different type of information about the target.

Task 1 – WHOIS
Command: whois networkwalks.com
WHOIS returned the publicly available registration record for the domain. The domain is registered through GoDaddy.com, LLC (IANA ID 146), was created on 6 November 2019, was last updated on 12 November 2025, and is set to expire on 6 November 2027. The registry domain ID is 2452319255_DOMAIN_COM-VRSN. The domain uses two HostGator name servers, NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM, and carries four client-side protection status codes (clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited and clientUpdateProhibited). DNSSEC is listed as unsigned.

Task 2 – WhatWeb
Command: whatweb networkwalks.com
WhatWeb fingerprinted the technologies behind the website. It confirmed that http://networkwalks.com issues a 301 redirect to the HTTPS version, and identified the site as running Apache with WordPress 7.1 and the WordPress Download Manager 3.3.58 plugin, together with Bootstrap 7.1, jQuery 3.7.1, HTML5 and Google Tag Manager. It also returned the site title "Networkwalks Academy", the contact address info@networkwalks.com and the server IP address 192.232.216.135.

Task 3 – nslookup
Command: nslookup networkwalks.com
Using the public resolver at 8.8.8.8, nslookup returned a non-authoritative answer resolving networkwalks.com to the IPv4 address 192.232.216.135. This matched the IP address that WhatWeb had already reported, confirming the result from a second independent source.

Task 4 – curl -I
Command: curl -I https://networkwalks.com
The site responded with HTTP/2 200. The response headers disclosed server: Apache and x-nginx-cache: WordPress, which confirms both the web server and the CMS. The link header exposed the WordPress REST API endpoint /wp-json/, and a __wpdm_client cookie was set with the HttpOnly and secure flags. A permissions-policy header and a referrer-policy of no-referrer-when-downgrade were also present.

Task 5 – wafw00f
Command: wafw00f networkwalks.com
wafw00f v2.4.2 sent two requests to the site and reported that https://networkwalks.com sits behind a ModSecurity (SpiderLabs) Web Application Firewall. This tells an attacker that simple, unmodified attack payloads are likely to be filtered before they reach the application.

Task 6 – dnsrecon
Command: dnsrecon -d networkwalks.com
dnsrecon enumerated the DNS records for the domain and found 8 records in total. The DNSSEC query returned no answer, consistent with the unsigned status seen in WHOIS. The SOA record points to ns6135.hostgator.com (50.87.144.87), with a second name server ns6136.hostgator.com (192.232.216.131). Both name servers disclosed their BIND version string as 9.16.23-RH. The MX record points to mail.networkwalks.com (192.232.216.135) and the A record resolves to the same address. The SPF record is v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all, alongside a Google site-verification TXT record. Several _autodiscover._tcp SRV records point to cpanelemaildiscovery.cpanel.net on port 443, indicating cPanel-managed email.

4.2  Network Scanning with Zenmap (W2-PM5)
For the second activity I used Zenmap to perform network discovery on my own local network. The practical required me to identify my local IP address and subnet, discover the live hosts, record their IP and MAC addresses, and generate and save a network topology.

I first ran the Windows ipconfig command to identify my local IP address and LAN subnet. The active adapter was "Wireless LAN adapter Wi-Fi"; the other adapters listed (vEthernet, Ethernet 2 and Ethernet 3) were virtual adapters belonging to Hyper-V and VirtualBox, and one Ethernet adapter was reported as media disconnected. Reading the IPv4 address and subnet mask from the Wi-Fi adapter gave me the subnet to scan. I then entered that subnet into Zenmap and ran a ping scan.

| Item | Result |
|---|---|
| Local IP address (this PC) | 10.134.29.62 |
| Subnet mask | 255.255.255.0 |
| LAN subnet scanned | 10.134.29.0/24 |
| Default gateway | 10.134.29.192 |
| Nmap command used | `nmap -sn 10.134.29.0/24` |
| Scan profile | Ping Scan (host discovery only) |
| Total addresses scanned | 256 |
| Live hosts discovered | 3 |
| Scan duration | 19.38 seconds |

The scan reported 256 IP addresses scanned with 3 hosts up. The discovered hosts and their MAC addresses are listed below.
| IP Address | MAC Address | Observation |
|---|---|---|
| 10.134.29.62 | Not reported (scanning host itself) | My own laptop - the machine running the scan |
| 10.134.29.183 | 2A:2E:D0:13:1C:60 | Unidentified client device on the same subnet |
| 10.134.29.192 | C2:7B:09:2A:32:10 | Default gateway / access point |

| IP Address | MAC Address | Observation |
|---|---|---|
| 10.134.29.62 | Not reported (scanning host itself) | My own laptop - the machine running the scan |
| 10.134.29.183 | 2A:2E:D0:13:1C:60 | Unidentified client device on the same subnet |
| 10.134.29.192 | C2:7B:09:2A:32:10 | Default gateway / access point |

No MAC address was reported for 10.134.29.62 because that is the machine performing the scan; a host does not ARP for its own address, so Nmap has no MAC entry to display. The vendor lookup returned "Unknown" for both of the other MAC addresses, meaning the OUI prefixes were not present in Nmap’s vendor database. It is worth noting that both prefixes (2A: and C2:) have the locally-administered bit set, which is typical of randomised or virtualised MAC addresses rather than factory-assigned ones.
After completing the scan I opened the Topology tab in Zenmap, reviewed the legend, and used Save Graphic to export the network topology in PDF format to the desktop, as required by the practical task.

5. Risk Analysis / Impact
   
Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Level |
|---|---|---|---|---|
| 1 | Web technology and version information exposed | WhatWeb identified WordPress 7.1, WP Download Manager 3.3.58, Bootstrap 7.1, jQuery 3.7.1 and Apache | Version information allows an attacker to look up known advisories for the exact software in use | Medium |
| 2 | Server IP address identifiable | nslookup and WhatWeb both resolved the domain to 192.232.216.135 | Reveals the network location of the web service and the hosting provider | Low |
| 3 | HTTP technical information exposed | curl -I returned server: Apache, x-nginx-cache: WordPress and the /wp-json/ REST API endpoint | Assists technology fingerprinting and gives a starting point for further enumeration | Low |
| 4 | WAF technology identifiable | wafw00f v2.4.2 identified ModSecurity (SpiderLabs) | Reveals the security architecture, which an attacker may attempt to tune evasion against | Low |
| 5 | DNS infrastructure and BIND version exposed | dnsrecon returned SOA/NS at HostGator, MX mail.networkwalks.com, SPF record and BIND version 9.16.23-RH | A disclosed resolver version and full record set help build a broad infrastructure profile | Medium|
| 6 | DNSSEC not enabled | WHOIS reports DNSSEC: unsigned; dnsrecon returned no answer for the DNSSEC query | Without DNSSEC, DNS responses cannot be cryptographically validated by resolvers | Medium |
| 7 | Unidentified live host on local network | Zenmap discovered 10.134.29.183 (MAC 2A:2E:D0:13:1C:60), which I could not attribute to a known device | Unknown or unauthorised devices may be present on a network that is treated as trusted | Medium |

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities.
These two modules involved information gathering and host discovery only. No exploitation or vulnerability validation was performed. Therefore the presence of a software version, an IP address or a DNS record does not by itself mean that the system is vulnerable. Further authorised security testing would be required to confirm any actual vulnerability.

6. Recommendations
Based on the observations from these activities, I recommend the following security improvements:
1.	Review publicly exposed technology information. Organisations should regularly review what information about their web technologies, CMS and plugins is publicly visible, and suppress version strings where they serve no functional purpose.
2.	Keep software updated. The CMS, plugins and other web components should be updated regularly and reviewed against current security advisories.
3.	Review HTTP response headers. Headers such as server and x-nginx-cache should be reviewed to determine whether unnecessary technical detail is being exposed, and security headers should be added where appropriate.
4.	Suppress the BIND version string. The name servers currently disclose version 9.16.23-RH; this should be hidden so that resolver software cannot be fingerprinted from a DNS query.
5.	Consider enabling DNSSEC. The domain is currently unsigned, so DNS responses cannot be cryptographically validated by resolvers.
6.	Review DNS records regularly. Records should be checked periodically to ensure only required services and information are publicly exposed.
7.	Keep the WAF configured and monitored. ModSecurity is already in place and should remain enabled and tuned, since it filters unsophisticated attacks before they reach the application.
8.	Perform regular internal network discovery. Organisations should periodically scan their own networks to identify active devices and build a baseline of what is normal.
9.	Investigate unknown devices. Any unexpected device found during a scan, such as the unattributed host in this exercise, should be identified and verified.
10.	Maintain network documentation. Topology diagrams and device inventories should be documented and kept up to date.
11.	Perform security testing only with authorisation. Reconnaissance and scanning should only be carried out against systems and networks where appropriate permission has been granted.

7. Conclusion
During Week 2 of my Cybersecurity & Ethical Hacking internship at Networkwalks, I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activity I used six Kali Linux tools to gather information about the target domain. I learned how WHOIS exposes registration and name server detail, how WhatWeb fingerprints the CMS and its plugins, how nslookup resolves a domain to its hosting IP, how curl -I reveals server technology through response headers, how wafw00f identifies a Web Application Firewall, and how dnsrecon enumerates the wider DNS footprint including mail, SPF and service records. Working through them in sequence showed me how each tool independently confirms and extends what the previous one found – the IP address returned by WhatWeb, nslookup and dnsrecon was the same in all three cases.

In the network scanning activity I used ipconfig to identify my local configuration and Zenmap to discover the active hosts on my subnet. An early attempt scanned the wrong range and returned only localhost, which taught me that the scan target has to be derived from the correct active adapter rather than assumed – the virtual adapters created by VirtualBox and Hyper-V appear alongside the real Wi-Fi adapter and are easy to confuse. Once I read the subnet from the correct adapter, the scan returned the live hosts, their MAC addresses and a topology diagram.

These exercises showed me that information gathering is a substantial part of security work. Before any attempt at exploitation, a security professional can learn a great deal about an environment by carefully analysing publicly available information and ordinary network responses. They also showed me the value of clear documentation: a good report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk. 

Finally, they reinforced that reconnaissance and scanning must always be performed within an authorised scope; these activities were completed as part of an assigned educational laboratory on my own network and on a target for which permission had been granted.

8. Evidences Collected
Evidence 1 – WHOIS lookup of networkwalks.com
<img width="938" height="484" alt="image" src="https://github.com/user-attachments/assets/0d81a216-37af-4c6b-8b09-e81dd949c2cd" />

Evidence 2 – WhatWeb technology fingerprint
 <img width="938" height="513" alt="image" src="https://github.com/user-attachments/assets/b91f7959-23ab-4b25-8c9d-5de108769b3c" />

Evidence 3 – nslookup DNS resolution
<img width="938" height="513" alt="image" src="https://github.com/user-attachments/assets/98d9d40f-651b-440a-ba08-a77990e61b56" />

Evidence 4 – curl -I HTTP response headers
<img width="938" height="575" alt="image" src="https://github.com/user-attachments/assets/9cd4fbc2-ca81-49e8-92c5-94bb130e7cf4" />

Evidence 5 – wafw00f WAF detection
 <img width="938" height="578" alt="image" src="https://github.com/user-attachments/assets/246b7054-0964-4062-9e72-b8254b1efb83" />

Evidence 6 – dnsrecon DNS record enumeration
<img width="938" height="508" alt="image" src="https://github.com/user-attachments/assets/673ffacd-bd57-4219-9a89-2572c7aef9a4" />

 Evidence 7 – Zenmap ping scan of 10.134.29.0/24 (hosts discovered)
 <img width="938" height="295" alt="image" src="https://github.com/user-attachments/assets/9c7b3635-5e65-46df-8b58-144501337891" />

 Evidence 8 – Zenmap network topology (saved to PDF)
 <img width="516" height="463" alt="image" src="https://github.com/user-attachments/assets/7375ba7d-9cd2-4739-8076-5189ca3001a8" />

 
- End -
Author
Ademola Oduola
Cybersecurity Professional – Batch B083, Networkwalks
GitHub: github.com/Ademolaoduola
LinkedIn: https://www.linkedin.com/in/ademolaoduola?originalSubdomain=ng
Project Information
Program Name: Cybersecurity & Ethical Hacking Internship at Networkwalks  |  Week: 02  |  Modules: W2-PM1, W2-PM5  |  Repository: GitHub


