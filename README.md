# Server OS Midterm — Urban

## Project overview

Urban is a five-member server administration midterm project for the  192-442 Server Operating Systems course. The project focuses on diagnosing and repairing broken infrastructure services in an Ubuntu Linux environment hosted on KillerCoda for the company domain `novapeak.lab` (checker group `lab01`). The team investigated root causes, resolved critical configuration defects across BIND9 DNS and ISC DHCP servers, restored full network service integration, documented each fault in an incident log, and audited AI-assisted troubleshooting capabilities.

## Team members

| Name | Student ID | GitHub username |
|---|---|---|
| Sai Wanna Htoo | 6705140028 | [lelouchlynx](https://github.com/lelouchlynx) |
| Toe Twel Tar Htut | 6705140007 | [YollaYollie](https://github.com/YollaYollie) |
| Kyaw San | 6705140029 | [KyawSanAlpha](https://github.com/KyawSanAlpha) |
| Aung Chan Myae | 6705140035 | [Rowan10-uzzz](https://github.com/Rowan10-uzzz) |
| Nyi Sett | 6705140078 | [Wha1e0134](https://github.com/Wha1e0134) |

- **Group:** Urban (Afternoon Class)
- **Company / Domain:** `novapeak.lab`
- **Checker Group ID:** `lab01`

## Repository structure

```text
server-os-midterm-urban/
├── fixed-configs/
│   ├── db.novapeak.lab            # Forward DNS zone records for novapeak.lab
│   ├── db.novapeak.lab.rev        # Reverse DNS zone PTR records (192.168.19.0/24)
│   ├── dhcpd.conf                 # ISC DHCP pool, gateway, and DNS options
│   ├── isc-dhcp-server            # Interface binding settings (lan0)
│   ├── named.conf.local           # BIND9 local zone configuration
│   └── named.conf.options         # BIND9 server options and query settings
├── AI_CRITIQUE.md                 # AI diagnostic usage, evaluation, and critique
├── INCIDENT_LOG.md                # Systematic incident response and fault investigation log
├── README.md                      # Project overview, team members, and navigation
└── session.log                    # Terminal recording of diagnostic session and checker runs
└── checker_result.txt             # checker runs result
```

## Checker

The automated checker evaluates 10 grading checkpoints (C1–C10): validating BIND9 configuration syntax and zone loading (C4), server reachability and authority for `novapeak.lab` (C1), correct forward A records for all hosts (`ns1`, `www`, `mail`, `api`) (C2), reverse PTR lookup resolution (C3), ISC DHCP daemon configuration and runtime health (C5), client lease acquisition (C6), subnet address pool and netmask allocation (C7), default gateway routing (C8), company DNS advertisement (`192.168.19.10`) (C9), and domain search name propagation (`novapeak.lab`) (C10).

## Project Components

| Part | Component |
|---|---|
| [Incident Log](INCIDENT_LOG.md) | Systematic incident response log — symptoms, hypotheses, proofs, fixes, and confirmations |
| [AI Critique & Evaluation](AI_CRITIQUE.md) | Declaration of AI tools used, human vs. AI diagnosis comparison, and reliability analysis |
| [DNS Zone Declarations](fixed-configs/named.conf.local) | BIND9 local zone declarations mapping forward and reverse database files |
| [DNS Options Configuration](fixed-configs/named.conf.options) | BIND9 daemon options, directory paths, and query permissions |
| [Forward DNS Zone File](fixed-configs/db.novapeak.lab) | Forward zone records (`SOA`, `NS`, and `A` records for `ns1`, `www`, `mail`, `api`) |
| [Reverse DNS Zone File](fixed-configs/db.novapeak.lab.rev) | Reverse zone records (`PTR` pointers for `192.168.19.0/24`) |
| [DHCP Server Configuration](fixed-configs/dhcpd.conf) | Subnet pool definition, router gateway, lease times, and company DNS option |
| [DHCP Interface Configuration](fixed-configs/isc-dhcp-server) | Daemon interface binding configured to `lan0` |
| [Session Audit Log](session.log) | Complete shell session recording of diagnosis, configuration edits, and verification runs |

## Conclusion

Our team methodically diagnosed and resolved all service misconfigurations in the `novapeak.lab` environment. Initially, critical DNS resolution and DHCP operations failed due to incorrect zone file references, bad host IP records, interface binding omissions, and improper DNS option advertisements. By formulating hypotheses, confirming proofs with diagnostic commands, and applying verified configuration fixes, our team achieved a 100% pass rate (22/22 points across C1–C10) on the automated evaluation suite. Through this project, we reinforced our practical experience in Linux system administration, network service integration, and evaluating the strengths and limitations of AI diagnostic tools on live systems.
