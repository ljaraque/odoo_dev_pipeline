## **ODOO DEVELOPMENT PIPELINE**  
(Development --> Lab --> Test --> Prod)    
  
---  


### **1. Install Odoo from Repository**  

**ENVIRONMENT SETUP**  

OS
```
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.1 LTS
Release:	16.04
Codename:	xenial
```

Python
```
Python 3.7.7 (default, Mar 10 2020, 17:25:08) 
[GCC 5.4.0 20160609] on linux
```  

Postgres  
```
psql (PostgreSQL) 9.5.21
```

**COMMANDS SEQUENCE**    
```
sudo apt-get update
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
sudo su - postgres -c "createuser -s odoo"
sudo su - postgres -c "createdb odoo"
sudo -u postgres psql
```
Then, in `Postgres` console:  
```
grant all privileges on database odoo to odoo;
```
Some of the following dependencies might be required:  
```
sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less

```

Install Wkhtmltopdf, required by Odoo. For this, we need to get the corresponding package for our OS from the page https://wkhtmltopdf.org/downloads.html. In this case for `Ubuntu 16.04`, we do:
```
cd /tmp
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.xenial_amd64.deb
sudo apt install ./wkhtmltox_0.12.6-1.xenial_amd64.deb
```
```
sudo su - odoo
```

Then, we clone the current version of Odoo (`branch 13.0`) with:  
```
git clone https://www.github.com/odoo/odoo --branch 13.0 /opt/odoo/odoo13
```

```
cd /opt/odoo
virtualenv -p python3.7 venv3.7
. venv3.7/bin/activate
pip install --upgrade pip
pip install -U wheel
pip install -r odoo13/requirements.txt
deactivate
mkdir /opt/odoo/odoo13-custom-addons
exit
```

Then we create the Odoo configuration file:  
```
sudo nano /etc/odoo13.conf

```
and add the following content:  
```
[options]
; This is the password that allows database operations:
admin_passwd = odoodbpass
db_host = False
db_port = False
db_user = odoo
db_password = False
addons_path = /opt/odoo/odoo13/addons,/opt/odoo/odoo13-custom-addons
```

Then create a systemd unit file:  
```
sudo nano /etc/systemd/system/odoo13.service
```
and add the followinf content:  
```
[Unit]
Description=Odoo
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo
PermissionsStartOnly=true
User=odoo
Group=odoo
ExecStart=/opt/odoo/venv3.7/bin/python3.7 /opt/odoo/odoo13/odoo-bin -c /etc/odoo13.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```
then, reload `systemd` daemon and start `odoo13` service:  
```
sudo systemctl daemon-reload
sudo systemctl enable --now odoo13
```  
To check status of `odoo13` service:  
```
sudo systemctl status odoo13
```

To stop or start odoo13 service use:  
```
sudo systemctl stop odoo13
```
or  
```
sudo systemctl stop odoo13
```
respectively.  

The output of the system execution can be monitored with:  
```
watch "journalctl _SYSTEMD_UNIT=odoo13.service | tail -40"
```