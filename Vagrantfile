# -*- mode: ruby -*-
# vi: set ft=ruby :

# options
hostname = "wp.test"
synced_host = "."
synced_guest = "/var/www/html/wp-content/themes/wp"

# setup script
$script = <<SCRIPT

# disable debconf interactive frontend
export DEBIAN_FRONTEND=noninteractive

# set mysql install options
debconf-set-selections <<< "mysql-server mysql-server/root_password password root"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password root"

# set phpmyadmin install options
debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect none"

# install nginx, mysql, php and phpmyadmin
apt-get update
apt-get -f -q -y install nginx mysql-server php-fpm php-mysql php-gd php-mcrypt php-ssh2 phpmyadmin

# install wp-cli
curl -s -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

# run nginx and php as vagrant user
sed -i 's|user www-data|user vagrant|g' /etc/nginx/nginx.conf
sed -i 's|user = www-data|user = vagrant|g' /etc/php/7.0/fpm/pool.d/www.conf
sed -i 's|group = www-data|group = vagrant|g' /etc/php/7.0/fpm/pool.d/www.conf

# edit nginx config
sed -i 's|sendfile on|sendfile off|g' /etc/nginx/nginx.conf

# edit php config
sed -i 's|post_max_size = 8M|post_max_size = 256M|g' /etc/php/7.0/fpm/php.ini
sed -i 's|upload_max_filesize = 2M|upload_max_filesize = 256M|g' /etc/php/7.0/fpm/php.ini

# add nginx default site config
cat <<'EOF' > /etc/nginx/sites-available/default
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  root /var/www/html;
  index index.php index.html index.htm index.nginx-debian.html;
  server_name localhost;
  client_max_body_size 256m;
  location / {
    try_files $uri $uri/ /index.php?$args;
  }
  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
  }
}
EOF

# create wordpress database
mysql -uroot -proot -e "CREATE DATABASE wordpress; GRANT ALL PRIVILEGES ON wordpress.* TO username@localhost IDENTIFIED BY 'password';"

# symlink phpmyadmin to html root
ln -s /usr/share/phpmyadmin /var/www/html
phpenmod mcrypt

# refresh nginx and php-fpm
service nginx reload
service php7.0-fpm restart

# set vagrant as owner of html root
chown vagrant:vagrant /var/www/html /var/www/html/wp-content /var/www/html/wp-content/plugins /var/www/html/wp-content/themes

# install wordpress
su - vagrant -c "wp core download --path=/var/www/html"

SCRIPT

# vagrant config
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.hostname = hostname
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder synced_host, synced_guest
  config.vm.provider "virtualbox" do |v|
    v.name = hostname
  end
  config.vm.provision "shell", inline: $script
end
