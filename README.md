# Desafio1_SRE
Repository create for studies for career Sys Engineer 

<h2>Trilha DevOps</h2>

<h3>1 - Criar um WebServer CentOS na AWS. Segue tutorial (by darrenoneill)</h3>

https://darrenoneill.eu/?p=816

![aws1](https://user-images.githubusercontent.com/45441463/95772177-b5d00700-0c92-11eb-802b-2a8672f41577.PNG)

<h3>2 - Criar uma loja com Magento (step by step)</h3>

2.1- Antes de mais nada você necessitará de um LEMP para iniciar a instalação do Magento, caso tenha dúvidas de como proceder, tem esse outro post que eu já havia feito:
https://github.com/rosariofe/LAMP-CentOS

2.2 - Após realizar a instalação do LEMP, podemos proceder, iremos editar o seu arquivo php.ini.

- Acesse o arquivo via VIM, utilizando o comando:

```
vim /etc/php.ini
```
- Procure pelas seguintes linhas e as altere.

```
memory_limit =512M
upload_max_filesize = 200M
zlib.output_compression = On 
max_execution_time = 300 
date.timezone = Asia/Kolkata
``` 

- Salve e feche o arquivo.

<h3>3 - Configurar o banco de dados</h3>

3.1 - Para iniciar utilize o script de instalação:

```
mysql_secure_installation
```
Responsa as devidas perguntas:
```
Enter current password for root (enter for none): 
Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y

```
3.2 - Após o término da instalação acesse o banco de dados:

```
mysql -u root -p

``` 
3.3 Agora, você irá criar o banco de dados e realizar as devidas alterações de acesso.

```
CREATE DATABASE magentodb;
GRANT ALL ON magentodb.* TO magento@localhost IDENTIFIED BY 'password';

``` 

3.4 Agora, você irá forçar os privilégios criados utilizando o comando:

```
FLUSH PRIVILEGES;
EXIT;
```
<h3>4 - Configurar PHP-FPM para Magento</h3>

4.1 Após configurar o banco de dados, você precisará configurar o seu pool de PHP-FPM para o Magento, crie o seguinte arquivo:

```
vim /etc/php-fpm.d/magento.conf

```
4.2 Adicione as seguintes linhas:

```
[magento] 
user = nginx 
group = nginx 
listen.owner = nginx 
listen.group = nginx 
listen = /run/php-fpm/magento.sock 
pm = ondemand 
pm.max_children = 50 
pm.process_idle_timeout = 10s 
pm.max_requests = 500 
chdir = /

```
4.3 Salve e feche o arquivo, e dê um restart no serviço:

```
systemctl restart php-fpm

```
<h3>5 - Agora sim! Iremos baixar o Magento </h3>

5.1 Primeiro você irá baixar a versão mais recento do git, acesse:
```
cd /var/www/html
wget https://github.com/magento/magento2/archive/2.3.zip

```
5.2 Após o download, descompacte os arquivos:

```
unzip 2.3.zip
```
5.3 Agora você precisa instalar o Composer, é ele quem vai instalar as dependências do PHP para o Magento funcionar corretamente, você pode instalar o Composer com o seguinte comando:

```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer

```
5.4 Agora, dentro do diretório /var/www/html/magento2, você irá realizar a instalação do Magento com os seguintes comandos:

```
composer update
```
5.4.1 Após o update ser realizado utilize o comando:

```
composer install

```
5.5 Agora, dê as permissões necessários para funcionar

```
chown -R nginx:nginx /var/www/html/magento2
chmod -R 755 /var/www/html/magento2

```

<h3>6. Pronto, Magento instalado, iremos configurar o Nginx para funcionar corretamente, acesse o arquivo:</h3>

```
nano /etc/nginx/conf.d/magento.conf

```
6.1 Adicione as seguintes linhas:

```
upstream fastcgi_backend {
  server   unix:/run/php-fpm/magento.sock;
}

server {
    listen 80;
    server_name (nome do seu dns);

    set $MAGE_ROOT /var/www/html/magento2;
    set $MAGE_MODE developer;

    access_log /var/log/nginx/magento-access.log;
    error_log /var/log/nginx/magento-error.log;

    include /var/www/html/magento2/nginx.conf.sample;
}

``` 

OBS: Lembre-se de alterar os devidos valores para a sua instalação, salve e feche o arquivo.

6.2 Realize o reínicio dos serviços

```
systemctl restart php-fpm
systemctl restart nginx

```

<h3>7 - Configurar o SELINUX e o Firewall</h3>

Dependências:

* Semanage instalado, comando: 

```
yum whatprovides semanage
yum install -y policycoreutils-python-utils
```
* FirewallD instalado, comando:

```
sudo yum install firewalld
```

7.1 Com as dependências instaladas, podemos configurar o SELINUX, utilize o comando:
```
semanage permissive -a httpd_t

semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2(/.*)?'

```
7.2 Em seguida, você precisará, criar uma regra de firewall para permitir o HTTP e o HTTPS, com o seguinte comando:

``` 
firewall-cmd --permanent --add-service=http

firewall-cmd --permanent --add-service=https
firewall-cmd --reload

```
8.Pronto, seu magento está pronto e funcionando! 

![magento](https://user-images.githubusercontent.com/45441463/95772468-28d97d80-0c93-11eb-98ca-000035452006.PNG)


9. Agora que temos nossa Loja, montaremos um site em WordPress.

9.1 Como já temos o LEMP e toda a estrutura criada, será bem rápido, acesse o banco de dados:

```
mysql -u root -p 

```
9.2Você irá criar o banco de dados wordpress:

```
CREATE DATABASE wordpress;

CREATE USER wordpressuser@localhost IDENTIFIED BY 'passoword';

```
9.3 Libere os privilégios para o usuário

```
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';

FLUSH PRIVILEGES;

exit
```

9.4 Instalando o WordPress

- Comece instalando os modúlos PHP:
```
sudo yum install php-gd

```
- Reinicie o serviço

```
sudo service httpd restart
```

- Acesse o diretório /var/www/html/ e realize o wget:

```
wget http://wordpress.org/latest.tar.gz

```
- Descompacte o arquivo

```
tar xzvf latest.tar.gz

```
- Dê as seguintes permissões
```
sudo chown -R nginx:nginx /var/www/html/wordpress

```
- Acesse o diretório em que ele foi instalado e irá copiar os arquivo padrão de configuração:

```
cp wp-config-sample.php wp-config.php

```

- Iremos editar o arquivo:

```
vim nano wp-config.php

```
- Os únicos valores que serão alterados são:

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
```

<h4>10. Pronto, seu WordPress está instado, agora partiremos para a criação de um site simples em php.<h4>




