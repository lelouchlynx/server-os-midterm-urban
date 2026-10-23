# Incident Log

Group: Urban   
Members:  
  - Sai Wanna Htoo (6705140028) — `lelouchlynx`
  - Toe Twel Tar Htut (6705140007) — `YollaYollie`
  - Kyaw San (6705140029) — `KyawSanAlpha`
  - Aung Chan Myae (6705140035) — `Rowan10-uzzz`
  - Nyi Sett (6705140078) — `Wha1e0134`
Company / domain: novapeak.lab

Fill in ONE entry per fault you found. Paste real command output -- an entry
without evidence earns little. Copy this block for each fault.

---

## Fault 1

- **Symptom (what we observed):**
  ```
  [ 1/2 ] C4 named-checkconf -z FAILED; named-checkzone clean
  ```
- **Hypothesis (what we suspected and why):**
  BIND was using an incorrect DNS zone file path because named-checkconf -z failed while named-checkzone was clean. This suggested the zone file itself was valid but BIND could not load it through its configuration.
- **Proof (the command that confirmed the cause):**
  ```
  [?2004h[01;32mroot@ubuntu[00m:[01;34m~[00m$ [7mnamed-checkconf -z[27mnamed-checkconf -z
  [?2004lzone novapeak.lab/IN: loading from master file /etc/bind/db.MISSING failed: file not found
  zone novapeak.lab/IN: not loaded due to errors.
  ```
- **Fix (what we changed):**
  sed -i 's|/etc/bind/db.MISSING|/etc/bind/db.novapeak.lab|' /etc/bind/named.conf.local
- **Confirmation (output showing it now works):**
  ```
  zone novapeak.lab/IN: loaded serial 2024010101
  zone 19.168.192.in-addr.arpa/IN: loaded serial 2024010101
  
  [   2/2 ] C4   named-checkconf -z clean; named-checkzone clean
  ```
- **Found/proved by:** Nyi Sett

---

## Fault 2 - DNS Reachability Blocked by Firewall Rule and Bad Zone File Reference (C1 & C3)

- **Symptom (what we observed):**

Running python3 /root/grading/check_lab.py --group lab01 --phase build failed both reachability (C1) and reverse lookup (C3), along with zone config verification (C4)

  ```
  [ 0/2 ] C1 client cannot reach named at 192.168.19.10 (timeout/no status). Check listen-on, allow-query, firewall, and that it serves on lan0.
[ 0/2 ] C3 reverse lookup of 192.168.19.10 did not return ns1 (got nothing). Check the reverse zone and its PTR.
[ 1/2 ] C4 named-checkconf -z FAILED; named-checkzone clean
  ```
  Running named-checkconf -z directly on the server outputted:

- **Hypothesis (what we suspected and why):**
  There were two underlying causes.

    (1) BIND failed to load its zone database because /etc/bind/named.conf.local referenced a non-existent file (/etc/bind/db.MISSING) instead of the valid file in /etc/bind/.

    (2) External queries timed out because an iptables firewall rule was dropping incoming UDP traffic on DNS port 53.

- **Proof (the command that confirmed the cause):**
First, we verified the file mismatch in /etc/bind/named.conf.local

    ```
       zone "novapeak.lab" {
        type master;
        file "/etc/bind/db.MISSING";
      };
    ```
Checking directory contents with ls -l /etc/bind/db.* showed the real forward zone file was /etc/bind/db.novapeak.lab.

Second, we inspected kernel firewall rules using iptables -vnL INPUT --line-numbers

    ```
      Chain INPUT (policy ACCEPT 498 packets, 33973 bytes)
       num   pkts bytes target     prot opt in     out     source               destination          
       1        0     0 DROP       17   --  *      *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
      };
    ```
Rule #1 explicitly dropped incoming UDP traffic on port 53 (prot 17), blocking all client network access to BIND.

- **Fix (what we changed):**
(1) Edited /etc/bind/named.conf.local using nano and changed file "/etc/bind/db.MISSING"; to file "/etc/bind/db.novapeak.lab";

(2) Restarted BIND:

    ```
     systemctl restart named
    ```
(3) Deleted the blocking firewall rule:

    ```
     iptables -D INPUT -p udp --dport 53 -j DROP
    ```

- **Confirmation (output showing it now works):**

Running named-checkconf -z confirmed the zone loaded successfully
    ```
     zone novapeak.lab/IN: loaded serial 2024010101
     zone 19.168.192.in-addr.arpa/IN: loaded serial 2024010101
    ```

Re-checking iptables -vnL INPUT --line-numbers verified the DROP rule was removed. Finally, running check_lab.py confirmed both C1 and C3 passed

    ```
    [ 2/2 ] C1 named is authoritative for novapeak.lab and reachable from the client.
    [ 2/2 ] C3 reverse lookup of 192.168.19.10 returns ns1.
   ```
- **Found/proved by:** Kyaw San

---

## Fault 3

- **Symptom (what we observed):**
  ```
  <paste the command and its output that showed something was wrong>
  ```
- **Hypothesis (what we suspected and why):**
  <your reasoning>
- **Proof (the command that confirmed the cause):**
  ```
  <paste the command and output that proved it>
  ```
- **Fix (what we changed):**
  <the change>
- **Confirmation (output showing it now works):**
  ```
  <paste the corrected output>
  ```
- **Found/proved by:** <member name>

---