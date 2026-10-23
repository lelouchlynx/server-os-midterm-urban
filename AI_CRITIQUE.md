# AI Usage and Critique

Group: Urban   

Members: Toe Twel Tar Htut (6705140007)
Sai Wanna Htoo (6705140028)
Kyaw San (6705140029)
Aung Chan Myae (6705140035)
Nyi Sett (6705140078)
## Part 3a: Declaration (required)
| AI tool | What we used it for |
| --- | --- |
| ChatGPT | Asked to explain why named-checkconf -z failed while named-checkzone passed, and to help diagnose the DNS configuration problem.|
| ChatGPT | Asked for initial diagnostic steps regarding why check_lab.py reported a network reachability timeout on C1. |
| Gemini | Asked to analyze our BIND named.conf.options and iptables rule outputs to determine why reverse lookup (C3) and client queries failed.|
| Claude | Asked to independently diagnose the C2 DNS fault, identify why the api A record returned the wrong IP address, and recommend a fix and verification commands. |
| ChatGPT | Asked why DHCP was advertising 8.8.8.8 (C9) and preventing the client from resolving internal domain names like www (C11). |
| Claude | Asked to verify the correct syntax for updating option domain-name-servers inside /etc/dhcp/dhcpd.conf to fix internal DNS integration. |

(If you used no AI, write "None" and say how you worked instead.)

## Part 3b: Confirm the team's diagnosis

We used the checker and terminal commands in KillerCoda to investigate the failures. AI was used as a support tool for explaining command output and suggesting areas to inspect, but we did not treat an AI response as proof. We checked the actual configuration and system state, applied the configuration changes ourselves, and reran the checker to confirm the outcome.

The main faults identified in our troubleshooting were:

1. **C4 — Incorrect forward-zone file path.** The first named-checkconf -z output said that /etc/bind/db.MISSING could not be found. The real forward zone file, `/etc/bind/db.novapeak.lab`, existed and was valid. We corrected the file path in `/etc/bind/named.conf.local`.
2. **C1 and the initial C3 failure — DNS traffic blocked by the firewall.** Even after the zone loaded, the client DNS request timed out. iptables -vnL INPUT --line-numbers showed a rule dropping incoming UDP traffic on port 53. We removed that rule. The client could then reach the authoritative DNS server, and the reverse lookup for 192.168.19.10 returned `ns1`.
3. **C2 — Incorrect api A record.** The checker reported 192.168.19.1 instead of `192.168.19.40`. We compared the forward zone with the expected record, changed the address, and verified it with `named-checkzone`, `dig`, and the checker.
4. **C9 and C11 — DHCP advertised the wrong DNS resolver.** DHCP initially advertised `8.8.8.8`, not the company's DNS server at `192.168.19.10`. We corrected the DNS option in `/etc/dhcp/dhcpd.conf`, checked the DHCP syntax, restarted the service, and reran the checker. C9 and C11 then passed.

## Part 3c: Putting the AIs to the test

We asked AI tools about selected symptoms and configuration excerpts. The table below summarises the purpose of those consultations and evaluates the relevant advice against our recorded terminal evidence.

| Fault (in our words) | Did we find it through our own checks? | AI #1 summary | AI #2 summary | Was the advice actually right? |
|---|---|---|---|---|
| C4 — BIND referenced a missing zone file | Yes. named-checkconf -z identified `/etc/bind/db.MISSING`, and the real zone file was present under a different name. | ChatGPT helped explain why checking the zone file alone was not enough; BIND also had to load the correct file through its configuration. | No second-AI response for this particular fault was preserved in our notes. | The explanation was consistent with the error. The proof was the failed configuration check followed by a clean named-checkconf -z after we corrected the path. |
| C1 and initial C3 — Client DNS queries timed out | Yes. We checked BIND's options and then inspected the firewall rules. | ChatGPT was used for diagnostic directions for the reachability timeout, including checking service configuration and possible network blocking. | Gemini was asked to interpret the BIND options and firewall output during the C1/C3 investigation. The full response was not retained, so we cannot reliably report its exact wording. | The live output showed a UDP port 53 DROP rule. After we removed it, C1 passed and C3 returned the expected ns1 PTR. These test results prove the fix worked; they do not by themselves prove which AI, if either, identified the exact rule first. |
| C2 — api returned the gateway address | Yes. The checker showed 192.168.19.1 instead of `192.168.19.40`, and the forward zone contained the wrong address. | Claude was asked to diagnose the wrong A record and suggest verification commands. | No separate second-AI answer for this exact record was retained in our notes. | The corrected record was verified: dig @192.168.19.10 api.novapeak.lab +short returned `192.168.19.40`, and C2 later received full marks. Because the complete Claude answer was not saved, we cannot claim with certainty that Claude gave the exact correction we used. |
| C9/C11 — DHCP supplied the public resolver | Yes. The checker explicitly reported 8.8.8.8 where 192.168.19.10 was expected. | ChatGPT helped explain why the advertised resolver mattered for clients resolving the company's internal names. | Claude was asked to check the syntax of the DHCP DNS option. | The final setting was correct and the configuration passed `dhcpd -t -cf /etc/dhcp/dhcpd.conf`. After restarting DHCP, the checker confirmed that C9 advertised 192.168.19.10 and C11 resolved www through the leased resolver. The checker confirms the resulting configuration, while the missing raw AI transcripts limit our ability to assess each tool's exact wording separately. |

## Part 3d: Verdict
- Where our human diagnosis beat the AIs:
- Where an AI genuinely helped:
- What we learned about trusting AI to diagnose a live system it cannot see:
