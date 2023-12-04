University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2023/2024  
Group: K34202  
Author: Gladushko Olga Vladimirovna  
Lab: Lab3  
Date of create: 01.12.2023  
Date of finished: 07.12.2023  

---
# Отчет по лабораторной работе №3  
## "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"  

### Цель работы  
Целью данной работы является собрать всю возможную информацию об устройствах c помощью Ansible и Netbox и сохранить их в отдельном файле.  

## Ход работы  
На виртуальной машине Ubuntu был поднят Netbox. Для этого сначала был установлен и настроен PostgreSQL с помощью следующи команд:  
```
sudo apt install postgresql libpq-dev -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo passwd postgres
su - postgres
psql
CREATE DATABASE netbox;
CREATE USER netbox WITH ENCRYPTED password '123';
GRANT ALL PRIVILEGES ON DATABASE netbox to netbox;
```  
Далее был установлен Redis с помощью команды:
```
sudo apt install -y redis-server
```
После проделанных шагов был уже установлен и настроен сам Netbox:
```
sudo apt install python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev git -y
sudo pip3 install --upgrade pip
sudo mkdir -p /opt/netbox/ && cd /opt/netbox/
sudo git clone -b master https://github.com/netbox-community/netbox.git .
sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
cd /opt/netbox/netbox/netbox/
sudo cp configuration.example.py configuration.py
sudo ln -s /usr/bin/python3 /usr/bin/python
sudo /opt/netbox/netbox/generate_secret_key.py
```  
Сгенерированный случайный код был скопирован для дальнейшего использования в конфигурационном файле. В данный файл был также добавлен ранее созданный пользователь netbox и его пароль.  
Для дальнейшей настройки были выполнены следующие команды:  
```
sudo /opt/netbox/upgrade.sh
source /opt/netbox/venv/bin/activate
cd /opt/netbox/netbox
python3 manage.py createsuperuser
sudo reboot
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
sudo cp /opt/netbox/contrib/*.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start netbox netbox-rq
sudo systemctl enable netbox netbox-rq
```  
Также был настроен веб-сервер Nginx для доступа к Netbox через браузер:
```
sudo apt install -y nginx
sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
```
Файл netbox был отредактирован так, чтобы в нем был белый IP-адрес ВМ и порт 8080 (порт 80 уже был занят OpenVPN).  
Для того, чтобы изменения вступили в силу, были выполнены следующие команды:  
```
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
sudo systemctl restart nginx
```
Настроить Netbox можно в браузере по указанному адресу и номеру порта, также необходимо указать имя созданного суперпользователя и пароль.  
В веб-интерфейсе был создан сайт, мануфактура, тип устройства, функция устройства, а далее само устройство – chr1 и CHR2. Для указания IP-адресов устройств необходимо было создать интерфейсы и IP-адреса. Занесенные в Netbox устройства:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/Netbox_devices.jpeg)  
Далее для работы с Netbox через Ansible нужно было устонавить модуль netbox.netbox командой:
```
ansible-galaxy collection install netbox.netbox
```
Для сохранения всех данных из Netbox в отдельный файл был создан inventory-файл, в котром был указан плагин, адрес netbox, токен (предварительно создвнный через веб-интерфейс):
```
plugin: netbox.netbox.nb_inventory
api_endpoint: http://130.193.42.42:8080/
token: созданный_токен
validate_certs: False
config_context: False
interfaces: True
```
Вся информация была сохранена в файл nb_inventory.yml с помощью команды:
```
ansible-inventory -v --list -y -i netbox_inventory.yml > nb_inventory.yml
```
Далее был написан сценарий для настройки CHR на основе данных из Netbox. Для этого полученный файл был изменен: в него добавлены переменные для подключения к устройствам из файла прошлой лабораторной работы, а также группа роутеров была названа devices вместо указанного по умолчанию ungrouped. Измененный файл: [inventory.yml](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/nb_inventory.yml).  
В playbook были прописаны команды по изменению имени устройства на имя, указанное в Netbox,  а также по добавлению IP-адреса на устройство (адрес, выданный VPN):
```
- name: Configuration
  hosts: devices
  tasks:
    - name: Set Name
      community.routeros.command:
        commands:
          - /system identity set name="{{interfaces[0].device.name}}"
    - name: Set IP-address
      community.routeros.command:
        commands:
        - /interface bridge add name="{{interfaces[1].display}}"
        - /ip address add address="{{interfaces[1].ip_addresses[0].address}}" interface="{{interfaces[1].display}}"
```
Обе таски были успешно выполнены, как можно увидеть ниже, именя устройств изменились (раньше они азывались MikroTik), а также добавились имена на новый созданный интерфейс:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/chr1.jpg) ![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/CHR2.jpg)  
Далее playbook был изменен для того, чтобы собрать серийный номер каждого устройства и внести его в Netbox:  
```
- name: Serial Numbers
  hosts: devices
  tasks:
    - name: Serial Number
      community.routeros.command:
        commands:
          - /system license print
      register: license
    - name: Add Serial Number
      netbox_device:
        netbox_url: http://130.193.42.42:8080/
        netbox_token: созданный_токен
        data:
          name: "{{interfaces[0].device.name}}"
          serial: "{{license.stdout_lines[0][0].split(' ').1}}"
        state: present
        validate_certs: False
```
Для выполнения плэйбука потребовалось установить дополнительный модуль python3-pynetbox. После этого таски успешно выполнились. В веб-интерфейсе Netbox можно увидеть добавленный серийный номер для каждого CHR:  
![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/chr1_sn.jpg) ![.](https://github.com/OlgaGladushko/2023_2024-network_programming-k34202-gladushko_o_v/blob/main/lab3/imgs/CHR2_sn.jpg)  
### Вывод  
В ходе лабораторной работы была собрана информация об устройствах и сохранена в отдельном файле с помощью Ansible и Netbox, а также была произведена настройка CHR на основе собранных данных.
