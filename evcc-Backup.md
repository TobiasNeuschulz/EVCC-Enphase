# evcc Backup Script for evcc.yaml and evcc.db

Das folgende Script sichert die evcc.yaml und die evcc-Datenbank in den Home-Folder des Users "admin" und packt die Dateien in eine Datei mit dem aktuellen Datum.
Ersetzt "admin" durch euren Benutzernamen.

evccbackup.sh
```bash

sleep 3
sudo cp /etc/evcc.yaml /home/admin/evcc.yaml.bak
sleep 3
sudo cp /var/lib/evcc/evcc.db /home/admin/evcc.db.bak
sleep 3
tar cPfvz evcc-$(date +%Y-%m-%d).tar.gz /home/admin/evcc.yaml.bak /home/admin/evcc.db.bak
```
