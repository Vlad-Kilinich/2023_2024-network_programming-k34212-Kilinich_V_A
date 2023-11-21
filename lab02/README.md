# Отчёт по лабораторной работе 2 "Развертывание дополнительного CHR, первый сценарий Ansible"
---
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2023/2024  
Group: K34212  
Author: Kilinich Vladislav Aleksandrovich  
Lab: Lab2  
Date of create: 21.11.2023  
Date of finished: - 
---
# Ход работы
---
Цель работы:  
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

# Развертывание 2 CHR  
Чтобы добавить новый образ VDI той же версии необходимо было поменять UUID образа  
```  
VBoxManage internalcommands sethduuid "/home/user/VirtualBox /etc/CHR-11_4.vdi"  
```  
Подключаем его к OpenVPN server как в 1 лабораторной работе.  

# Создание hosts и playbook

Файл hosts. Указываем адреса chr в группе chrs, также указываем rid (router_id) для дальнейшей настройки OSPF.  
Указываем данные для подключения к chr по ssh.  
```  
[chrs]
chr_1 ansible_host=172.27.240.20 rid=10.10.10.1
chr_2 ansible_host=172.27.240.21 rid=10.10.10.2

[chrs:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=community.routeros.routeros
ansible_user=admin
ansible_ssh_pass=123
```

При создании Ansible Playbook указываем команды для создания пользователя, найтроки NTP client, а также OSPF.  
Также собираем инсформацию об интерфейсах, чтобы вывести на экран при запуске playbook.  

```
- name:  chr_conf
  hosts: chrs
  tasks:
    - name: user_add
      routeros_command:
        commands:
          - /user add name=chr_user group=read password=123

    - name: ntp_client
      routeros_command:
        commands:
          - /system ntp client set enabled=yes server=0.ru.pool.ntp.org

    - name: routing_ospf
      routeros_command:
        commands:
          - /interface bridge add name=lo
          - /ip address add address={{ rid }}/32 interface=lo
          - /routing ospf instance add name=ospf-1 version=2 router-id={{ rid }}
          - /routing ospf area add instance=ospf-1 name=backbone area-id=0.0.0.0
          - /routing ospf interface-template add network=0.0.0.0/0 area=backbone

    - name: facts_
      routeros_facts:
        gather_subset:
          - interfaces
      register: output_ospf

    - name: debug_output
      debug:
        var: "output_ospf"
```
# Запуск playbook и вывод результатов    
Перед запуском для устранения ошибки подключения создаем и добавляем ssh-ключи на оба роутера.
После запуска файла получаем следующий результат:  

<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/assets/63118851/dfcf35b6-be6d-4d6b-be8f-acec430261ee" width="800" heidth = 700>   
</p>  
  
<p align="center">  
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/assets/63118851/5a8c149d-37a7-46f8-b664-09b1d7f38164" width="800" heidth = '700'>  
</p>  
 
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/assets/63118851/ec4a3db3-78b2-4b1f-857d-cc4223fb05d4" width="800" >  
</p>  
  
# Проверка CHR и связанности   

Вводим команду export на каждом CHR:  
Полная конфигурация файлов: [chr1](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/chr1_config.txt), [chr2](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/chr2_config.txt).  

<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/images/4.jpg?raw=true" width="400" heidth = '350'>  
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/images/3.jpg?raw=true" width="460" heidth = '350'>  
</p>  

NTP client print на одном из CHR:  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/images/6.jpg?raw=true" width="600" heidth = '500'>  
</p>

Результаты пингов, проверки локальной связности:  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/assets/63118851/7448e018-c2f1-41d9-9348-98dc98a903e3" width="600" heidth = '500'>  
</p>

# Схема сети   
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab02/images/12.drawio.png?raw=true" width="400" heidth = '300'>  
</p>  

---  
# Вывод
В ходе лабораторной работы были настроены CHR с помощью ansible. Были созданы два файла: inventory-файл и playbook. На роутерах были настроены: логин/пароль для нового пользователя, NTP client, ospf с указанием router id, а также были собраны данные настройки роутеров.

