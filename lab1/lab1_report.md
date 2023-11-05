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
## "Установка CHR и Ansible, настройка VPN"  

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
В результате установки openvpn-as были получены логин и пароль для входа в веб-интерфейс администратора по ссылке https://84.201.177.192:943/admin:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/OVPN_activation.jpg)  
В этом окне былл введен ключ активации для возможности бесплатного подключения пользователей (огрничение – 2 подключения). В качестве протокола был указан только TCP. Также был полностью отключен TLS Control Channel Security.  
Для полуучения файла конфигурации клиента сначала был создан пользователь, после чего получен данный файл во вкладке User Profiles:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/OVPN_user.jpg)
Для доступа к сети в настройках VirtuaBox для CHR был настроен сетевой мост. Для того, чтобы устоновить подключение к VPN-серверу на CHR через WinBox был добавлен ранее полученный файл конфигурации клиента. Из данного файла были получены сертификаты с помощью команды:
```
certificate import file-name=***
```  
Во вкладке interfaces был создан интерфейс OVPN client. Были указаны следующие параметры: Connect to – 84.201.177.192 (белый IP-адрес сервера), Port – 443 (порты указанный при настройке VPN-сервера), User – olga (имя созданного клиента), Certificate – импортированный ранее сертификат:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/WinBox_OVPN_client.jpg)  
Как видно на изображении выше, подключение было установлено (в правом нижнем углу: Status:connected). Успешность подключения также была проверена пингом с CHR на внутренний адрес сервера:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab1/imgs/ping_VPN_server.jpg)  
### Вывод  
В ходе лабораторной работы была развернута виртуальная машина на базе платформы Yandex Cloud – Ubuntu, на которой далее были установлены система контроля конфигураций Ansible и python3. Также была установлена машина CHR в VirtualBox, для которой был настроен выход в сеть. Также был поднят VPN туннель между сервером на Ubuntu и клиентом на RouterOS (CHR). Для реализации VPN был выбран OpenVPN с использованием веб-интерфейс администратора Access Server. Для удобства работы с роутером также был установлен WinBox.
