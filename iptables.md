sudo nano /etc/sysctl.conf  #Перейти в каталог
#net.ipv4.ip_forward = 1 # Раскоментировать

sudo nano /etc/network/if-up.d/forwarding
#!/bin/bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

chmod +x /etc/network/if-up.d/forwarding  #Сделать исполняемым

Перезапустить виртуальные машины SRV и CLI



Запрет портов:
sudo nano /etc/network/if-up.d/firewall
#!/bin/bash
iptables -A FORWARD -p tcp --dport 80 -j DROP  #Дропается http
iptables -A FORWARD -p tcp --dport 443 -d vk.com -j DROP  # Дропается htpps
iptables -A FORWARD -p tcp --dport 22 -j ACCEPT   #Accept ssh

chmod +x /etc/network/if-up.d/firewall  #Сделать исполняемым

