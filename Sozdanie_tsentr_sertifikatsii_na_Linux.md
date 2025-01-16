apt-get install easy-rsa
sudo make-cadir /etc/ca/Central CA
cd /etc/ca/Central CA
nano vars
И задаём в нём значение переменных:
export KEY_COUNTRY="RUS"
export KEY_PROVINCE="Rostov region"
export KEY_CITY="Taganrog"
export KEY_ORG="ictis.sfedu"
export KEY_EMAIL="admin-ca@myhost.mydomain"
export KEY_OU="ibtks"
export KEY_NAME="EasyRSA"
export KEY_ALTNAMES="laba-server"

source vars #Вышла ошибка пиши нижнию команду
cp openssl-1.0.0.cnf openssl.cnf
./clean-all    #Очищаем CA
./build-ca    #Создаём корневой центр

Создание для сервера файлов закрытого ключа и сертификата.
./build-key-server openvpn-server  # При запуски команды нажимать enter и y (2 раза)
ls /keys  #Убедиться в создании сертификатов сервера


Создание для клиента файлов закрытого ключа и сертификата.
./build-key-server openvpn-client-1  # При запуски команды нажимать enter и y (2 раза)
ls /keys  #Убедиться в создании сертификатов клиента