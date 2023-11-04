# Отчёт по лабораторной работе 3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS""
---
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)  
Year: 2022/2023  
Group: K33212  
Author: Kilinich Vladislav Aleksandrovich  
Lab: Lab3  
Date of create: 12.12.2022  
Date of finished: 16.12.2022  
---
# Ход работы
---
1) Развертывание контейнера:
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/Снимок.PNG?raw=true)
# Найстройка всех устройств
2) Конфигурация настройки OSPF, MPLS и EoMPLS RO1.NY  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/1.PNG?raw=true)  
3) Конфигурация настройки OSPF, MPLS RO1.LND  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/2.PNG?raw=true)  
4) Конфигурация настройки OSPF, MPLS RO1.HKL
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/4.PNG?raw=true)
5) Конфигурация настройки OSPF, MPLS и EoMPLS RO1.SPB  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/SPB.PNG?raw=true)  
6) Конфигурация настройки OSPF, MPLS RO1.MSK  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/5.PNG?raw=true)  
7) Конфигурация настройки OSPF, MPLS RO1.LBN  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/3.PNG?raw=true)  
8) Добавление интерфейса в SGI Prism и PC1  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/8.PNG?raw=true)  
![avatar](https://github.com/Vladkilinichh/2022_2023-introduction_in_routing-k33212-Kilinich-Vladislav/blob/main/lab3/images/9.PNG?raw=true)  
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






