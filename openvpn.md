Прежде чем начать настраивать OpenVpn, убедитетсь в настройке центра сертификации!

apt-get install openvpn 
systemctl status openvpn

cp ~/openvpn-ca/keys/ca.crt /etc/openvpn/
cp ~/openvpn-ca/keys/openvpn-server.crt /etc/openvpn/
cp ~/openvpn-ca/keys/openvpn-server.key /etc/openvpn/

cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
gzip -d /etc/openvpn/server.conf.gz


nano server.conf


Изменяем следующие строки:
server 10.0.1.0 255.255.255.0
ca ca.crt
cert openvpn-server.crt
key openvpn-server.key 
tls-auth ta.key 0
user nobody
group nogroup
push "route 192.168.1.0 255.255.255.0"
push "redirect-gateway def1 bypass-dhcp"



openssl dhparam -out dh2048.pem 2048
openvpn --genkey --secret ta.key

systemctl start openvpn@server

systemctl restart openvpn@server


nano /etc/sysctl.conf
#«net.ipv4.ip_forward=1

iptables -A INPUT/OUTPUT -p udp --dport 1194 -j ACCEPT
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

iptables -L -v -n


Настройка клиента

scp student0@192.168.1.114:~/openvpn-ca/keys/openvpn-client-1.crt
/etc/openvpn/openvpn-client-1.crt
scp student0@192.168.1.114:~/openvpn-ca/keys/openvpn-client-1.key
/etc/openvpn/openvpn-client-1.key
scp student0@192.168.1.114:~/openvpn-ca/keys/ca.crt /etc/openvpn/ca.crt
scp student0@192.168.1.114:/etc/openvpn/ta.key /etc/openvpn/ta.key


cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf
/etc/openvpn/

Переходим в каталог «/etc/openvpn/» и открываем файл «client.conf» для
редактирования командой «nano client.conf». Изменяем следующие строки:
remote 192.168.1.114 1194
ca ca.crt
cert openvpn-client-1.crt
key openvpn-client-1.key
user nobody
group nogroup


systemctl start openvpn@client
