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
We gave the following to at least two AI tools: <describe what you gave them>.

| Fault (in our words) | We found it? | AI #1 (<name>) | AI #2 (<name>) | Was each AI actually right? |
| --- | --- | --- | --- | --- |
| | | | | |
| | | | | |

For each AI suggestion that was WRONG or misleading, describe it and how you
verified it was wrong:
<notes>

## Part 3d: Verdict
- Where our human diagnosis beat the AIs:
- Where an AI genuinely helped:
- What we learned about trusting AI to diagnose a live system it cannot see:
