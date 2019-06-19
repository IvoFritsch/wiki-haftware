====== Preparando um Droplet para rodar os sistemas da Haftware ======

Após criar o Droplet na Digital Ocean, acessar com SSH e seguir os seguintes passos:

==== Instalando o Java ====

<code>
sudo apt-get update
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
</code>

Agora, se tudo ocorreu corretamente, o Java estará instalado, para testar isso, rodar o comando:

<code>
java -version
</code>
Esse comando deverá exibir a versão do Java instalada.

==== Instalando o Glassfish ====
Agora que o java está instalado, podemos instalar o Glassfish:
<code>
wget http://download.oracle.com/glassfish/4.1.2/release/glassfish-4.1.2.zip
apt-get install unzip
unzip glassfish-4.1.2.zip -d /opt
</code>

Se tudo ocorreu OK, o Glassfish agora está instalado, não rodar ele ainda pois precisamos de mais algumas coisas.

==== Instalando o HSQLDB Manager ====
O [[https://github.com/IvoFritsch/hsqldb-manager|HSQLDB Manager]] serve para facilitar a criação e instalação de bancos de dados HSQL, para instalá-lo:
<code>
wget https://github.com/IvoFritsch/hsqldb-manager/raw/master/hsqldb-manager.zip
unzip hsqldb-manager.zip -d /opt/hsqldb-manager
chmod +x /opt/hsqldb-manager/hsqlman
</code>

Se tudo ocorreu OK, o HSQLDB Manager agora está instalado, não rodar ele ainda pois precisamos de mais algumas coisas.
==== Adicionando o Glassfish e o HSQLDB Manager ao PATH ====
Para adicionar os dois diretorios criados anteriormente ao PATH, basta rodar o comando:
<code>
echo 'export PATH=/opt/glassfish4/bin:/opt/hsqldb-manager:$PATH' >> .profile
</code>

==== Adicionando o HSQLDB Manager e o Glassfish à inicialização ====
Para que o Glassfish e os Bancos de dados rodem ao inicializar o servidor, rodar os comandos abaixo:
<code>
touch /etc/init.d/haftware-init
echo '#!/bin/bash' >> /etc/init.d/haftware-init
echo '/opt/hsqldb-manager/hsqlman start' >> /etc/init.d/haftware-init
echo '/opt/glassfish4/bin/asadmin start-domain' >> /etc/init.d/haftware-init
chmod +x /etc/init.d/haftware-init
update-rc.d haftware-init defaults 100
</code>

Após isso feito, reiniciar a janela do SSH e continuar o tutorial...
==== Continuar... ====