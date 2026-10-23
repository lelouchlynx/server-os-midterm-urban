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

## Fault 2

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