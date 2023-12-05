University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2023/2024  
Group: K34202  
Author: Gladushko Olga Vladimirovna  
Lab: Lab4  
Date of create: 05.12.2023  
Date of finished: 07.12.2023  

---
# Отчет по лабораторной работе №4  
## "Базовая 'коммутация' и туннелирование используя язык программирования P4"  

### Цель работы  
Целью данной работы является изучить синтаксис языка программирования P4 и выполнить 2 обучающих задания от Open network foundation для ознакомления на практике с P4.  

## Ход работы  
На компьютере уже были устанвлены Vagrant и VirtualBox, поэтому необходимо было только склонировать на него репозиторий https://github.com/p4lang/tutorials. Далее в папке vm-ubuntu-20.04 была развернута тестовая среда командой ```vagrant up```. В результате установки была создана ВМ с двумя аккаунтами:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab4/imgs/users.jpg)  

### Implementing Basic Forwarding  
В первой части работы нужно было написать программу на P4, которая реализует базовую переадресацию с использованием следующей топологии  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab4/imgs/topo1.png)  
В каталоге задания уже был файл с неполным кодом basic.p4. После запуска mininet была попытка выполнения ping между хостами в топологии. Как видно, ни один пинг не прошел:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab4/imgs/pingall_X.jpg)  
Для того, чтобы заработала передресация пакетов, в код с уже имеющимися ключевыми элементами нужно было добавить логику парсера. В начале файла уже определены необходимые библиотеки и константы, а также блок заголовков. Из входящего пакета с помощью парсера будет получен заголовок ipv4.
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
Далее был прописон action, который задает порт, обновляет адрес назначения, обновляет адрес источника и уменьшает значение ttl:  
```
    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
        standard_metadata.egress_spec = port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }
```
Также было прописано пропущенное условие, проверяющее правильность ipv4 заголовка:
```
    apply {
        if (hdr.ipv4.isValid()) {
            ipv4_lpm.apply();
        }
    }
```
Далее был написан депрасер, который обратно сформировывает пакет из заголовков:
```
control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        packet.emit(hdr.ipv4);
    }
}
```
Итоговый файл: basic.p4.
Для проверки работоспособности снова был запущен mininet. На этот раз все пинги прошли успшно:
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab4/imgs/ping%26pingall.jpg)  


