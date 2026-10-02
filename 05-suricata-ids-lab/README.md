# Project 5: Lightweight IDS with Suricata
 
> Built a detection layer on top of my earlier attack projects, found that a default IDS ruleset missed a real exploit, and wrote a custom Suricata rule to close the gap.
 
## Summary
 
| | |
|---|---|
| **Goal** | Test whether a default IDS ruleset detects the attacks already carried out in this lab |
| **Tools** | Suricata 8.0.6, Nmap, Metasploit, Kali Linux, Metasploitable2 |
| **Key result** | Reconnaissance was detected out of the box. The vsftpd 2.3.4 exploit was not, until I wrote a custom rule |
| **Skills shown** | IDS deployment, alert analysis, custom rule writing, configuration troubleshooting |
 
## Objective
 
The goal of this project was to build a detection layer on top of the offensive work performed in Projects 2 and 3, moving from "attack" to "attack + detect." The original plan was a full SIEM deployment using Wazuh, but this was scaled down to **Suricata**, a lightweight intrusion detection system (IDS), due to host resource constraints. Suricata still delivers the core learning objective (real-time network traffic monitoring and alerting) with a much smaller footprint, running directly on the existing Kali VM rather than requiring a dedicated SIEM server.
 
This project set out to answer a practical question: **can a default, out-of-the-box IDS ruleset detect the attacks already carried out in this lab, and if not, can a custom rule close that gap?**
 
## Environment
 
| Component | Details |
|-----------|---------|
| Attacker / monitoring machine | Kali Linux (host-only network, `192.168.56.x`) |
| Target | Metasploitable2 (`192.168.56.102`) |
| IDS | Suricata 8.0.6, installed via `apt` |
| Ruleset | Emerging Threats Open (440 rules), updated with `suricata-update` |
| Configuration | `HOME_NET` set to the lab subnet (`192.168.56.0/24`), capture interface `eth0` |
 
## Results at a Glance
 
| Activity | Detected by default ruleset? | Detected after custom rule? |
|----------|------------------------------|-----------------------------|
| `nmap -sV` version scan | Yes (2 alerts) | n/a |
| `nmap -sS -A` aggressive scan | Yes (many alerts across several ports) | n/a |
| vsftpd 2.3.4 backdoor exploit | **No** | **Yes** |
 
## Methodology
 
### Test 1: Reconnaissance Detection
 
**Basic version scan:**
```bash
nmap -sV 192.168.56.102
```
Result: 2 alerts fired, *"Applayer Detect protocol only one direction"*, triggered by Nmap's version probes against PostgreSQL (port 5432) not completing a normal handshake.
 
![Suricata alerts for the Nmap version scan](./05-suricata-01-nmap-scan-detected.png)
 
**Aggressive scan:**
```bash
nmap -sS -A 192.168.56.102
```
Result: significantly more alerts, spanning multiple ports and protocols, including SMB malformed request alerts (port 445, likely from `-A`'s default script enumeration) and protocol mismatch alerts on ports 21 and 25.
 
![Suricata alerts for the aggressive Nmap scan](./05-suricata-02-aggressive-scan-detected.png)
 
**Finding:** Suricata's default ruleset reliably detects reconnaissance activity, and detection volume and breadth scale with how intrusive the scan technique is.
 
### Test 2: Exploitation Detection Gap
 
Re-ran the Project 2 exploit against the same target:
```
msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf > set RHOSTS 192.168.56.102
msf > set LHOST 192.168.56.103
msf > run
```
Result: the exploit succeeded and a Meterpreter session was opened, confirming full compromise of the target. However, **no new Suricata alerts were generated**, either at the moment of exploitation or during subsequent post-exploitation activity (`sysinfo`, shell commands).
 
![Exploit succeeded with no new Suricata alerts](./05-suricata-03-exploit-detection-gap.png)
 
**Finding:** the default Emerging Threats Open ruleset has no signature covering this specific exploit technique. Reconnaissance was detected; actual exploitation was not. This is a real and reproducible detection gap.
 
### Test 3: Closing the Gap with a Custom Rule
 
The vsftpd 2.3.4 backdoor has a distinctive fingerprint: once triggered, a malicious shell listens on **TCP port 6200**, a port with no legitimate service association. This made it a good candidate for a simple, targeted custom rule.
 
**Rule written** (`/etc/suricata/rules/local.rules`):
```
alert tcp any any -> any 6200 (msg:"Possible vsftpd 2.3.4 backdoor connection attempt"; flow:to_server; sid:1000001; rev:1;)
```
 
**Deployment steps:**
1. Added the rule to `local.rules` and referenced it in `suricata.yaml` under `rule-files`.
2. On restart, Suricata logged a warning: `No rule files match the pattern /var/lib/suricata/rules/local.rules`. The rule had been saved to `/etc/suricata/rules/`, but Suricata's `default-rule-path` pointed to `/var/lib/suricata/rules/`. Copying the file to the correct path resolved this.
3. Restarted Suricata cleanly with no warnings.
![Custom rule added to local.rules](./05-suricata-05-local-rule-added.png)
![Rule file linked in suricata.yaml](./05-suricata-06-rule-file-linked.png)
![Suricata restarted with no warnings](./05-suricata-07-suricata-restarted.png)
 
**Result:** re-running the exploit immediately produced alerts from the custom rule:
```
[**] [1:1000001:1] Possible vsftpd 2.3.4 backdoor connection attempt [**]
{TCP} 192.168.56.103:38475 -> 192.168.56.102:6200
```
 
![Custom rule triggered during the exploit](./05-suricata-08-custom-rule-triggered.png)
 
## MITRE ATT&CK Mapping
 
| Activity | Technique |
|----------|-----------|
| Nmap scans | T1046, Network Service Discovery |
| vsftpd 2.3.4 backdoor exploit | T1190, Exploit Public-Facing Application |
 
## Challenges and Troubleshooting
 
- **`suricata-update` not bundled by default** on this Kali build. It required a separate `apt install suricata-update`, which itself required temporarily switching the network adapter from host-only to NAT for internet access, then back again.
- **Terminal focus issues** in VirtualBox caused commands to silently fail to register on two occasions, initially misread as the scan/tail process being "stuck." Resolved by verifying process state independently with `pgrep` before assuming a hang.
- **Rule path mismatch**, the most instructive issue: a rule can be syntactically correct and correctly referenced in `rule-files`, but still silently ignored if it isn't in the directory Suricata's `default-rule-path` actually points to. Diagnosed by reading the startup warning output carefully rather than assuming the rule had failed outright.
## Key Takeaways
 
- Installing an IDS is not the same as having working detection. Default rulesets have real, identifiable gaps.
- Reconnaissance and exploitation traffic look very different at the network level, and require different detection logic.
- Writing a targeted custom rule based on an exploit's known behaviour (a distinctive port, in this case) is a practical, learnable way to close a specific gap.
- Configuration issues (like a rule-path mismatch) can look identical to "the rule doesn't work." Methodical troubleshooting (checking logs, verifying each config layer) matters as much as the detection logic itself.
## Limitations
 
- The custom rule alerts on any TCP connection to port 6200, so it detects the backdoor shell *after* the exploit has triggered rather than the trigger itself, and it would also flag any legitimate service that happened to use that port.
- Detection is signature-based and was tested only against known lab attacks, not a wider set of traffic.
## Future Improvements
 
- Add a rule that matches the backdoor trigger on FTP (port 21), so the attempt is detected before the shell opens.
- Test detection coverage against the web application attacks from Project 3 (Juice Shop SQLi, XSS, IDOR) to compare network-layer vs. application-layer detection.
- Expand custom rule coverage to other exploit techniques used in Project 2.
- Revisit a full SIEM (e.g. Wazuh) for log correlation and longer-term alerting at scale, once lab resources allow.
