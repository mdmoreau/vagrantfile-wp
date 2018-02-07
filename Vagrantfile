# -*- mode: ruby -*-
# vi: set ft=ruby :

hostname = "wp.test"
synced_host = "./"
synced_guest = "wp-content/themes/wp" # relative to /usr/share/nginx/html/

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = hostname
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder synced_host, "/usr/share/nginx/html/" + synced_guest
  config.vm.provider :virtualbox do |vb|
    vb.name = hostname
    vb.customize ["modifyvm", :id, "--memory", "1024"]
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "95"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.customize ["modifyvm", :id, "--uartmode1", "disconnected"]
  end
  config.vm.provision "shell", inline: <<-SHELL

debconf-set-selections <<< "mysql-server mysql-server/root_password password root"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password root"
debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect none"

apt-get update
apt-get -f install -y nginx mysql-server php7.0-fpm php7.0-mysql php7.0-gd php7.0-mcrypt php-ssh2 phpmyadmin

curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

sed -i 's|user www-data|user vagrant|g' /etc/nginx/nginx.conf
sed -i 's|sendfile on|sendfile off|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_vary|gzip_vary|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_proxied|gzip_proxied|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_comp_level|gzip_comp_level|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_buffers|gzip_buffers|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_http_version|gzip_http_version|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_types|gzip_types image/svg+xml|g' /etc/nginx/nginx.conf
sed -i 's|user = www-data|user = vagrant|g' /etc/php/7.0/fpm/pool.d/www.conf
sed -i 's|group = www-data|group = vagrant|g' /etc/php/7.0/fpm/pool.d/www.conf
sed -i 's|post_max_size = 8M|post_max_size = 256M|g' /etc/php/7.0/fpm/php.ini
sed -i 's|upload_max_filesize = 2M|upload_max_filesize = 256M|g' /etc/php/7.0/fpm/php.ini

cd /etc/nginx/sites-available
rm default
touch default
cat >> default << 'EOF'
server {
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;
  root /usr/share/nginx/html;
  index index.php index.html index.htm;
  server_name localhost;
  client_max_body_size 256m;
  location / {
    try_files $uri $uri/ /index.php?$args;
  }
  rewrite /wp-admin$ $scheme://$host$uri/ permanent;
  location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
    access_log off;
    log_not_found off;
    expires max;
  }
  location ~ [^/]\.php(/|$) {
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    if (!-f $document_root$fastcgi_script_name) {
      return 404;
    }
    fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}
EOF

mysql -u root -proot -e "CREATE DATABASE wordpress; GRANT ALL PRIVILEGES ON wordpress.* TO username@localhost IDENTIFIED BY 'password'"

ln -s /usr/share/phpmyadmin /usr/share/nginx/html
phpenmod mcrypt

service nginx reload
service php7.0-fpm restart

chown vagrant:vagrant /usr/share/nginx/html /usr/share/nginx/html/wp-content /usr/share/nginx/html/wp-content/plugins /usr/share/nginx/html/wp-content/themes

rm /usr/share/nginx/html/index.html
su - vagrant -c "wp core download --path=/usr/share/nginx/html"

  SHELL
end
