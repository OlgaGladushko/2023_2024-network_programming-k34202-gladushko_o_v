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
В каталоге задания уже был файл с неполным кодом basic.p4. После запуска mininet была попытка выполнения ping между хостами в топологии. Как видно, ни однитпинг не прошел:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab4/imgs/pingall_X.jpg)  

