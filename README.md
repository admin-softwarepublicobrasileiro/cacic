Este roteiro irá orientá-lo como fazer uma NOVA instalação do Gerente Cacic 3.0 em máquinas com Sistema Operacional Ubuntu e Debian.

A instalação ocorre basicamente por linhas de comando através do Terminal, para tanto, acesse o Painel Inicial (Launcher) que fica localizado no canto superior esquerdo de sua tela.

Após aparecer a opção de busca digite "Terminal" e pressione a tecla "Enter". Agora seguiremos com os comandos dentro do Terminal.

Caso não o encontre utilize as teclas de atalho “CTRL + ALT + T”.

## 1 - Terminal

Observação sobre o uso do terminal:

Dentro do terminal o cursor ficará sempre depois de "$" ou "#".

Sempre que o comando a ser copiado for precedido por "$", significa que este é um comando de usuário normal;

Sempre que o comando a ser copiado for precedido por "#", significa que este é um comando de usuário “root”.

Caso o comando a ser copiado não seja precedido por "$" nem por "#", significa que este comando pode ser executado sem restrições.

Para acessar como “root” digite "sudo su".

Foi utilizado para este tutorial o “Terminal” em idioma inglês, então as confirmações apresentadas aqui estão em (Yes/Y ou No/N), caso seu sistema esteja em português confirme com (Sim/S ou Não/N)


## 2 - Instalando os Pacotes necessários:

Instale os pacotes que você vai precisar:

apt-get -y install postgresql apache2 php5 php5-pgsql php5-gd php5-mcrypt libapache2-mod-php5 php5-ldap php-pear subversion  git  openjdk-7-jre  

 Com este comando irá instalar todos os pacotes necessários.

## 3 – Configurando o PostgreSQL:

O arquivo "php.ini" vem com fuso horário da Europa, logo precisamos configurá-lo para o Brasil. 

Edite o arquivo "php.ini" através do comando abaixo:

nano /etc/php5/apache2/php.ini 

Quando o arquivo abrir digite "CTRL + W" para abrir a ferramenta de busca e digite "Module Settings" 

Você verá o comando abaixo:

[Date] 

; Defines the default timezone used by the date functions

; http://php.net/date.timezone

Na linha imediata abaixo digite: 

date.timezone = America/Sao_Paulo 

Em alguns casos, pode ser que já tenha na linha ";date.timezone =", neste caso complete com "America/Sao_Paulo".

Nã esqueça de remover o "ponto e virgula" 

Caso já esteja atualizado, continue.

Digite "CTRL + X" para salvar, 

Confirme a alteração com "Y + Enter"

Como "root" reinicie o Apache.

# /etc/init.d/apache2 restart 


## 4 - Montando ambiente de desenvolvimento

Baixe o código do repositório oficial (necessário o "subversion", que foi previamente instalado) 

Após instalação do “subversion” execute os comandos abaixo: 

# cd /srv 

# svn --username SEU_USER_DO_PORTAL co http://svn.softwarepublico.gov.br/svn/cacic/cacic/tags/3.0b1/gerente

# chown -R www-data.www-data gerente 

Crie um link simbólico da sua pasta web para o Apache 

# ln -s /srv/gerente/web /var/www/cacic 


## 5 - Crie banco de dados para o Symfony - PostgreSQL

(É possível que já exista o banco de dados criado, caso isso ocorra passe para o item 5.0).

Execute os seguintes comandos no terminal:

$ sudo su 

# su - postgres

$ createuser -D -R -S -w cacic

Se você pretende usar o modulo do importador, substitua o "-S" por "-s"

Crie o banco com o comando abaixo: 

$ createdb -w -O cacic cacic 


## 5.1 - Liberando acesso ao Banco:

Edite o arquivo "/etc/pg_hba.conf":

  # nano /etc/postgresql/9.1/main/pg_hba.conf 

  Procure as linhas abaixo. (estão logo no início do texto) 

  # PostgreSQL Client Authentication Configuration File

  # ===================================================

  #

  # Refer to the "Client Authentication" section in the PostgreSQL

  # documentation for a complete description of this file. A short

  # synopsis follows.

  #

  # This file controls: which hosts are allowed to connect, how clients
  
  # are authenticated, which PostgreSQL user names they can use, which

  # databases they can access. Records take one of these forms:

  #

  # local DATABASE USER METHOD [OPTIONS]

  # host DATABASE USER ADDRESS METHOD [OPTIONS]
  
  # hostssl DATABASE USER ADDRESS METHOD [OPTIONS]

  # hostnossl DATABASE USER ADDRESS METHOD [OPTIONS]

  Agora, acrescente as próximas linhas. Sem o “#” 

  host cacic cacic 127.0.0.1/32 trust  

  host cacic cacic localhost trust  

Digite "CTRL + X" para sair, confirme com "y" e "enter".  

Reinicie o banco de dados:

  $ /etc/init.d/postgresql restart 


Execute a linha a baixo e verifique se a mesma se encontra igual ao exemplo: 

  $ psql -U cacic -h localhost cacic  

psql (9.1.9)

SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)

Type "help" for help.

cacic=>

Digite "\q", depois "exit"

  $ exit


## 5.2 - Configurando o arquivo parameters.yml

Abra o arquivo "parameters.yml" conforme o comando abaixo:

# nano /srv/gerente/app/config/parameters.yml

Adicione as seguintes linhas: (este arquivo conterá somente essas linhas) 

parameters: 

     database_driver: pdo_pgsql

     database_host: 127.0.0.1

     database_port: null

     database_name: cacic

     database_user: cacic

     database_password: null

     mailer_transport: smtp

     mailer_host: 127.0.0.1

     mailer_user: null

     mailer_password: null

     locale: pt_BR

     secret: d7c123f25645010985ca27c1015bc76797

     database_path: null

Digite "CTRL+X" para fechar 

Confirme com "Y + Enter"


## 5.3 - Executando comandos do Symfony

Execute os comandos do symfony necessários para o sistema funcionar: 

  # su - www-data  

  $ bash

  $ cd /srv/gerente

Instalação dos vendors 

  $ php composer.phar install 

Aguarde o fim da instalação (este processo pode levar alguns minutos) 

Digite o comando "exit" e depois digite o mesmo comando "exit" novamente. 

Carregando os assets: (necessário haver o "java" instalado) 
php app/console doctrine:schema:update --force

  php app/console assets:install --symlink

  php app/console assetic:dump

  

## 5.4 - Carregando dados iniciais 

  # php app/console doctrine:fixtures:load 

Digite o comando "exit" e depois digite o mesmo comando "exit" novamente. 

Caso apareça a mensagem:

Could not open input file: app/console

finalise o terminal com "exit"

Terminada a instalação e configuração do Gerente Cacic 3.0, execute o navegador.

----------------------------------------------------------------------------------------------------------- 

ABRA O NAVEGADOR:


## 6 - Abrindo a tela de login do Gerente Cacic 3.0 

Digite:

  http://localhost/cacic/

Pressione "enter".

Clique em app.php

Entre com o usuário e a senha.

  Usuário: admin

  Senha: 123456  
