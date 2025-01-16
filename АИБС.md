## *Базовая настройка*

##### 1. Настройте имена всех сетевых устройств, виртуальных машин и серверов в соответствии с диаграммой
   Cisco: `Router (config)# hostname R1`
   Linux: `hostnamectl set-hostname <имя-устройства> || hostname <имя-устройства>`
   Windows: `hostnamectl set-hostname <имя-устройства> || sconfig`
##### 2. Сконфигурируйте доменные имена на сетевом оборудовании, используйте имя домена futuregadgetlab.com
   Cisco: `R1 (config)# ip domain-name futuregadgetlab.com`
##### 3. На всех серверах Linux и сетевом оборудовании создайте пользователя Admin с паролем 1@tuTTuru@1
###### 3.а Для сетевого оборудования пароль должен быть регистрочувствительным и хранится в виде результате хэш функции
   Cisco: `R1 (config)# username Admin privilege algorithm-type scrypt 1@tuTTuru@1`
###### 3.b На Linux серверах обеспечьте возможность использования sudo только для группы Admins, а также локального пользователя Admin. Отключите необходимость дополнительный аутентификации для использования sudo.
   Linux: 
   ```bash
sudo useradd -m -s /bin/bash -G Admins Admin
echo "Admin:1@tuTTuru@1" | sudo chpasswd
echo "Admin ALL=(ALL) NOPASSWD: ALL" | sudo tee etc/sudoers.d/admin
```
###### 3.a Ограничьте локальный вход пользователя root только пятым виртуальным терминалом.
   Lunix: `echo "tty5" | sudo tee /etc/securetty`
##### 4. В качестве пароля для входа в привилегированный режим используйте IBTKS
   Cisco: `R1 (config)# enable secret IBTKS`
##### 5. Организуйте удаленный доступ до всего сетевого оборудования, а также до серверов и виртуальных машин с ОС Linux по протоколу SSH v2. Разрешите доступ всем пользователям, включая пользователя root.
   Linux: 
```bash
   sudo apt install -y openssh-server
   sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
   sudo systemctl restart ssh
```
Cisco:
```bash
    ip ssh version 2
    line vty 0 15
    transport input ssh
    ```
##### 6. Отключите все неиспользуемые порты на сетевых устройствах.
   Cisco: 
   ```bash
   R1 (config)# interface range f0/2-24
   R1 (config)# shutdown
```

## *Конфигурация активного сетевого оборудования*
##### 1. Таблица VLAN на сетевых устройствах должна соответствовать:
   1.а VLAN 1010 - WIN
   1.b VLAN 1011 - LIN
   1.c VLAN 50 - Clients
   1.d VLAN 999 - Trunk
   1.e VLAN 6 - MGMT
   1.f VLAN 666 - ISP
   ```bash
   S1 (config)# vlan 1010
   S1 (config-vlan)# name WIN
   S1 (config-vlan)# exit
   S1 (config)# vlan 1011
   S1 (config-vlan)# name LIN
   S1 (config-vlan)# exit
   S1 (config)# vlan 50
   S1 (config-vlan)# name Clients
   S1 (config-vlan)# exit
   S1 (config)# vlan 999
   S1 (config-vlan)# name Trunk
   S1 (config-vlan)# exit
   S1 (config)# vlan 6
   S1 (config-vlan)# name MGMT
   S1 (config-vlan)# exit
   S1 (config)# vlan 666
   S1 (config-vlan)# name ISP
   S1 (config-vlan)# exit
```
##### 2. Настройте передачу тегированного трафика между коммутаторами. В качестве Native VLAN используйте VLAN 999.
   ```bash
   S1 (config)# int e0/0
   S1 (config-interface)# switchport mode trunk
   S1 (config-interface)# switchport trunk native vlan 999
   S1 (config-interface)# switchport trunk allowed vlan 1010,1011,50,999,6,666
   S1 (config-interface)# switchport nonegotiate(не обязательно)
   S1 (config-interface)# no shutdown
```
##### 3. Сконфигурируйте Management адреса на коммутаторах. В качестве подсети используйте 10.0.100.0/24, адреса сконфигурируйте на ваше усмотрение.
   ```bash
   S1 (config)# interface vlan 6
   S1 (config-if)# ip address 10.0.100.X 255.255.255.0
   S1 (config-if)# no shutdown
```
##### 4. Используйте адресацию основываясь на диаграммах.
##### 5. Настройте магистральные каналы в соответствии с диаграммой L2. Явно отключите протокол DTP.
   ```bash
   S1 (config)# interface range e0/1 - 2
   S1 (config-if-range)# channel-group 1 mode active
   S1 (config-if-range)# no shutdown
   S1 (config-if-range)# exit
   S1 (config)# interface Port-Channel1
   S1 (config-if)# switchport mode trunk
   S1 (config-if)# switchport trunk native vlan 999
   S1 (config-if)# switchport trunk allowed vlan 1010,1011,50,999,6,666
   S1 (config-if)# switchport nonegotiate
   S1 (config-if)# no shutdown
   S1 (config-if)# exit
```
##### 6. Сконфигурируйте портовые группы в соответствии с диаграммой L2.
###### 6.a SW3 должен быть активным для обоих портовых групп. Протокол динамического согласования на ваше усмотрение.
   ```bash
   conf t
! Портовая группа 1 (например, для связи с S1)
interface range GigabitEthernet0/1 - 2
 description EtherChannel to S1
 channel-group 1 mode active
 no shutdown
 exit

! Портовая группа 2 (например, для связи с S2)
interface range GigabitEthernet0/3 - 4
 description EtherChannel to S2
 channel-group 2 mode active
 no shutdown
 exit

! Настройка интерфейсов Port-Channel
interface Port-Channel1
 description Trunk to S1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1010,1011,50,999,6,666
 switchport nonegotiate
 no shutdown
 exit

interface Port-Channel2
 description Trunk to S2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1010,1011,50,999,6,666
 switchport nonegotiate
 no shutdown
 exit
```

```bash
Настройка S1 (похожая для S2)
conf t
interface range GigabitEthernet0/1 - 2
 description EtherChannel to SW3
 channel-group 1 mode passive
 no shutdown
 exit

interface Port-Channel1
 description Trunk to SW3
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1010,1011,50,999,6,666
 switchport nonegotiate
 no shutdown
 exit

```
###### 6.b Сконфигурируйте портовую группу между S1 и S2 без использования протоколов согласования.
   S1
   ```bash
   conf t
interface range GigabitEthernet0/3 - 4
 description EtherChannel to S2
 channel-group 2 mode on
 no shutdown
 exit

interface Port-Channel2
 description Trunk to S2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1010,1011,50,999,6,666
 switchport nonegotiate
 no shutdown
 exit
```
S2 
```bash
conf t
interface range GigabitEthernet0/3 - 4
 description EtherChannel to S1
 channel-group 2 mode on
 no shutdown
 exit

interface Port-Channel2
 description Trunk to S1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1010,1011,50,999,6,666
 switchport nonegotiate
 no shutdown
 exit

```
##### 7. Настройте протокол остовного древа для защиты от петель на канальном уровне.
###### 7.a Используйте реализацию протокола RapidSTP
   ```bash
conf t
spanning-tree mode rapid-pvst
exit

```
###### 7.b Расставьте приоритеты, чтобы порядок выбора корневого моста в следующем порядке S3 -> S1 -> S2
   S3
   ```bash
conf t
spanning-tree vlan 1 priority 4096
exit
```
   S1
```bash
conf t
spanning-tree vlan 1 priority 8192
exit
```
   S2
  ```bash 
conf t
spanning-tree vlan 1 priority 12288
exit
```
**Примечание**: Уровень приоритета должен быть кратным 4096, так как это требование стандарта STP.
###### 7.c Сконфигурируйте порты f0/4 и f0/1 коммутатора S3 таким образом, чтобы порты сразу переходили в состояние forwarding, не дожидаясь пересчета остовного дерева.
   ```bash
conf t
interface FastEthernet0/4
 description Fast Forwarding Port
 spanning-tree portfast
 exit

interface FastEthernet0/1
 description Fast Forwarding Port
 spanning-tree portfast
 exit
```
**Примечание**: Использование `spanning-tree portfast` рекомендуется только для конечных устройств. На trunk-портах это нужно делать с осторожностью.
Полная конфигурация для S3 (пример):
```bash
conf t
spanning-tree mode rapid-pvst

!Установка приоритета для корневого моста
spanning-tree vlan 1 priority 4096

!Настройка портов для быстрого forwarding
interface FastEthernet0/4
 description Fast Forwarding Port
 spanning-tree portfast
 exit
interface FastEthernet0/1
 description Fast Forwarding Port
 spanning-tree portfast
 exit
```
##### 8. Взаимодействие с ISP осуществляется через VLAN 50.
   Маршрутизатор
   ```bash
   conf t
interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.1.2 255.255.255.252
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 192.168.1.1
```
   Коммутатор
```bash
conf t
vlan 50
name ISP

interface GigabitEthernet0/1
 description Connection to ISP
 switchport mode access
 switchport access vlan 50
 no shutdown

interface GigabitEthernet0/2
 description Trunk to Router
 switchport mode trunk
 switchport trunk allowed vlan 50
 no shutdown
exit
```
##### 9. Для сетей WIN и LIN обеспечьте возможность выхода в Internet
   ```bash
conf t
! Интерфейсы VLAN
interface GigabitEthernet0/0.1010
 encapsulation dot1Q 1010
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/0.1011
 encapsulation dot1Q 1011
 ip address 192.168.11.1 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.1.2 255.255.255.252
 ip nat outside
 no shutdown

! Маршрут по умолчанию
ip route 0.0.0.0 0.0.0.0 192.168.1.1

! Настройка NAT
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.11.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0.50 overload
exit
```
##### 10. Обеспечьте маршрутизацию между внутренними сетями посредством протокола OSPF на маршрутизаторах R1 и R2
###### 10.a Анонсируйте все сети, необходимые для достижения полной связанности.
###### 10.b Соседство должно устанавливаться только через сеть R1R2, остальные порты необходимо перевести в пассивный режим.
R1
```bash
conf t
! Интерфейсы
interface GigabitEthernet0/0.1010
 encapsulation dot1Q 1010
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0.1011
 encapsulation dot1Q 1011
 ip address 192.168.11.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.1.1 255.255.255.252
 no shutdown

! Настройка OSPF
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.11.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/0.50
exit
```
R2
```bash
conf t
! Интерфейсы
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.1.2 255.255.255.252
 no shutdown

! Настройка OSPF
router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.3 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/0.50
exit
```
###### Примечания:
- **`192.168.1.0/30`**: Используется как пример адресации для связи между R1 и R2. Убедитесь, что эта сеть совпадает с вашей диаграммой.
- **Passive Interfaces**: Все интерфейсы, кроме интерфейса R1-R2, должны быть пассивными для предотвращения лишнего обмена OSPF-пакетами.

##### 11. Для обеспечения связанности между офисами, сконфигурируйте GRE туннель. Используйте адресацию в соответствии с диаграммой, защита туннеля не требуется. 
R1
```bash
conf t
interface Tunnel0
 ip address 192.168.100.1 255.255.255.252
 tunnel source 192.168.1.1
 tunnel destination 192.168.1.2
 no shutdown

router ospf 1
 network 192.168.100.0 0.0.0.3 area 0
exit
```
R2
```bash
conf t
interface Tunnel0
 ip address 192.168.100.2 255.255.255.252
 tunnel source 192.168.1.2
 tunnel destination 192.168.1.1
 no shutdown

router ospf 1
 network 192.168.100.0 0.0.0.3 area 0
exit
```

## Настройка серверов под управлением Linux
##### 1. На сервере MON созданы 5 дополнительных дисков по 100 МБ.
   Проверьте доступные диски - `lsblk`
   Установите mdadm:
   `sudo apt update` 
   `sudo apt install -y mdadm`
   ###### 1.а Объедините их в RAID 6;
   `sudo mdadm --create --verbose /dev/md0 --level=6 --raid-devices=5 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf`
   Проверьте статус массива:
   `cat /proc/mdstat` 
   `sudo mdadm --detail /dev/md0`
   Сохраните конфигурацию RAID:
   `sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf`
   `sudo update-initramfs -u`
   ###### 1.b Разметьте том, как ext4;
   `sudo mkfs.ext4 /dev/md0`
   Проверьте файловую систему: `sudo blkid /dev/md0`
   ###### 1.c Обеспечьте автоматическое монтирование тома в директорию /opt Монтирование должно производится автоматически при загрузке системы.
   Найдите UUID RAID:`sudo blkid /dev/md0`
   Откройте `/etc/fstab` для редактирования:`sudo nano /etc/fstab`
   Добавьте строку:`UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /opt ext4 defaults 0 0`
   Смонтируйте файловую систему:`sudo mount -a`
   Проверьте, что том смонтирован:`df -h /opt`
   
   Убедитесь, что RAID работает:`cat /proc/mdstat`
   Проверьте, что диск монтируется после перезагрузки:
   `sudo reboot`
   `df -h /opt`
##### 2. На сервере WEB настройте web-сервер Apache.
   ###### 2.a Сервер должен работать на порту 80 по протоколу http;
   Установите Apache: `sudo apt update sudo apt install -y apache2`
   Убедитесь, что Apache работает: `sudo systemctl enable apache2 sudo systemctl start apache2`
   Проверьте, что сервер слушает порт 80: `sudo netstat -tuln | grep :80`
   ###### 2.b В качестве стартовой страницы создайте документ index.html с содержимым:
   - Создайте файл `index.html` с указанным содержимым: `sudo nano /var/www/html/index.html`
- Вставьте следующий HTML-код:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <title>Лаборатория Гаджетов Будущего</title>
   </head>
   <body>
   <div id='header' style="height: 200px;">
   <a href="https://www.futuregadgetlab.com">Вас приветствует Лаборатория Гаджетов Будущего!</a>
   </div>
   <div id='content' style="height: 400px; font-size: 36pt; border: 5px solid #999; background-color: #B0E0E6;">
   <p>!ЛИНУКС!</p>
   </div>
   <div id='footer'>ИБТКС</div>
   </body>
   </html>
```
    Проверьте доступность страницы: `curl http://localhost`
    ###### 2.c Сайт должен быть доступен с узла Student по имени elearn.futuregadgetlab.com
    Редактирование файла VirtualHost: Создайте конфигурацию для `elearn.futuregadgetlab.com`: `sudo nano /etc/apache2/sites-available/elearn.conf`
    Добавьте следующий конфигурационный файл:
  ```bash
  <VirtualHost *:80> 
	  ServerName elearn.futuregadgetlab.com 
	  DocumentRoot /var/www/html 
	  <Directory /var/www/html> 
		  AllowOverride All 
		  Require all granted 
	  </Directory> 
	  ErrorLog ${APACHE_LOG_DIR}/elearn_error.log 
	  CustomLog ${APACHE_LOG_DIR}/elearn_access.log combined </VirtualHost>
```
	Активируйте сайт и перезапустите Apache: `sudo a2ensite elearn.conf sudo systemctl reload apache2`
**Настройка DNS**
	Убедитесь, что узел `elearn.futuregadgetlab.com` разрешается на IP-адрес сервера WEB.
	Если DNS используется: 
	Добавьте запись `A` для домена `elearn.futuregadgetlab.com` в ваш DNS-сервер. Если DNS не используется (тестовая среда):
	Добавьте строку в файл `/etc/hosts` на узле Student: `<WEB_SERVER_IP> elearn.futuregadgetlab.com`
##### 3. На сервере VPN создайте центр сертификации.
**Установка Easy-RSA**
	Установите Easy-RSA:`sudo apt update sudo apt install -y easy-rsa`
	Создайте директорию для центра сертификации:`sudo mkdir -p /etc/ca`
	Скопируйте скрипты Easy-RSA в эту директорию: `sudo cp -r /usr/share/easy-rsa/* /etc/ca/`
   3.a Имя центра должно быть Central CA, остальные параметры и атрибуты на Ваше усмотрение
	Перейдите в директорию `/etc/ca`: `cd /etc/ca`
	Инициализируйте инфраструктуру PKI:`sudo ./easyrsa init-pki`
	Создайте корневой сертификат для **Central CA**: `sudo ./easyrsa build-ca`
	    При запросе имени организации введите `Central CA`.
	    Для пароля ключа используйте любой подходящий пароль.
   3.b Корень СА должен быть в папка /etc/ca/
   Переместите корневой CA (если требуется): `sudo mv /etc/ca/pki/ca.crt /etc/ca/`

##### 4. На сервере VPN установите и настройте OpenVPN:
###### 4.a Используйте сеть 10.0.10.0/24 для клиентов;
###### 4.b Используйте сертификаты выданные центром сертификации на сервере VPN;
###### 4.c Клиенты должны быть автоматически настроены на использование внутренних DNS серверов компании.
   Установите OpenVPN и необходимые пакеты:`sudo apt update sudo apt install -y openvpn easy-rsa`
**Настройка сертификатов**
	Перейдите в директорию CA:`cd /etc/ca`
	Создайте серверный ключ и запрос сертификата:`sudo ./easyrsa gen-req server nopass`
	Подпишите запрос сертификата с использованием CA:`sudo ./easyrsa sign-req server server`
	Создайте ключ Diffie-Hellman:`sudo ./easyrsa gen-dh`
	Создайте файл HMAC (для защиты от DoS-атак):`sudo openvpn --genkey --secret /etc/ca/ta.key`
	Переместите необходимые файлы в `/etc/openvpn`:
   `sudo cp /etc/ca/pki/issued/server.crt /etc/openvpn/`
   `sudo cp /etc/ca/pki/private/server.key /etc/openvpn/` 
   `sudo cp /etc/ca/pki/ca.crt /etc/openvpn/` 
   `sudo cp /etc/ca/pki/dh.pem /etc/openvpn/` 
   `sudo cp /etc/ca/ta.key /etc/openvpn/`
   Создайте конфигурационный файл сервера:`sudo nano /etc/openvpn/server.conf`
	   Добавьте следующие строки:
```bash
   port 1194
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
tls-auth /etc/openvpn/ta.key 0
server 10.0.10.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS <Internal_DNS_IP>"  # Замените <Internal_DNS_IP> на IP вашего DNS-сервера
keepalive 10 120
persist-key
persist-tun
status /var/log/openvpn-status.log
log-append /var/log/openvpn.log
verb 3 
```
Убедитесь, что файл конфигурации сохранен.
**Настройка маршрутизации**
	Включите маршрутизацию на сервере:`echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf sudo sysctl -p`
	Настройте NAT для клиентов OpenVPN:`sudo iptables -t nat -A POSTROUTING -s 10.0.10.0/24 -o eth0 -j MASQUERADE`
	Убедитесь, что правило сохранено:`sudo apt install -y iptables-persistent sudo netfilter-persistent save`
**Запуск и проверка OpenVPN**
	Запустите OpenVPN:`sudo systemctl start openvpn@server`
	Убедитесь, что OpenVPN запущен:`sudo systemctl status openvpn@server`
	Проверьте, что сервер слушает порт 1194:`sudo ss -tuln | grep 1194`
**Настройка клиента**
	Создайте клиентский ключ и запрос сертификата: `sudo ./easyrsa gen-req client1 nopass`
    Подпишите запрос сертификата: `sudo ./easyrsa sign-req client client1`
	 Создайте клиентский файл конфигурации:`sudo nano /etc/openvpn/client.ovpn`
    Добавьте в файл следующие строки:`client dev tun proto udp remote <VPN_Server_IP> 1194 resolv-retry infinite nobind persist-key persist-tun ca ca.crt cert client1.crt key client1.key tls-auth ta.key 1 remote-cert-tls server verb 3`
    Переместите файлы `ca.crt`, `client1.crt`, `client1.key`, и `ta.key` на клиентскую машину.

##### 5. Сконфигурируйте на всех LINUX серверах межсетевой экран при помощи IPTABLES:
   5.a Обеспечьте доступ только к необходимым для работы сервисов портам;
   5.b Все остальные соединения должны отбрасываться;
   5.c На ICMP сообщения все LINUX хосты должны отвечать icmp-host-prohibited.
   ```bash
# Очистка правил
sudo iptables -F
sudo iptables -X

# Разрешить локальные соединения
sudo iptables -A INPUT -i lo -j ACCEPT

# Разрешить установленные соединения
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Разрешить SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP и HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Отвечать на ICMP запросы с кодом icmp-host-prohibited
sudo iptables -A INPUT -p icmp -j REJECT --reject-with icmp-host-prohibited

# Отбрасывать все остальные соединения
sudo iptables -A INPUT -j DROP

# Сохранение правил
sudo netfilter-persistent save
```


#### Получаемые баллы за каждый модуль

##### **Модуль 1: Базовая настройка**
| №   | Критерий                                                                         | Баллы |
|-----|----------------------------------------------------------------------------------|-------|
| 1   | Настроены имена всех сетевых устройств, виртуальных машин и серверов             | 1,00  |
| 2   | Сконфигурированы доменные имена на сетевом оборудовании                          | 1,00  |
| 3   | Создан пользователь Admin                                                       | 0,40  |
| 4   | Настроена возможность использования sudo                                         | 0,40  |
| 5   | Локальный вход root ограничен                                                   | 0,20  |
| 6   | Для привилегированного режима используется пароль IBTKS                         | 1,00  |
| 7   | Организован удаленный доступ                                                    | 2,00  |
| 8   | Отключены неиспользуемые порты                                                  | 1,00  |
**Итог за модуль 1: 7,00 баллов**

---
##### **Модуль 2: Конфигурация активного сетевого оборудования**
| №   | Критерий                                                                         | Баллы |
|-----|----------------------------------------------------------------------------------|-------|
| 9   | VLAN 1010 – WIN                                                                 | 0,20  |
| 10  | VLAN 1011 – LIN                                                                 | 0,20  |
| 11  | VLAN 50 – Clients                                                               | 0,20  |
| 12  | VLAN 999 – Trunk                                                                | 0,20  |
| 13  | VLAN 6 – MGMT                                                                   | 0,20  |
| 14  | VLAN 666 – ISP                                                                  | 0,20  |
| 15  | Передача тегированного трафика (Native VLAN 999)                                | 0,80  |
| 16  | Management-адреса (10.0.100.0/24)                                               | 1,00  |
| 17  | Использована адресация по диаграмме                                             | 1,00  |
| 18  | Магистральные каналы (протокол DTP отключен)                                    | 1,00  |
| 19  | SW3 активен для портовых групп                                                  | 0,50  |
| 20  | Портовая группа между S1 и S2 без протоколов                                    | 0,50  |
| 21  | Протокол остовного дерева RapidSTP                                              | 0,20  |
| 22  | Расставлены приоритеты корневого моста S3 -> S1 -> S2                           | 0,40  |
| 23  | Порты f0/4 и f0/1 на S3 сразу переходят в forwarding                            | 0,40  |
| 24  | Взаимодействие с ISP через VLAN 50                                              | 1,00  |
| 25  | Выход в Интернет для WIN и LIN                                                  | 1,00  |
| 26  | Маршрутизация OSPF на R1 и R2                                                   | 0,50  |
| 27  | Соседство OSPF только через сеть R1-R2                                          | 0,50  |
| 28  | GRE туннель настроен                                                            | 1,00  |
**Итог за модуль 2: 12,00 баллов**

---
##### **Модуль 3: Настройка серверов под управлением Windows**
| №   | Критерий                                                                         | Баллы |
|-----|----------------------------------------------------------------------------------|-------|
| 29  | Настроен домен futuregadgetlab.com                                              | 0,50  |
| 30  | Машины FS, RDS и CLI введены в домен                                            | 0,50  |
| 31  | Служба DNS (A и PTR записи)                                                     | 1,00  |
| 32  | Организационные подразделения, группы и пользователи                            | 1,00  |
| 33  | Минимальная длина пароля для группы Admins                                      | 1,00  |
| 34  | RAID 5 из 4 дополнительных дисков (FS)                                          | 1,00  |
| 35  | Диск F: для общих каталогов, Z: для Admins                                      | 0,50  |
| 36  | Запрещен запуск файлов, квота 100 МБ                                            | 1,00  |
| 37  | Пользователи не видят чужие каталоги                                            | 0,50  |
| 38  | Общая папка для LabMembers (диск W:)                                            | 1,00  |
| 39  | Настроен IIS (сервер работает на порту 80)                                      | 0,50  |
| 40  | Создан index.html для IIS                                                      | 0,50  |
| 41  | В IE стартовая страница: https://www.futuregadgetlab.com                        | 1,00  |
| 42  | Настроена роль DHCP (5 минут аренды)                                            | 1,00  |
**Итог за модуль 3: 11,00 баллов**

---
##### **Модуль 4: Настройка серверов под управлением Linux**
| №   | Критерий                                                                         | Баллы |
|-----|----------------------------------------------------------------------------------|-------|
| 43  | На MON созданы 5 дисков по 100 МБ, объединены в RAID 6                          | 0,80  |
| 44  | Том размечен как ext4                                                           | 0,80  |
| 45  | Автоматическое монтирование в /opt                                              | 0,40  |
| 46  | Apache на WEB (порт 80)                                                         | 0,40  |
| 47  | Создан index.html для Apache                                                    | 0,80  |
| 48  | Сайт доступен по имени elearn.futuregadgetlab.com                               | 0,80  |
| 49  | Центр сертификации (Central CA)                                                 | 1,00  |
| 50  | Корень CA в /etc/ca/                                                            | 1,00  |
| 51  | Настроен OpenVPN (сеть 10.0.10.0/24)                                            | 1,00  |
| 52  | Используются сертификаты, выданные CA                                           | 1,00  |
| 53  | Автоматическая настройка DNS для клиентов                                       | 1,00  |
| 54  | IPTABLES: доступ только к необходимым портам                                   | 0,80  |
| 55  | Иные соединения отбрасываются                                                  | 0,80  |
| 56  | ICMP — icmp-host-prohibited                                                    | 0,40  |
**Итог за модуль 4: 10,00 баллов**

---
##### **Общий итог**
- **Модуль 1: 7,00 баллов**
- **Модуль 2: 12,00 баллов**
- **Модуль 3: 11,00 баллов**
- **Модуль 4: 10,00 баллов**

**Общий итог: 40,00 баллов**
