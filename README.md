
# Tutorial Instalação Odoo v12 (TrustCode) no Ubuntu 18.04


## AJUSTES NO UBUNTU 18.04

### Configurar com IP manual a sua respectiva interface de rede:
  - Defina o endereço IP (ex: 192.168.15.5)
  - Defina sua subrede (ex: 192.168.15.0/24)
  - Defina o seu respctivo gateway (ex: 192.168.15.1)
  - Defina os seus servidores DNS (ex: 192.168.15.1, 8.8.8.8, 8.8.4.4)  

### Dados do usuário padrão:
  - Defina o nome de sua empresa (ex: COMDESK Tecnologia)
  - Defina o nome do servidor (ex: srvOdoo)
  - Defina o seu nome de usuário (ex: comdesk)
  - Defina a respectiva senha do usuário

### Via terminal, habilite o usuário root:
	> sudo passwd root
  
  - Informe a senha do usuário atual
  - Informe uma nova senha para o usuário "root" (duas vezes)
  - Saia da conta do usuário atual e acesse com o usuário "root".

### Habilitar o acesso SSH no Ubuntu:
	`vim /etc/ssh/sshd_config

- Comente a linha "PermitRootLogin prohibit-password" e adicione o item "PermitRootLogin yes" logo abaixo.

	> #Authentication:
	LoginGraceTime 120
	#PermitRootLogin prohibit-password
	**PermitRootLogin yes**
	StrictModes yes

 - Reinicie o serviço SSH:
	> systemctl restart sshd

### Atualize o servidor:
  `sudo apt update && sudo apt upgrade

### Definir a região de fuso horário:
	sudo dpkg-reconfigure tzdata

### Instalar e configurar o servidor NTP Chrony:
	apt-get install chrony

	- Após a instalação, edite o arquivo de configuração e ajuste a sua zona de fuso horário:
	`vim /etc/chrony/chrony.conf`
	
	- Adicione o seguinte conteúdo:	
		> server a.ntp.br iburst
		server b.ntp.br iburst
		server c.ntp.br iburst


Confira a versão do Python. Deve ser acima da 3.6:
python3 --version


Habilitar o repositório "universe" que contém os pacotes python-pip.
vim /etc/apt/sources.list

E então, adicionar "universe" , no final de cada linha, como mostrado a seguir:
deb http://archive.ubuntu.com/ubuntu bionic main universe
deb http://archive.ubuntu.com/ubuntu bionic-security main universe
deb http://archive.ubuntu.com/ubuntu bionic-updates main universe

Atualizar repositórios:
sudo apt update && sudo apt upgrade



//INSTALAÇÃO ODOO

Instale as dependências:
sudo apt install git gcc python3-pip build-essential python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools python3-pypdf2

pip3 install Babel decorator docutils ebaysdk feedparser gevent greenlet html2text Jinja2 lxml Mako MarkupSafe mock num2words ofxparse passlib Pillow psutil psycogreen psycopg2 pydot pyparsing PyPDF2 pyserial python-dateutil python-openid pytz pyusb PyYAML qrcode reportlab requests six suds-jurko vatnumber vobject Werkzeug XlsxWriter xlwt xlrd

pip3 install pyldap beautifulsoup4 python-stdnum


apt-get install npm
sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css rtlcss
sudo apt-get install node-less
sudo python3 -m pip install libsass

sudo apt-get install software-properties-common


Instalar PostgreSQL 9.6:
sudo vim /etc/apt/sources.list.d/pgdg.list

Adicione a linha para o repositório:
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sudo apt-get update
sudo apt-get install postgresql-9.6


Definir uma senha para o Postgres:
$ sudo passwd postgres
Enter new UNIX password:**** (ex: Psql-503501)
Retype new UNIX password:**** (ex: Psql-503501)
passwd: password updated successfully


Crie o usuário do banco de dados:
sudo su postgres
createuser -s odoo
createuser -s ubuntu_user_name (ex: comdesk)
exit(Ctrl + d)

Crie o usuário do banco de dados:
sudo su - postgres -c "createuser -s odoo"

Verificar a versão do Postgres:
# psql --version
psql (PostgreSQL) 9.6.11

Crie o usuário do sistema Odoo:
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo

Adicione o usuário ao grupo sudo:
sudo adduser odoo sudo


Habilite e inicie o serviço PostgreSQL:
systemctl enable postgresql.service
systemctl start postgresql.service


Instalar Gdata

cd /opt/odoo
sudo wget https://pypi.python.org/packages/a8/70/bd554151443fe9e89d9a934a7891aaffc63b9cb5c7d608972919a002c03c/gdata-2.0.18.tar.gz
sudo tar zxvf gdata-2.0.18.tar.gz
sudo chown -R odoo: gdata-2.0.18
sudo -s
cd gdata-2.0.18/
python setup.py install
exit


Instale o Wkhtmltopdf para poder imprimir relatórios em PDF:

cd /tmp
wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo apt install ./wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin
sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin



Instale o Odoo:
cd /opt/odoo
sudo git clone --depth 1 --branch 12.0 https://www.github.com/odoo/odoo /opt/odoo/odoo-server/


Install the necessary Python packages for the Odoo-12
cd /opt/odoo/odoo-server
sudo pip3 install -r requirements.txt

Crie o diretório para o arquivo de log:
sudo mkdir /var/log/odoo
cd /var/log/odoo
sudo touch odoo-server.log
sudo chown -R odoo:root /var/log/odoo

Crie o diretório e subdiretório de módulos customizados:
sudo su odoo -c "mkdir -p /opt/odoo/custom/addons"

Mude as permissões da pasta do Odoo:
sudo chown -R odoo:odoo /opt/odoo/*


Crie o arquivo de configuração:
sudo vim /etc/odoo-server.conf

Coloque o seguinte conteúdo:
[options]
; This is the password that allows database operations:
admin_passwd = Psql-503501 ;Senha Master do Postgres
db_host = False
db_port = False
db_user = odoo
db_password = False
xmlrpc_port = 8069
logfile = /var/log/odoo/odoo-server.log
addons_path = /opt/odoo/odoo-server/addons, /opt/odoo/custom/addons/oca, /opt/odoo/odoo-server/odoo, /opt/odoo/custom/addons, /opt/odoo/custom/addons/odoo-brasil, /opt/odoo/custom/addons/oca/server-ux, /opt/odoo/custom/addons/oca/account-financial-tools

Defina as respectivas permissões ao arquivo:
sudo chown odoo: /etc/odoo-server.conf
sudo chmod 640 /etc/odoo-server.conf


Criar o arquivos systemd, para rodar o Odoo como serviço:
vim /etc/systemd/system/odoo12.service

Coloque o seguinte conteúdo:
[Unit]
Description=Odoo Open Source ERP
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo-server
PermissionsStartOnly=true
User=odoo
Group=odoo
ExecStart=/opt/odoo/odoo-server/odoo-bin -c /etc/odoo-server.conf
WorkingDirectory=/opt/odoo/odoo-server/
StandardOutput=journal+console

[Install]
WantedBy=default.target


Após, execute os comandos a seguir:
systemctl daemon-reload
systemctl restart odoo12.service
systemctl status odoo12.service

systemctl enable odoo12.service
sudo journalctl -u odoo12
cat /var/log/odoo/odoo-server.log


Open Browser on and type :
localhost:8069

Não crie o banco de dados ainda!!!


