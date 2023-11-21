University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2023/2024  
Group: K34202  
Author: Gladushko Olga Vladimirovna  
Lab: Lab2  
Date of create: 17.11.2023  
Date of finished: 23.11.2023  

---
# Отчет по лабораторной работе №2  
## "Развертывание дополнительного CHR, первый сценарий Ansible"  

### Цель работы  
Целью данной работы является настроить несколько сетевых устройств c помощью Ansible и собрать информацию о них. Правильно собрать файл Inventory.  

## Ход работы  
Для дальнейшего выполнения работы необходимо было установить второй CHR той же версии, что и первый, чтобы не возникло проблем при подключении к VPN-серверу. В OpenVPN Server был добавлен второй клиент и chr был успешно подключен. 
Далее на Ubuntu был изменен инвентори-файл hosts: там были указаны оба chr (объединенные в группу chrs) – выданные им IP-адреса и rid, которы понадобится при настройке ospf. Также были указаны данные для подключения к chr по ssh, тип подключения и ОС. Файл hosts имел следующий вид: 
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
Далее был создан playbook, с помощью которого на роутерах должен быть создан новый пользователь, настроен ntp client, настоен ospf с указанием router id – rid, указанный выше. Также были собраны факты об интерфейсах, чтобы вывести на экран при запуске плейбука. Файл playbook содержит следующие команды: 
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
Для устранения ошибки подключения к chr были созданы и добавлены ssh-ключи на оба роутера.  
Playbook был запущен командой: 
```
ansible-playbook -i hosts playbook.yaml
``` 
Результат выполнения: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/play_recup.jpg) 
Все три таски с изменением конфигурации роутеров были успешно выполнены. Также и вывелись собранные файкты, часть вывода: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/debug_output.jpg)      
  С пмомощью вывода команды ```export``` можно увидеть, что все команды были успешно добавлены на роутеры:
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/export.jpg) 
На роутерах были выведены пользователи, как видно, новый пользоввтель chr_user был добавлен: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/user.jpg) 
Также была выведена информация об ntp клиенте, теперь в качестве NTP-сервера указано введенное доменное имя сервера из кластера pool.ntp.org: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/ntp.jpg) 
Также были выведены neighbor для каждого роутера, теперь они могут друг друга увидеть, что подтверждает верную настройку ospf: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/neighbor.jpg) 
Для подтверждения работоспособнотсти сети также были запущены пинги: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/ping.jpg) 
Полные конфигурации устройств, собранные командой export: [chr1](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/chr1_conf), [chr2](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/chr2_conf).
### Вывод  
В ходе лабораторной работы были настроены CHR с помощью ansible. Были созданы два файла: инвентори-файл и плейбук. С пмощью прописанных команд на роутерах были настроены: логин/пароль для нового пользователя, NTP client, ospf с указанием router id, а также были побраны данные настройки роутеров. Схема сети, полученная в результате выполнения работы: 
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab2/imgs/lab2.drawio.png)
