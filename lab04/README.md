# Отчёт по лабораторной работе 4 "Базовая 'коммутация' и туннелирование используя язык программирования P4"
---
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2023/2024  
Group: K34212  
Author: Kilinich Vladislav Aleksandrovich  
Lab: Lab4  
Date of create: 06.12.2023  
Date of finished: -
---

Цель работы:  
Изучить синтаксис языка программирования P4 и выполнить 2 задания обучающих задания от Open network foundation для ознакомления на практике с P4.

### Ход работы  
Клонируем репозиторий https://github.com/p4lang/tutorials. Далее в папке vm-ubuntu-20.04 была развернута тестовая среда командой ```vagrant up```. В результате установки была создана ВМ с двумя аккаунтами vagrant и p4, заходим на p4:  

### Задание Implementing Basic Forwarding  
В первой части работы нужно было написать программу на P4, которая реализует базовую переадресацию с использованием следующей топологии:  
<p align="center">
<img src="https://github.com/p4lang/tutorials/blob/master/exercises/basic/pod-topo/pod-topo.png?raw=true" width="600" heidth = '350'>  
</p>

Заходим в каталог p4\tutorials\exercises\basiс и запускаем mininet командой  
```make run```  
Пробуем выполнить ping между хостами в топологии. Как видно, ни один пинг не прошел:  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/2.2.jpg?raw=true">  
</p>

### Задание Implementing Basic Forwarding
Исправляем парсер. 
```
parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start {
        transition parse_ethernet;
    }

    state parse_ethernet {
        packet.extract(hdr.ethernet);
        transition select(hdr.ethernet.etherType) {
            TYPE_IPV4: parse_ipv4;
            default: accept;
        }
    }

    state parse_ipv4 {
        packet.extract(hdr.ipv4);
        transition accept;
    }

}
```  
Далее был прописан action, с помощью которого задается порт, обновляется адрес назначения, обновляется адрес источника и уменьшается значение ttl:  
```
    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
        standard_metadata.egress_spec = port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }
```
Добавили проверку ipv4 заголовка:
```
    apply {
        if (hdr.ipv4.isValid()) {
            ipv4_lpm.apply();
        }
    }
```
Исправление депарсера:
```
control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        packet.emit(hdr.ipv4);
    }
}
```

Итоговый файл: [basic.p4](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/basic.p4).  
Проверки работоспособности: снова запускаем mininet. На этот раз все работает:
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/5.5.jpg?raw=true" width="500" heidth = '350'>  
</p>

### Задание Implementing Basic Tunneling  
Во второй части работы нужно было добавить поддержку базового туннелирования. топология сети:  
<p align="center">
<img src="https://github.com/p4lang/tutorials/blob/master/exercises/basic_tunnel/topo.png?raw=true" width="600" heidth = '350'>  
</p>  
Заходим в каталог p4\tutorials\exercises\basic_tunnel и исправляем файл basic_tunnel.p4. Исправляем парсер:  

```
parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start {
        transition parse_ethernet;
    }

    state parse_ethernet {
        packet.extract(hdr.ethernet);
        transition select(hdr.ethernet.etherType) {
            TYPE_MYTUNNEL: parse_myTunnel;
            TYPE_IPV4: parse_ipv4;
            default: accept;
        }
    }

    state parse_myTunnel {
        packet.extract(hdr.myTunnel);
        transition select(hdr.myTunnel.proto_id) {
            TYPE_IPV4: parse_ipv4;
            default: accept;
        }
    }

    state parse_ipv4 {
        packet.extract(hdr.ipv4);
        transition accept;
    }

}
```
Устанавливаем выходной порт с помощью action:
```
    action myTunnel_forward(egressSpec_t port) {
        standard_metadata.egress_spec = port;
    }
```
Переадрисация туннелирования:
```
    table myTunnel_exact {
        key = {
            hdr.myTunnel.dst_id: exact;
        }
        actions = {
            myTunnel_forward;
            drop;
        }
        size = 1024;
        default_action = drop();
    }
```
Проверка валидности заголовка myTunnel:
```
    apply {
        if (hdr.ipv4.isValid() && !hdr.myTunnel.isValid()) {
            // Process only non-tunneled IPv4 packets
            ipv4_lpm.apply();
        }

        if (hdr.myTunnel.isValid()) {
            // process tunneled packets
            myTunnel_exact.apply();
        }
    }
```
Исправляем депарсер:
```
control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        packet.emit(hdr.myTunnel);
        packet.emit(hdr.ipv4);
    }
}
```  
Итоговый файл: [basic_tunnel.p4](https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/basic_tunnel.p4).  

Запускаем mininet. Для тестирования были открыты терминалы для h1 и h2,  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/6.6.jpg?raw=true" width="600" heidth = '350'>  
</p>  

На h2 запускаем ./receive.py. Сначала проводим проверку без туннелирования:  
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/7.7.jpg?raw=true">  
</p>  
Проверка отправки сообщения с помощью туннелирования. Теперь видно заголовок туннеля:
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/8.8.jpg?raw=true">  
</p>

Проверка отправки сообщения с помощью туннелирования на адрес h3. Сообщение дошло на h2:
<p align="center">
<img src="https://github.com/Vladkilinichh/2023_2024-network_programming-k34212-Kilinich_V_A/blob/main/lab04/images/9.9.jpg?raw=true">  
</p>

---  
### Вывод  
В ходе лабораторной работы были выполнены задания для ознакомления с языком программирования P4: реализация алгоритмов переадресации и туннелирования в тестовой среде.
