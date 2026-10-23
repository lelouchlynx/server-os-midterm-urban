# Incident Log

Group: Urban   
Members:  
  - Sai Wanna Htoo (6705140028) — `lelouchlynx`
  - Toe Twel Tar Htut (6705140007) — `YollaYollie`
  - Kyaw San (6705140029) — `KyawSanAlpha`
  - Aung Chan Myae (6705140035) — `Rowan10-uzzz`
  - Nyi Sett (6705140078) — `Wha1e0134`
  
Company / domain: novapeak.lab



---

## Fault 1 — Incorrect Forward-Zone Filename (C4)

- **Symptom (what we observed):**
  ```
  [ 1/2 ] C4 named-checkconf -z FAILED; named-checkzone clean
    ...
  TOTAL: 11.0 / 25
  ```
- **Hypothesis (what we suspected and why):**
  BIND was using an incorrect DNS zone file path because named-checkconf -z failed while named-checkzone was clean. This suggested the zone file itself was valid but BIND could not load it through its configuration.
- **Proof (the command that confirmed the cause):**
First, we verified the file mismatch in /etc/bind/named.conf.local

  ```
       zone "novapeak.lab" {
        type master;
        file "/etc/bind/db.MISSING";
      };
  ```
- **Fix (what we changed):**
  ```
  nano /etc/bind/named.conf.local
  ```
- **Confirmation (output showing it now works):**
  ```
  zone novapeak.lab/IN: loaded serial 2024010101
  zone 19.168.192.in-addr.arpa/IN: loaded serial 2024010101
  
  [  2/2 ] C4   named-checkconf -z clean; named-checkzone clean
    TOTAL: 12.0 / 25
  ```
- **Found/proved by:** Nyi Sett (6705140078) — `Wha1e0134`

---

## Fault 2 — DNS Reachability Blocked by Firewall Rule (C1 & C3)

- **Symptom (what we observed):**

  After Fault 1 was corrected, the DNS zone configuration loaded successfully, but the client still could not reach the DNS server or complete the reverse lookup.

  Running `python3 /root/grading/check_lab.py --group lab01 --phase build` produced:

  ```
  [ 0/2 ] C1 client cannot reach named at 192.168.19.10 (timeout/no status). Check listen-on, allow-query, firewall, and that it serves on lan0.
  [ 0/2 ] C3 reverse lookup of 192.168.19.10 did not return ns1 (got nothing). Check the reverse zone and its PTR.
    ...
  TOTAL: 12.0 / 25
  ```


- **Hypothesis (what we suspected and why):**
  We suspected that a firewall rule was blocking DNS requests from the client. The timeout reported by C1 suggested a reachability problem, which could also prevent the client from completing the reverse lookup tested by C3.

- **Proof (the command that confirmed the cause):**
We inspected the firewall rules using:

  ```
       iptables -vnL INPUT --line-numbers
  ```
The output showed a rule dropping incoming UDP traffic on DNS port 53:

  ```
    Chain INPUT (policy ACCEPT 498 packets, 33973 bytes)
    num  pkts bytes target  prot opt in  out  source       destination
    1    0    0     DROP    17   --  *   *    0.0.0.0/0    0.0.0.0/0    udp dpt:53
  ```
This rule provided evidence that incoming DNS queries over UDP were being dropped by the firewall.

- **Fix (what we changed):**
We removed the blocking firewall rule:

  ```
    iptables -D INPUT -p udp --dport 53 -j DROP
  ```
We then inspected the firewall rules again to confirm that the DROP rule had been removed.

  ```
   iptables -D INPUT -p udp --dport 53 -j DROP
  ```

- **Confirmation (output showing it now works):**
After removing the firewall rule, we reran the checker:
  ```
   [ 2/2 ] C1 named is authoritative for novapeak.lab and reachable from the client.
   [ 2/2 ] C3 reverse lookup of 192.168.19.10 returns ns1.
   TOTAL: 18.25 / 25
  ```
These results confirmed that the client could reach the authoritative DNS server and that the reverse lookup worked. C4 remained passing from the previous fix.

- **Found/proved by:** Kyaw San (6705140029) — `KyawSanAlpha`

## Fault 3 — Incorrect API Host Address (C2)

- **Symptom (what we observed):**
  ```
    root@ubuntu:~# python3 /root/grading/check_lab.py --group lab01 --phase build
   [2.25/3 ] C2 api expected 192.168.19.40 got ['192.168.19.1']
  ...
  TOTAL: 18.25 / 25
  ```
- **Hypothesis (what we suspected and why):**
  The `api` A record gives 192.168.19.1. That is the gateway address (`option routers 192.168.19.1`), not an API server. The reverse zone already maps host 40 to `api`, so the forward record is the wrong one.
- **Proof (the command that confirmed the cause):**
  ```
    root@ubuntu:~# cat /etc/bind/db.novapeak.lab
  $TTL 3600
  @ IN SOA ns1.novapeak.lab. admin.novapeak.lab. ( 2024010101 3600 1800 604800 3600 )
  @ IN NS ns1.novapeak.lab.
  ns1 IN A 192.168.19.10
  www IN A 192.168.19.20
  mail IN A 192.168.19.30
  api IN A 192.168.19.1
  root@ubuntu:~# cat /etc/bind/db.novapeak.lab.rev
  $TTL 3600
  @ IN SOA ns1.novapeak.lab. admin.novapeak.lab. ( 2024010101 3600 1800 604800 3600 )
  @ IN NS ns1.novapeak.lab.
  10 IN PTR ns1.novapeak.lab.
  20 IN PTR www.novapeak.lab.
  30 IN PTR mail.novapeak.lab.
  40 IN PTR api.novapeak.lab.
  ```
- **Fix (what we changed):**
  In `/etc/bind/db.novapeak.lab` we changed `api IN A 192.168.19.1` to `api IN A 192.168.19.40`. Then we ran `named-checkzone`, `named-checkconf -z` and `systemctl restart named`.
- **Confirmation (output showing it now works):**
  ```
  root@ubuntu:~# named-checkzone novapeak.lab /etc/bind/db.novapeak.lab
  zone novapeak.lab/IN: loaded serial 2024010101
  OK
  root@ubuntu:~# dig @192.168.19.10 api.novapeak.lab +short
  192.168.19.40
  root@ubuntu:~# python3 /root/grading/check_lab.py --group lab01 --phase build
   [ 3.0/3 ] C2 all four A records correct
  ...
  TOTAL: 19.0 / 25

  ```
- **Found/proved by:** Sai Wanna Htoo (6705140028) — `lelouchlynx`

---

## Fault 4: DHCP Advertises External Resolver Blocking Client Domain Resolution (C9 & C11)

  - **Symptom (what we observed):**

    Running python3 /root/grading/check_lab.py --group lab01 --phase build failed both the DHCP resolver check (C9) and client resolution test (C11)
    
    ```
    [ 0/3 ] C9 DHCP advertises 8.8.8.8, expected the company DNS 192.168.19.10 -- this is the integration the whole project is about
    [ 0/3 ] C11 using the leased resolver 8.8.8.8, www did not resolve to 192.168.19.20 (got nothing). If the leased resolver is a public one, that is the integration bug.
    TOTAL: 19.0 / 25
    ```


    **Hypothesis (what we suspected and why):**
    The DHCP server configuration file (/etc/dhcp/dhcpd.conf) was set to hand out Google's public DNS IP (8.8.8.8) instead of the internal company DNS server (192.168.19.10). Because clients used Google's public DNS, they could not resolve internal domain names like www.novapeak.lab.

    **Proof (the command that confirmed the cause):**
    We inspected the DHCP configuration file using cat /etc/dhcp/dhcpd.conf:
    ```
    subnet 192.168.19.0 netmask 255.255.255.0 {
            range 192.168.19.104 192.168.19.144;
            option routers 192.168.19.1;
            option domain-name-servers 8.8.8.8;
    }
    ```

    The setting option domain-name-servers 8.8.8.8; confirmed the integration bug.

    **Fix (what we changed):**

    Edited /etc/dhcp/dhcpd.conf using nano and updated the DNS option line to

        ```
        option domain-name-servers 192.168.19.10;

        Validated syntax with dhcpd -t -cf /etc/dhcp/dhcpd.conf.

        Restarted the DHCP service:
        Bash

        systemctl restart isc-dhcp-server
        ```

    **Confirmation (output showing it now works):**
    We checked service status with systemctl status isc-dhcp-server --no-pager

    ```
    ● isc-dhcp-server.service - ISC DHCP IPv4 server
         Active: active (running) since Fri 2026-10-23 07:23:36 UTC; 7s ago
    ```

    Re-running python3 /root/grading/check_lab.py --group lab01 --phase build confirmed both C9 and C11 passed cleanly:

    ```
    [ 3/3 ] C9 DHCP advertises the company DNS (192.168.19.10)
    [ 3/3 ] C11 client resolves www via the leased resolver
      TOTAL: 25.0 / 25
    ```

  - **Found/proved by:** Aung Chan Myae (6705140035) — `Rowan10-uzzz`
---