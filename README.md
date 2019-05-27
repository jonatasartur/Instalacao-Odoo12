
# Instalação Odoo v12 no Ubuntu 18.04

Este tutorial é baseado na instalação da versão Comunity do Odoo 12 com a localização brasileira, ao qual utilizaremos o excelente fork da TrustCode (https://github.com/Trust-Code/odoo-brasil)


## 1 - AJUSTES NO UBUNTU 18.04
### 1.1 - Configurar com IP manual a sua respectiva interface de rede:
  - Defina o endereço IP (ex: 192.168.15.5)
  - Defina sua subrede (ex: 192.168.15.0/24)
  - Defina o seu respctivo gateway (ex: 192.168.15.1)
  - Defina os seus servidores DNS (ex: 192.168.15.1, 8.8.8.8, 8.8.4.4)  

### 1.2 - Dados do usuário padrão:
  - Defina o nome de sua empresa (ex: COMDESK Tecnologia)
  - Defina o nome do servidor (ex: srvOdoo)
  - Defina o seu nome de usuário (ex: comdesk)
  - Defina a respectiva senha do usuário

### 1.3 - Via terminal, habilite o usuário root:
```sh
$ sudo passwd root
```
  - Informe a senha do usuário atual
  - Informe uma nova senha para o usuário "root" (duas vezes)
  - Saia da conta do usuário atual e acesse com o usuário "root".
  
### 1.4 - Habilitar o acesso SSH no Ubuntu:
```sh
$ vim /etc/ssh/sshd_config
```

- Comente a linha *"PermitRootLogin prohibit-password"* e adicione o item ***"PermitRootLogin yes"***, como mostrado logo abaixo.
 ```
 #Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin yes
StrictModes yes
```
- Reinicie o serviço SSH:
 ```sh
 systemctl restart sshd
 ```

### 1.5 - Atualize o servidor:
```sh
sudo apt update && sudo apt upgrade
```

### 1.6 - Definir a região de fuso horário:
```sh
sudo dpkg-reconfigure tzdata
```

### 1.7 - Instalar e configurar o servidor NTP Chrony:
```sh
apt-get install chrony
```
- Após a instalação, edite o arquivo de configuração e ajuste a sua zona de fuso horário:
```sh
vim /etc/chrony/chrony.conf
```
- Adicione os seus respectivos servidores NTP:

> server a.ntp.br iburst
> server b.ntp.br iburst
> server c.ntp.br iburst

### 1.8 - Habilitar o repositório "universe" que contém os pacotes python-pip.
```sh
vim /etc/apt/sources.list
```
- E então, adicionar ***"universe"*** , no final de cada linha, como mostrado a seguir:

> deb http://archive.ubuntu.com/ubuntu bionic main **universe**
> deb http://archive.ubuntu.com/ubuntu bionic-security main **universe**
> deb http://archive.ubuntu.com/ubuntu bionic-updates main **universe**

- Atualizar repositórios:
```sh
sudo apt update && sudo apt upgrade
```

- Confira a versão do Python. Deve ser acima da 3.6:
```sh
python3 --version
```

- Reinicie o servidor:
```sh
init 6
```



## 2 - INSTALAÇÃO ODOO

### 2.1 - Instale as dependências:

```sh
sudo apt install git gcc python3-pip build-essential python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools python3-pypdf2
```
```sh
pip3 install Babel decorator docutils ebaysdk feedparser gevent greenlet html2text Jinja2 lxml Mako MarkupSafe mock num2words ofxparse passlib Pillow psutil psycogreen psycopg2 pydot pyparsing PyPDF2 pyserial python-dateutil python-openid pytz pyusb PyYAML qrcode reportlab requests six suds-jurko vatnumber vobject Werkzeug XlsxWriter xlwt xlrd
```

```sh
pip3 install pyldap beautifulsoup4 python-stdnum
```

```
apt-get install npm
```
```sh
sudo ln -s /usr/bin/nodejs /usr/bin/node
```
```sh
sudo npm install -g less less-plugin-clean-css rtlcss
```
```sh
sudo apt-get install node-less
```
```sh
sudo python3 -m pip install libsass
```
```sh
sudo apt-get install software-properties-common
```

### 2.2 - Instalar PostgreSQL 9.6:

- Crie e edite um arquivo de repositório para o PostgreSQL:
```sh
sudo vim /etc/apt/sources.list.d/pgdg.list
```
- Adicione a seguinte linha no repositório e saia salvando o arquivo:
> deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

- Importe a chave de assinatura do repositório:
```sh
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

- Atualize os repositórios:
```sh
sudo apt-get update
```

- Faça a instalação do PostgreSQL:
```sh
sudo apt-get install postgresql-9.6
```

- Seguindo as etapas de instalação e configuração, defina uma senha para o usuário "postgres":
```sh
sudo passwd postgres
```
```
Enter new UNIX password:**** (ex: Psql-123456)
Retype new UNIX password:**** (ex: Psql-123456)
passwd: password updated successfully
```

- Crie o usuário "Odoo":
```sh
sudo su postgres
```
```
createuser -s odoo
```
```
createuser -s ubuntu_user_name (ex:comdesk)
```

- Volte para o usuário anterior, pressionando Ctrl + d

- Crie o usuário do banco de dados:
```sh
sudo su - postgres -c "createuser -s odoo"
```
- Verifique a versão do Postgres:
```sh
psql --version
```
> A versão deve ser: psql (PostgreSQL) 9.6.11


- Crie o usuário do sistema Odoo:
```sh
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
```

- Adicione o usuário ao grupo sudo:
```sh
sudo adduser odoo sudo
```

- Habilite e inicie o serviço PostgreSQL:
```sh
systemctl enable postgresql.service
```
```sh
systemctl start postgresql.service
```


### 2.3 - Instalar Gdata

- Como usuário "root", faça os seguintes procedimentos:
```sh
cd /opt/odoo
sudo wget https://pypi.python.org/packages/a8/70/bd554151443fe9e89d9a934a7891aaffc63b9cb5c7d608972919a002c03c/gdata-2.0.18.tar.gz
sudo tar zxvf gdata-2.0.18.tar.gz
sudo chown -R odoo: gdata-2.0.18
sudo -s
cd gdata-2.0.18/
python setup.py install
exit
```

### 2.4 - Instale o Wkhtmltopdf para poder imprimir relatórios em PDF
- Como usuário "root", faça os seguintes procedimentos:
```sh
cd /tmp
wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo apt install ./wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin
sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin
```

### 2.5 - Instalação e configuração do Odoo

- Ainda como usuário "root", faça os seguintes procedimentos:
```sh
cd /opt/odoo
```
```sh
sudo git clone --depth 1 --branch 12.0 https://www.github.com/odoo/odoo /opt/odoo/odoo-server/
```

- Instale os pacotes Python necessários pata Odoo 12:
```sh
cd /opt/odoo/odoo-server
```
```sh
sudo pip3 install -r requirements.txt
```

- Crie o diretório para o arquivo de log:
```sh
sudo mkdir /var/log/odoo
cd /var/log/odoo
sudo touch odoo-server.log
sudo chown -R odoo:root /var/log/odoo
```

- Crie o diretório e subdiretório de módulos customizados:
```sh
sudo su odoo -c "mkdir -p /opt/odoo/custom/addons"
```

- Mude as permissões da pasta do Odoo:
```sh
sudo chown -R odoo:odoo /opt/odoo/*
```

- Crie e edite o arquivo de configuração:
```sh
sudo vim /etc/odoo-server.conf
```

- Coloque o seguinte conteúdo:
> [options]
> ; This is the password that allows database operations:
> admin_passwd = Psql-123456 ;Senha Master do Postgres
> db_host = False
> db_port = False
> db_user = odoo
> db_password = False
> xmlrpc_port = 8069
> logfile = /var/log/odoo/odoo-server.log
> addons_path = /opt/odoo/odoo-server/addons, /opt/odoo/custom/addons/oca, /opt/odoo/odoo-server/odoo, /opt/odoo/custom/addons, /opt/odoo/custom/addons/odoo-brasil, /opt/odoo/custom/addons/oca/server-ux, /opt/odoo/custom/addons/oca/account-financial-tools


- Defina as respectivas permissões ao arquivo:
```sh
sudo chown odoo: /etc/odoo-server.conf
sudo chmod 640 /etc/odoo-server.conf
```

- Criar o arquivos systemd, para rodar o Odoo como serviço:
```sh
vim /etc/systemd/system/odoo12.service
```

- Coloque o seguinte conteúdo:
> [Unit]
> Description=Odoo Open Source ERP
> Requires=postgresql.service
> After=network.target postgresql.service
> 
> [Service]
> Type=simple
> SyslogIdentifier=odoo-server
> PermissionsStartOnly=true
> User=odoo
> Group=odoo
> ExecStart=/opt/odoo/odoo-server/odoo-bin -c /etc/odoo-server.conf
> WorkingDirectory=/opt/odoo/odoo-server/
> StandardOutput=journal+console
> 
> [Install]
> WantedBy=default.target


- Após, execute os comandos a seguir:
```sh
systemctl daemon-reload
```
```sh
systemctl restart odoo12.service
```
```
systemctl status odoo12.service
```
```
systemctl enable odoo12.service
```
```
sudo journalctl -u odoo12
```
```
cat /var/log/odoo/odoo-server.log
```

- Abra um navegador web e acesse o seu respectivo servidor:
> localhost:8069 ou seu respectivo endereço IP (ex: 192.168.15.5:8069)


Observação: Não crie o banco de dados ainda!!!
