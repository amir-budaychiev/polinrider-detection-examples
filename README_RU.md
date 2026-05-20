# Примеры проверки PolinRider

English version: [README.md](README.md)

В этом репозитории лежат два легких Bash-скрипта для первичной проверки и подробное описание реального инцидента с PolinRider-like компрометацией GitHub supply chain.

Подробная история:

- [Описание на русском](DESCRIPTION_RU.md)
- [Описание на английском](DESCRIPTION.md)

## Что делают скрипты

`malware_local_scanner.sh` сканирует директории проектов и ищет PolinRider-like признаки:

- известные сигнатуры обфусцированного JavaScript payload;
- подозрительные `.bat`, `.cmd`, `.ps1`, `.vbs` файлы;
- batch-файлы, скрытые через `.gitignore`;
- слишком длинные строки в типичных JavaScript config-файлах.

`malware_log_scanner.sh` сканирует читаемые логи или экспортированные текстовые артефакты и ищет подозрительную активность:

- `temp_auto_push.bat`, `temp_interactive_push.bat`, `config.bat`;
- команды `git commit --amend` и force-push;
- install/build команды рядом с подозрительными config-файлами;
- известные JavaScript payload-сигнатуры.

Эти скрипты не являются антивирусом и не доказывают, что машина чистая. Это инструменты первичного триажа: если есть finding, нужно остановиться и разобраться.

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

Если часть логов недоступна для чтения, повторите только log scan через `sudo`:

```bash
sudo ./malware_log_scanner.sh /var/log/auth.log /var/log/syslog
```

## Windows

Используйте Git Bash или WSL.

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

Для данных Windows Event Viewer сначала экспортируйте подозрительные события в текстовый или CSV-файл, а потом передайте этот файл или папку в `malware_log_scanner.sh`.

## Отчеты и exit codes

Локальный сканер создает отчеты только если что-то нашел:

- `polinrider_infected_files.txt`
- `polinrider_suspicious_findings.txt`

Log scanner создает:

- `polinrider_log_hits.txt`

Exit codes:

- `0`: findings нет;
- `1`: есть suspicious findings или suspicious log hits;
- `2`: локальный сканер нашел malware-like payload-сигнатуры.

Можно выбрать, куда сохранять отчеты:

```bash
./malware_local_scanner.sh /path/to/projects --output-dir ./scan_reports
./malware_log_scanner.sh /path/to/logs --output-dir ./scan_reports
```

## Если сканер что-то нашел

Не удаляйте файлы сразу. Сначала сохраните доказательства: скриншоты, commit hashes, diff, время, GitHub-уведомления и report-файлы.

Не запускайте `npm install`, `yarn install`, build или dev server в подозрительном репозитории, пока не поймете масштаб проблемы. Считайте доступные секреты потенциально скомпрометированными и ротируйте GitHub tokens, SSH keys, deploy keys, cloud/API/payment secrets и другие credentials, которые могли быть доступны с зараженной машины.
