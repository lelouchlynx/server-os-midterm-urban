# AI Usage and Critique

Group: Urban   

Members:
- Toe Twel Tar Htut (6705140007)
- Sai Wanna Htoo (6705140028)
- Kyaw San (6705140029)
- Aung Chan Myae (6705140035)
- Nyi Sett (6705140078)

## Part 3a: Declaration

| AI tool | What we used it for |
|---|---|
| ChatGPT | Asked to explain why named-checkconf -z failed while named-checkzone passed, and to help diagnose the DNS configuration problem. |
| None | C1 and C3 were diagnosed by running the checker, inspected the firewall rules using `iptables -vnL INPUT --line-numbers`, identified the UDP port 53 DROP rule, removed it, and reran the checker to verify the results. |
| None | C2 was diagnosed by comparing the checker's expected and actual IP addresses, inspected the forward DNS zone file, corrected the `api` A record, and verified the result using `named-checkzone`, `dig`, and the checker. |
| ChatGPT | Asked why DHCP was advertising 8.8.8.8 (C9) and preventing the client from resolving internal domain names like www (C11). | 

## Part 3b: Confirm the team's diagnosis

We used the checker and terminal commands in KillerCoda to investigate the failures. AI was used as a support tool for explaining command output and suggesting areas to inspect, but we did not treat an AI response as proof. We checked the actual configuration and system state, applied the configuration changes ourselves, and reran the checker to confirm the outcome.



1. **C4 — Incorrect forward-zone file path.** The first named-checkconf -z output said that /etc/bind/db.MISSING could not be found. The real forward zone file, `/etc/bind/db.novapeak.lab`, existed and was valid. We corrected the file path in `/etc/bind/named.conf.local`.
2. **C1 and the initial C3 failure — DNS traffic blocked by the firewall.** Even after the zone loaded, the client DNS request timed out. iptables -vnL INPUT --line-numbers showed a rule dropping incoming UDP traffic on port 53. We removed that rule. The client could then reach the authoritative DNS server, and the reverse lookup for 192.168.19.10 returned `ns1`.
3. **C2 — Incorrect api A record.** The checker reported 192.168.19.1 instead of `192.168.19.40`. We compared the forward zone with the expected record, changed the address, and verified it with `named-checkzone`, `dig`, and the checker.
4. **C9 and C11 — DHCP advertised the wrong DNS resolver.** DHCP initially advertised `8.8.8.8`, not the company's DNS server at `192.168.19.10`. We corrected the DNS option in `/etc/dhcp/dhcpd.conf`, checked the DHCP syntax, restarted the service, and reran the checker. C9 and C11 then passed.

## Part 3c: Putting the AIs to the test

We tested with ChatGPT and Claude C4, C9, C11 diagnose and fix with each AI testing.

| Fault | You found it? | AI #1 (ChatGPT) | AI #2 (Claude) | Was each AI actually right? |
| --- | --- | --- | --- | --- |
| **C4: Incorrect DNS zone file path** | Yes | Explained why `named-checkconf -z` failed while `named-checkzone` passed. | Suggested checking the zone file path in `named.conf.local` and validating the configuration after correction. | The diagnosis matched the evidence. We corrected the missing file reference, and C4 passed. |
| **C1/C3: DNS timeout and reverse lookup failure** | Yes | Not used for these faults. | Not used. | We identified the UDP port 53 DROP rule using `iptables`, removed it, and confirmed C1 and C3 passed. |
| **C2: Incorrect `api` A record** | Yes | Not used for this fault. | Not used. | We compared the expected and actual IP addresses, corrected the record, and verified it with `dig` and the checker. C2 passed. |
| **C9/C11: DHCP advertised the wrong DNS resolver** | Yes | Explained why clients need the company's DNS server to resolve internal hostnames. | Suggested setting the DHCP DNS option to `192.168.19.10`, then validating the configuration and restarting DHCP. | The advice matched the intended configuration. After the change, C9 and C11 passed. |

## Part 3d: Verdict

- **Where our human diagnosis beat the AIs:** The team connected the symptoms to different causes rather than treating every failure as the same DNS problem. We inspected the actual zone-file path, firewall rules, forward DNS record, and DHCP option. We then made the configuration changes in KillerCoda and tested the outcome. AI could suggest possible causes, but it could not see the live server state without the outputs we supplied.

- **Where an AI genuinely helped:** AI was useful for explaining the difference between configuration and zone checks, suggesting places to investigate after a timeout, and checking the syntax of a DHCP option. It helped us understand possible causes, but the team carried out the configuration changes and verified the result.

- **What we learned about trusting AI to diagnose a live system it cannot see:** AI suggestions can be plausible without being confirmed. A real server may have several faults at once, and a fix that addresses only one symptom may not restore the whole service. We learned to compare suggestions with command output, make targeted changes, and rerun the checker from the client side. The final recorded build result was **25.0/25**, with C1–C11 passing. This confirms the tested DNS/DHCP build passed the checker; it does not establish that every AI suggestion was correct.