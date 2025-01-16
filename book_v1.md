**Логи к виртуалкам**

cli deb - root root

gui deb - root test123

Test123 || test123



# **Нулевой вариант экзамена**

## 1. **Базовая настройка оборудования(без линукса)**


#### **1. Настройка имен всех устройств в соответствии с диаграммой**

Для задания имени каждого устройства выполните следующее:

На каждом устройстве выполните команды:  
(пример для маршрутизатора R1)

```bash
enable
configure terminal
hostname R1
exit
```

Для коммутаторов:

```bash
hostname S1
```

Для других устройств замените `hostname` на имя, указанное в диаграмме.

---

#### **2. Конфигурация доменного имени**

На каждом сетевом устройстве настройте доменное имя `futuregadgetlab.com`:

```bash
enable
configure terminal
ip domain-name futuregadgetlab.com
exit
```

---

#### **3. Создание пользователя Admin с хэшированным паролем**

1. **Генерация хэшированного пароля для пользователя Admin:**
    
    ```bash
    enable
    configure terminal
    username Admin privilege 15 secret 1@tuTTuru@1
    exit
    ```
    
    - **`privilege 15`** — дает полный доступ к командам.
    - **`secret`** — сохраняет пароль в зашифрованном виде.
2. Для всех устройств создайте пользователя Admin с этой конфигурацией.
    

---

#### **4. Установка пароля для привилегированного режима**

Чтобы установить пароль для привилегированного режима, выполните:

```bash
enable
configure terminal
enable secret IBTKS
exit
```

Пароль будет храниться в зашифрованном виде благодаря `enable secret`.

---

#### **5. Организация удалённого доступа через SSH (вторая версия)**

1. **Включите SSH на всех устройствах.** Убедитесь, что доменное имя уже задано (см. пункт 2). Сгенерируйте RSA-ключи:
    
    ```bash
    enable
    configure terminal
    crypto key generate rsa
    ```
    
    При запросе длины ключа укажите:
    
    ```bash
    How many bits in the modulus [512]: 2048
    ```
    
2. **Настройте SSH версии 2:**
    ```bash
    ip ssh version 2
    ```
    
3. **Настройка VTY-линий для SSH-доступа:**
    ```bash
    line vty 0 15
    transport input ssh
    login local
    exit
    ```
    
    - Это обеспечит доступ для всех пользователей, включая `root`.

---

#### **6. Отключение неиспользуемых портов**

1. **Определите список активных интерфейсов:** Используйте команду:
    
    ```bash
    show ip interface brief
    ```
    
    Она покажет все интерфейсы и их состояние (`up/down`).
    
2. **Отключите неиспользуемые порты:** Для каждого неиспользуемого порта выполните команды:
    
    ```bash
    interface <порт>
    shutdown
    exit
    ```
    
    Например, если порт `GigabitEthernet0/2` не используется:
    
    ```bash
    interface GigabitEthernet0/2
    shutdown
    exit
    ```
    
3. **Повторите процедуру для всех неиспользуемых портов.**
    

---

### Итоговый пример конфигурации для R1:

```bash
enable
configure terminal

! Настройка имени
hostname R1

! Настройка доменного имени
ip domain-name futuregadgetlab.com

! Создание пользователя Admin
username Admin privilege 15 secret 1@tuTTuru@1

! Установка привилегированного пароля
enable secret IBTKS

! Настройка SSH
crypto key generate rsa
ip ssh version 2
line vty 0 15
transport input ssh
login local

! Отключение неиспользуемых портов
interface GigabitEthernet0/2
shutdown
exit

! Сохранение конфигурации
end
write memory
```

Аналогично выполните конфигурацию для всех сетевых устройств (R2, S1, S2, S3 и т.д.).