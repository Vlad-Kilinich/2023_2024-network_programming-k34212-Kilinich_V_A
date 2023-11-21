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
Цель работы: С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

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
# Запуск playbook и проверка результатов    
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
  

---
# Вывод
В результате выполнения лабораторной работы было выполнено развертывание виртуальной машины на платформе Yandex Cloud. Также установлен CHR в VirtualBox и настроен VPN туннель между VPN server и CHR.

