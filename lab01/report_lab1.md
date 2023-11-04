# Отчёт по лабораторной работе 1 "Установка CHR и Ansible, настройка VPN"
---
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2023/2024  
Group: K34212  
Author: Kilinich Vladislav Aleksandrovich  
Lab: Lab1  
Date of create: 04.11.2023  
Date of finished: - 
---
# Ход работы
---
Цель работы: развертывание виртуальной машины на базе платформы Yandex Cloud с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox.  

# Yandex cloud
Для создания VPN сервера была выбрана платформа cloud.yandex. Далее была создана ВМ с ОС ubuntu:
![image](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/1.PNG?raw=true)
Для подключения по ssh пишем
```
ssh ubuntu@84.201.177.192
```
Далее устанавливаем python3
```
sudo apt install python3-pip
```
И Ansible:
```
sudo pip3 install ansible
```
# Настройка VPN сервера 
---
Для создания сервера был выбран OpenVPN  Server. Прописываем следующие команды для установки ПО:  
```
apt update && apt -y install ca-certificates wget net-tools gnupg
wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repository.asc
echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repository.asc] http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt update && apt -y install openvpn-as
```  
После выполнения команд выводится сообщение с логином и паролем для подключения к OpenVPN Access Server по следующей ссылке: https://84.201.177.192:943/admin. Сразу указываем протокол TCP, и проверяем что IP-адрес указан верно.    
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/assets/63118851/0300353c-0afd-4246-a51e-699bd748bcb0" width="800" heidth = 700/>   
</p>  
Зарегестрировавшись в сервесе OpenVPN копируем ключ активации и вставляем его в CONFIGURATION -> Activation  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/3.3.PNG?raw=true" width="800" heidth = '700'/>  
</p>  

В настройках CONFIGURATION -> Advanced VPN отключаем TLS.  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/4.PNG?raw=true" width="800" heidth = '700'/>  
</p>  
  
Далее в User managment -> User Permissions создаем пользователя, а в User Profiles создаем новый профайл. После создания профайла скачивается файл с расширением ovpn.  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/5.PNG?raw=true" width="800" heidth = '700'/>  
</p>  

# Настройка CHR  
Скачиваем образ  .vdi CHR c официального сайта Mikrotik. Создаем виртуальную машину, используя Virtualbox. Настраиваем тип подключения - Сетевой мост.  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/6.PNG?raw=true" width="800" heidth = '700'/>  
</p>  



---
# Созданный тунель EoMPLS связал порты eth2 на R01.NY и eth4 на R01.SPB.
Для проверки связанности были пропингованы компьютеры:
![image](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/10.PNG?raw=true)
![image](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/11.PNG?raw=true)






