# PolinRider detection examples

Русская версия: [README_RU.md](README_RU.md)

This repository contains two lightweight Bash scanners and a real incident write-up about a PolinRider-like GitHub supply-chain compromise.

Long-form story:

- [English description](DESCRIPTION.md)
- [Russian description](DESCRIPTION_RU.md)

## What the scripts do

`malware_local_scanner.sh` scans project directories for PolinRider-like indicators:

- known obfuscated JavaScript payload signatures;
- suspicious `.bat`, `.cmd`, `.ps1`, `.vbs` files;
- batch files hidden through `.gitignore`;
- unusually long lines in common JavaScript config files.

`malware_log_scanner.sh` scans readable logs or exported text artifacts for suspicious activity:

- `temp_auto_push.bat`, `temp_interactive_push.bat`, `config.bat`;
- `git commit --amend` and force-push commands;
- install/build commands near suspicious config files;
- known JavaScript payload signatures.

These scripts are not an antivirus and do not prove that a machine is clean. They are first-pass triage tools: a hit means "stop and investigate".

## macOS

```bash
chmod +x malware_local_scanner.sh malware_log_scanner.sh

./malware_local_scanner.sh /path/to/projects
./malware_log_scanner.sh ~/Library/Logs /path/to/app/log
```

## Ubuntu

```bash
chmod +x malware_local_scanner.sh malware_log_scanner.sh

./malware_local_scanner.sh /home/you/projects
./malware_log_scanner.sh /var/log/auth.log /var/log/syslog
```

If some logs are not readable, rerun only the log scan with `sudo`:

```bash
sudo ./malware_log_scanner.sh /var/log/auth.log /var/log/syslog
```

## Windows

Use Git Bash or WSL.

Git Bash:

```bash
bash malware_local_scanner.sh "/c/Users/you/projects"
bash malware_log_scanner.sh "/c/Users/you/AppData/Local" "/c/Users/you/AppData/Roaming"
```

WSL:

```bash
bash malware_local_scanner.sh /mnt/c/Users/you/projects
bash malware_log_scanner.sh /mnt/c/Users/you/AppData/Local /mnt/c/Users/you/AppData/Roaming
```

For Windows Event Viewer data, export suspicious events to text/CSV first, then pass the exported file or folder to `malware_log_scanner.sh`.

## Reports and exit codes

The local scanner writes reports only when it finds something:

- `polinrider_infected_files.txt`
- `polinrider_suspicious_findings.txt`

The log scanner writes:

- `polinrider_log_hits.txt`

Exit codes:

- `0`: no findings;
- `1`: suspicious findings or suspicious log hits;
- `2`: malware-like payload signatures found by the local scanner.

You can choose where reports are saved:

```bash
./malware_local_scanner.sh /path/to/projects --output-dir ./scan_reports
./malware_log_scanner.sh /path/to/logs --output-dir ./scan_reports
```

## If the scanner finds something

Do not immediately delete files. First preserve evidence: screenshots, commit hashes, diffs, timestamps, GitHub notifications, and report files.

Stop running `npm install`, `yarn install`, builds, or dev servers in the suspicious repository until you understand the scope. Treat accessible secrets as potentially compromised and rotate GitHub tokens, SSH keys, deploy keys, cloud/API/payment secrets, and other credentials that may have been available from the affected machine.
