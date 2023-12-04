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
Настроить Netbox можно в браузере по указанному адресу и номера порта, также необходимо указать имя созданного суперпользователя и пароль.
В веб-интерфейсе был создан сайт, мануфактура, тип устройства, функция устройства, а далее само устройство – chr1 и CHR2. Для указания IP-адресов устройств необходиом было создать интерфейсы и IP-адреса. Занесенные в Netbox устройства:  
![.](
