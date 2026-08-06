# Systemd Timer - System Check

Ez a feladat egy egyszerű systemd timer létrehozása, ami percenként lefuttat egy scriptet.

## 1. Script létrehozása

```bash
echo '#!/bin/bash' > /usr/local/bin/system-check.sh
echo 'date >> /var/log/system-check.log' >> /usr/local/bin/system-check.sh
chmod +x /usr/local/bin/system-check.sh
```

## 2. Service unit létrehozása

```bash
vi /etc/systemd/system/system-check.service
```

Tartalom:

```ini
[Unit]
Description=System Check Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system-check.sh
```

## 3. Timer unit létrehozása

```bash
vi /etc/systemd/system/system-check.timer
```

Tartalom:

```ini
[Unit]
Description=Run System Check every minute

[Timer]
OnCalendar=minutely
Unit=system-check.service

[Install]
WantedBy=timers.target
```

## 4. Aktiválás

```bash
systemctl daemon-reload
systemctl enable --now system-check.timer
```

## 5. Ellenőrzés

```bash
systemctl status system-check.timer
systemctl list-timers
cat /var/log/system-check.log
```

## Rövid összefoglaló

- `.service` fájl: mit kell futtatni
- `.timer` fájl: mikor kell futtatni
- `OnCalendar=minutely`: percenként fut
- `systemctl enable --now`: azonnal indul és reboot után is aktív marad
