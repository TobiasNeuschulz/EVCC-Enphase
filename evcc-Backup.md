
sleep 3
sudo cp /etc/evcc.yaml /home/admin/evcc.yaml.bak
sleep 3
sudo cp /var/lib/evcc/evcc.db /home/admin/evcc.db.bak
sleep 3
tar cPfvz evcc-$(date +%Y-%m-%d).tar.gz /home/admin/evcc.yaml.bak /home/admin/evcc.db.bak
