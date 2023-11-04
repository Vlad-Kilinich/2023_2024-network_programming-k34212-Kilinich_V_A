# Отчёт по лабораторной работе 1 "Установка CHR и Ansible, настройка VPN""
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

1) Для создания VPN сервера была выбрана платформа cloud.yandex. Далее была создана ВМ с ОС ubuntu:
![avatar](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab01/images/1.PNG?raw=true)
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

---
# Таблицы MPLS маршрутов на всех роутерах
---
1. RO1.NY
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/NY_MPLS.PNG?raw=true)
2. RO1.LND
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/LND_MPLS.PNG?raw=true)
3. RO1.HKL
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/hki_mpls.PNG?raw=true)
4. RO1.SPB
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/hki_mpls.PNG?raw=true)
5. RO1.MSK
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/msk_mpls.PNG?raw=true)
6. RO1.LBN
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/LBN_MPLS.PNG?raw=true)
---
# Созданный тунель EoMPLS связал порты eth2 на R01.NY и eth4 на R01.SPB.
Для проверки связанности были пропингованы компьютеры:
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/10.PNG?raw=true)
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/11.PNG?raw=true)






