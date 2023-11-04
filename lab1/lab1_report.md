University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2023/2024  
Group: K34202  
Author: Gladushko Olga Vladimirovna  
Lab: Lab1  
Date of create: 30.10.2023  
Date of finished: 09.11.2023  

---
# Отчет по лабораторной работе №1  
## "Установка ContainerLab и развертывание тестовой сети связи"  

### Цель работы  
Целью данной работы является развертывание виртуальной машины на базе платформы Yandex Cloud с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox.  

## Ход работы  
В Yandex Cloud была создана виртуальная машина Ubuntu 22:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/YC_vm.jpg)  
К данной машине было осущтвлено ssh подключение. Далее на ней были установлены python3 и Ansible с помощью следующих команд:  
```
sudo apt install python3-pip  
sudo pip3 install ansible
```  
На VirtualBox был установлен CHR, образ которого был взят с сайта MikroTik:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/CHR_VB.jpg)  
Далее на Ubuntu был установлен пакет "openvpn-as" с помощью следующих команд:  
```
apt update && apt -y install ca-certificates wget net-tools gnupg
wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repository.asc
echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repository.asc] http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt update && apt -y install openvpn-as  
```
Также на официальном сайте OpenVPN рекомендовано установить обновления и перенастроить часовой пояс с помощью команд:
```
apt update  
apt upgrade  
apt install tzdata  
dpkg-reconfigure tzdata  
```  
В результате установки openvpn-as были получены логин и пароль для входа в веб-интерфейс администратора:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/OVPN_activation.jpg)  
В этом окне былл введен ключ активации для возможности бесплатного подключения пользователей (огрничение – 2 подключения). 
