
# Instalação Odoo v12 no Ubuntu 18.04

Este tutorial é baseado na instalação da versão Comunity do Odoo 12 com a localização brasileira, ao qual utilizaremos o excelente fork da TrustCode (https://github.com/Trust-Code/odoo-brasil)

Este tutorial também pode ser visto com uma melhor organização estrutural, em: http://comdesk.com.br/blog/item/10-instalacao-odoo-v12-no-ubuntu-18-04


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
sudo apt install git gcc python3-pip build-essential libpq-dev python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools python3-pypdf2
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

- Crie o usuário "odoo" no postgres, para isso, altere o usuário atual para postgres, a fim de ter os privilégios necessários para configurar a base de dados:
```sh
sudo su postgres
```
- Crie o novo usuário do banco de dados para gerenciamento. O usuário “odoo12” terá direitos de acesso para se conectar, criar e eliminar bancos de dados.

```
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo12
```

- Saia do usuário Postgres para continuar a instalação:
```
exit
```

- Verifique a versão do Postgres:
```sh
psql --version
```
> A versão deve ser: psql (PostgreSQL) 9.6.11


- Crie o usuário do sistema para executar o serviço Odoo:
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
> addons_path = /opt/odoo/odoo-server/addons, /opt/odoo/custom/addons


- Defina as respectivas permissões ao arquivo:
```sh
sudo chown odoo: /etc/odoo-server.conf
sudo chmod 640 /etc/odoo-server.conf
```

Para testar até aqui, mude para o usuario Odoo
```sh
sudo su – odoo -s /bin/bash
```

Voltar para o usuário root
```sh
exit
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
systemctl start odoo12.service
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


***Observação: Não crie o banco de dados ainda!!!***


# 3. Instalação Localização TrustCode no Odoo v12

Neste tutorial iremos utilizar o repositório da Trustcode: https://github.com/Trust-Code/odoo-brasil

- Logado como usuário "root" em seu servidor Odoo, faça a instalação das biliotecas:
```
apt-get install libjpeg-dev
apt-get install libxml2-dev libxmlsec1-dev
```

- Baixe o arquivo de dependencias pip:
```
cd /tmp
wget https://raw.githubusercontent.com/Trust-Code/odoo-brasil/12.0/requirements.txt
```

- Instale as dependências apt:
```
sudo apt-get install -y --no-install-recommends $(grep -v '^#' apt-requirements)
```

- Faça upgrade do pip:
```
sudo pip3 install --upgrade pip
```

- Instale as dependências pip:
```
sudo pip3 install -r requirements.txt
```

- Crie o diretório e subdiretório de módulos customizados:
```sh
sudo su odoo -c "mkdir -p /opt/odoo/custom/addons"
```

- Baixe o repositório Truscode do github no local correto:
```
cd /opt/odoo/custom/addons/
sudo git clone --branch 12.0 https://github.com/Trust-Code/odoo-brasil.git
sudo chown -R odoo:odoo /opt/odoo/custom/addons
```

- Adicione o repositorio "odoo-brasil" ao caminho 'addons_path' no arquivo de configuração:
```
sudo vim /etc/odoo-server.conf
```

- Adicione a seguinte linha em **'addons_path'**:
> /opt/odoo/custom/addons/odoo-brasil

- Reinicie o Odoo e confira como estão os serviços:
```
systemctl restart odoo12
systemctl status odoo12
sudo journalctl -u odoo12
cat /var/log/odoo/odoo-server.log
```

# 4. Acesso ao sistema e criação do banco de dados

- Acesse a interface web e crie o respectivo banco de dados:
```
http://<EndereçoIP>:8069
```

- Informe os respectivos parâmetros do banco de dados:
```
Master Password: <SenhaPostgres>
Database Name: <NomeBancoDados>
Email: <EmailUsuarioAdmin>
Password: <SenhaUsuarioAdmin>
Language: Portugues (BR)
Country: Brazil
```

# 5. Instalação dos módulos da localização brasileira

- Acesse o Odoo com os dados administrativos passados anteriormente:
```
Email: <EmailUsuarioAdmin>
Senha: <SenhaUsuarioAdmin>
```

- Após login, localize canto superior esquerdo, o ícone de menus principais e escolha o menu "Configurações".

-  Na janela que surge, clique na opção "Ativar o modo desenvolvedor"

- Volte ao ícone de menus principais e clique na opção "Aplicativos" e clique em "Atualizações" e em "Atualizar lista de aplicativos"

- Volte para o menu "Aplicativos" e procure por ‘br’ na barra de pesquisa para achar os módulos da localização.

- Sugerimos a instalação dos seguintes módulos:

> Contas a pagar e receber
> Faturamento
> Brazilian Localization Account
> Cash Flow Report
> Tax_Accounting
> Brazilian Localisation ZIP Codes
> Generate CNAB Files
> Pagamentos via Boleto Bancário
> Account E-Invoice
> Vinculo entre boleto e NFe
>
> Vendas  sale_management
> Sale Purchase sale_purchase
> Inventario stock
> Compra  purchase
> Contacts
> Painéis


- Os módulos a seguir  são opcionais, mas são largamente utilizados. Se desejar, também instale-os.

> Funcionários  hr (Gestão de Funcionários)
> Brazilian Localization HR
> Frota (Gestão de Frotas de veículos)
> Maintenance - HR (Manutenção de Equipamentos)


# 6 - Configurações iniciais

- Vamos habilitar as funções de contabilidade, para isso, confira se está ativo o Modo Desenvolvedor
- Após, vá até o menu [Configurações] -> [Utilizadores e Empresas] -> [Usuários]
- Edite o usuário "Administrador"
- Habilite "Mostrar todas as funções de contabilidade"


# 6.1 -  Plano de Contas

O Plano de Contas é o conjunto de todas as contas existentes em determinada empresa, ou seja, o conjunto de contas deve abranger todo tipo de fato ou acontecimento que ocorre na empresa. O Plano serve como base para que a contabilidade direcione seus trabalhos de registros referentes à organização, chamado de escrituração contábil. Um plano de contas deve ser completo, bem estruturado, condizente com as normas e preceitos contábeis aceitos.

Sua importância é fundamental, por isso sugerimos a criação de um módulo personalizado adequado para a sua empresa. Caso não possa desenvolver o respectivo módulo, sugerimos utilizar o "Plano de Contas Simplificado Brasil" (br_coa_simple) e ajustá-lo de acordo com suas necessidades.

- Volte para o menu "Aplicativos" e procure por ‘br_coa_simple’ na barra de pesquisa e proceda com a instalação do módulo.

- Após instalado o módulo, personalizado ou não e com o Modo Desenvolvedor ativo, vá em "Configurações Gerais" -> "Faturamento" e no campo "Localização Fiscal", escolha o respectivo pacote (ex: Plano de Contas Simplificado Brasil).

- Para concluir, vá até [Faturamento] -> [Configuração] -> [Plano de Contas] e revise os campos "Tipo", "Tipo de conta" e "Permite Conciliação" de todos os itens.

> Exemplos: 
> 
> Contas a Receber (Clientes)
> Tipo: A Receber
> Tipo de conta: Receita
> Permite Conciliação: Sim
> 
> 
> Clientes Recorrentes
> Tipo: Receitas
> Tipo de conta: Receita
> Permite Conciliação: Sim
> 
> Clientes Não Recorrentes
> Tipo: Receitas
> Tipo de conta: Receita
> Permite Conciliação: Sim
> 
> 
> Contas a Pagar (Fornecedores)
> Tipo: A Pagar
> Tipo de conta: Despesa
> 
> 
> Telefonia Fixa, Móvel e Internet
> Tipo: Despesas
> Tipo de conta: Despesa


# 6.2 - Definir servidor de emails SMTP:

- Ainda com o Modo Desenvolvedor ativo, vá até [Configurações] -> [Configurações Gerais] -> [External Email Servers] -> [Servidores de E-mail de Saída] e configure o seu respectivo servidor SMTP.

> Exemplo Gmail: smtp.gmail.com, <endereço@gmail.com>, SSL/TLS, 465


# 6.3 - Definir a moeda corrente

- Em [Configurações] -> [Configurações Gerais] -> [Faturamento] e defina a "Moeda" como BRL (Real Brasil)


# 6.4 - Configurar Contas Bancárias:

- Vá até [Faturamento] -> [Configuração] -> [Contas Bancárias] e edite a conta existente de acordo a sua necessidade, e se necessário, crie mais contas.

> Exemplo:
> Banco CEF
> Conta Bancária CC: xxxxxxxx-x
> Número de Conta: xxxxxxxx
> Agência: xxxx
> Titular da Conta: Nome do titular da sua conta
> Banco: CAIXA ECONOMICA FEDERAL - 104


# 6.5 - Configuração de Diários:

- Vá até [Faturamento] -> [Configuração] -> [Diários] e configure os diários que serão utilizados.

- Marcar em todos os diários, nas **"Opções Avançadas"** a função **"Permite cancelar lançamentos"**

> Exemplos:
> 
> **Faturas de Clientes**
> Tipo: Venda
> Conta de débito padrão: 1000 Contas a Receber (Clientes)
> Conta de crédito padrão: 1000 Contas a Receber (Clientes)
> ************************************************************
> 
> **Faturas de Fornecedor**
> Tipo: Compra
> Conta de débito padrão: 2000 Contas a Pagar (Fornecedores)
> Conta de crédito padrão: 12000 Contas a Pagar (Fornecedores)
> ************************************************************
> 
> **Banco CEF**
> Tipo: Banco
> Conta de débito padrão: 1031 Banco CEF
> Conta de crédito padrão: 1031 Banco CEF
> ************************************************************
> 
> **Dinheiro**
> Tipo: Dinheiro
> Conta de débito padrão: 1021 Dinheiro
> Conta de crédito padrão: 1021 Dinheiro


# 6.6 - Criação de novos usuários
O usuário administrativo já está criado, mas provavelmente será necessário a criação de outros usuários para utilização do sistema.

- Com o Modo Desenvolvedor ativo, vá até o menu [Configurações] -> [Utilizadores e Empresas] -> [Usuários] e crie os respectivos usuários para uso do sistema Odoo.

- Também é possível definir restrições de acesso. Ex: Determinado usuário não tem acesso ao módulo financeiro.
