sudo apt install bind9

sudo nano /etc/bind/db.local
Привести содержимое к такому виду:
;
; BIND data file for local loopback interface
;
$TTL	604800
elearn.futuregadgetlab.com.	IN	SOA	ns1.elearn.futuregadgetlab.com.	root.elearn.futuregadgetlab.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
elearn.futuregadgetlab.com.	IN	NS	ns1.elearn.futuregadgetlab.com.
elearn.futuregadgetlab.com.	IN	MX	10	mail.elearn.futuregadgetlab.com.
elearn.futuregadgetlab.com.	IN	A	192.168.1.122
ns1	IN	A	192.168.1.122
www	IN	CNAME	elearn.futuregadgetlab.com.
mail	IN	A	192.168.1.122




sudo nano /etc/bind/db.255 
;
; BIND reverse data file for broadcast zone
;
$TTL	604800
1.168.192.in-addr.arpa.	IN	SOA	ns1.elearn.futuregadgetlab.com.	root.elearn.futuregadgetlab.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
1.168.192.in-addr.arpa.	IN	NS	ns1.elearn.futuregadgetlab.com.
122.1.168.192.in-addr.arpa.	IN	PTR	elearn.futuregadgetlab.com.

sudo nano /etc/bind/named.conf.local
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	forwarders { 8.8.8.8; };
	listen-on port 53 { 192.168.1.0/24; };
	allow-query { 192.168.1.0/24; };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
};




sudo nano /etc/bind/named.conf.local
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";



zone "elearn.futuregadgetlab.com" {
type master;
file "/etc/bind/db.local";
};


zone "1.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.255";
};


sudo systemctl restart bind9 # Перезапуск

sudo apt install apache2   # Установка apache2
sudo nano /var/www/html/index.html   # Создание index.html
cd /etс/apache2/sites-avaliable/
sudo cp 000.default.conf new.site.conf   # Создание копии

sudo nano new.site.conf

Добавить/Изменить следующее
ServerAdmin webmaster@localhost
ServerName new.elearn.futuregadgetlab.com
ServerAlias www.elearn.futuregadgetlab.com
DocumentRoot /var/www/html/


a2ensite new.site.conf
sudo a2dissite 000-default.conf

sudo nano /etc/apache2/apache2.conf
и в конце файла добавить
ServerName 127.0.0.1

sudo systemctl restart apache2.service




Настройка DHCP
default-lease-time 600;
max-lease-time 7200;
subnet 10.0.10.0 netmask 255.255.255.0 {
range 10.0.10.200 10.0.10.250;
option routers 10.0.10.1;
option domain-name-servers 10.0.10.1;
option domain-name "prudnikov.ru";
}










