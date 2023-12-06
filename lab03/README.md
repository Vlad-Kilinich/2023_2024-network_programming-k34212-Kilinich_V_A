# Отчёт по лабораторной работе 3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"
---
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2023/2024  
Group: K34212  
Author: Kilinich Vladislav Aleksandrovich  
Lab: Lab3  
Date of create: 06.12.2023  
Date of finished: -
---
# Ход работы
---
Цель работы:  
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

###Ход работы  
Для того, чтобы поднять Netbox сначала был установлен и настроен PostgreSQL:  
```
sudo apt install postgresql libpq-dev -y
```
Создание БД и юзера:
```
CREATE DATABASE netbox;
CREATE USER netbox WITH ENCRYPTED password '123';
GRANT ALL PRIVILEGES ON DATABASE netbox to netbox;
```
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab03/images/1.jpg?raw=true">  
</p>   

Далее устанавливаемн Redis с помощью команды:
```
sudo apt install -y redis-server
```

Установка и создание каталога NetBox:
```
sudo apt install python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev git -y
sudo pip3 install --upgrade pip  

sudo mkdir -p /opt/netbox/ && cd /opt/netbox/
sudo git clone -b master https://github.com/netbox-community/netbox.git .
```
Создание юзера netbox и настройка:  
```
sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
```  

Копируем конфигурационный файл и Генерируем ключ:  
```
sudo cp configuration.example.py configuration.py
sudo ln -s /usr/bin/python3 /usr/bin/python
sudo /opt/netbox/netbox/generate_secret_key.py
```

Далее настроим конфигурационный файл:  
```
ALLOWED_HOSTS = ['*']
DATABASE = {
    'NAME': 'netbox',               
    'USER': 'netbox',               
    'PASSWORD': '123', 
    'HOST': 'localhost',           
    'PORT': '',                     
    'CONN_MAX_AGE': 300,          
}

SECRET_KEY = 'ключ'
```

Для дальнейшей настройки были выполнены следующие команды:    
Создание виртуальной среды и настройка пакетов, также создаем юзера, чтобы заходить через него в NetBox  
```
sudo /opt/netbox/upgrade.sh
source /opt/netbox/venv/bin/activate
cd /opt/netbox/netbox
python3 manage.py createsuperuser  
```

Установка Gunicorn  
```
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
sudo cp /opt/netbox/contrib/*.service /etc/systemd/system/
sudo systemctl daemon-reload
```

Start netbox и проверка:  
```
sudo systemctl start netbox netbox-rq
sudo systemctl enable netbox netbox-rq
```
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab03/images/2.jpg?raw=true">  
</p>  

Также был настроен веб-сервер Nginx для доступа к Netbox через браузер:  
```
sudo apt install -y nginx
sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
```
---  
Настроить Netbox можно в браузере по указанному адресу и номеру порта, также необходимо указать имя созданного суперпользователя и пароль.  
В веб-интерфейсе был создан сайт, мануфактура, тип устройства, функция устройства, а далее само устройство – chr1 и CHR2. Для указания IP-адресов устройств необходимо было создать интерфейсы и IP-адреса. Занесенные в Netbox устройства:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/Netbox_devices.jpeg)  
Далее для работы с Netbox через Ansible нужно было устонавить модуль netbox.netbox командой:
```
ansible-galaxy collection install netbox.netbox
```
Для сохранения всех данных из Netbox в отдельный файл был создан inventory-файл, в котром был указан плагин, адрес netbox, токен (предварительно создвнный через веб-интерфейс):
```
plugin: netbox.netbox.nb_inventory
api_endpoint: http://130.193.42.42:8080/
token: созданный_токен
validate_certs: False
config_context: False
interfaces: True
```
Вся информация была сохранена в файл nb_inventory.yml с помощью команды:
```
ansible-inventory -v --list -y -i netbox_inventory.yml > nb_inventory.yml
```
Далее был написан сценарий для настройки CHR на основе данных из Netbox. Для этого полученный файл был изменен: в него добавлены переменные для подключения к устройствам из файла прошлой лабораторной работы, а также группа роутеров была названа devices вместо указанного по умолчанию ungrouped. Измененный файл: [inventory.yml](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/nb_inventory.yml).  
В playbook были прописаны команды по изменению имени устройства на имя, указанное в Netbox,  а также по добавлению IP-адреса на устройство (адрес, выданный VPN):
```
- name: Configuration
  hosts: devices
  tasks:
    - name: Set Name
      community.routeros.command:
        commands:
          - /system identity set name="{{interfaces[0].device.name}}"
    - name: Set IP-address
      community.routeros.command:
        commands:
        - /interface bridge add name="{{interfaces[1].display}}"
        - /ip address add address="{{interfaces[1].ip_addresses[0].address}}" interface="{{interfaces[1].display}}"
```
Обе таски были успешно выполнены, как можно увидеть ниже, имена устройств изменились (раньше они назывались MikroTik), а также добавились адреса на новый созданный интерфейс:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/chr1.jpg) ![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/CHR2.jpg)  
Далее playbook был изменен для того, чтобы собрать серийный номер каждого устройства и внести его в Netbox:  
```
- name: Serial Numbers
  hosts: devices
  tasks:
    - name: Serial Number
      community.routeros.command:
        commands:
          - /system license print
      register: license
    - name: Add Serial Number
      netbox_device:
        netbox_url: http://130.193.42.42:8080/
        netbox_token: созданный_токен
        data:
          name: "{{interfaces[0].device.name}}"
          serial: "{{license.stdout_lines[0][0].split(' ').1}}"
        state: present
        validate_certs: False
```
Для выполнения плэйбука потребовалось установить дополнительный модуль python3-pynetbox. После этого таски успешно выполнились. В веб-интерфейсе Netbox можно увидеть добавленный серийный номер для каждого CHR:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/chr1_sn.jpg) ![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/CHR2_sn.jpg)  
### Вывод  
В ходе лабораторной работы была собрана информация об устройствах и сохранена в отдельном файле с помощью Ansible и Netbox, а также была произведена настройка CHR на основе собранных данных.












# Схема сети   
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/images/12.drawio.png?raw=true" width="400" heidth = '300'>  
</p>  

---  
# Вывод
В ходе лабораторной работы были настроены CHR с помощью ansible. Были созданы два файла: inventory-файл и playbook. На роутерах были настроены: логин/пароль для нового пользователя, NTP client, ospf с указанием router id, а также были собраны данные настройки роутеров.
