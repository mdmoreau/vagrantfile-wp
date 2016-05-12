# -*- mode: ruby -*-
# vi: set ft=ruby :

hostname = "wp.dev"
synced_host = "./dist"
synced_guest = "wp-content/themes/wp" # relative to /usr/share/nginx/html/

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = hostname
  config.vm.synced_folder synced_host, "/usr/share/nginx/html/" + synced_guest, owner: "www-data", group: "www-data"
  config.vm.provider :virtualbox do |vb|
    vb.name = hostname
    vb.customize ["modifyvm", :id, "--memory", "1024"]
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "95"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end
  config.vm.provision "shell", inline: <<-SHELL

debconf-set-selections <<< "mysql-server mysql-server/root_password password root"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password root"

apt-get update
apt-get -f install -y nginx mysql-server php5-fpm php5-mysql php5-gd libssh2-php unzip

sed -i 's|sendfile on|sendfile off|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_vary|gzip_vary|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_proxied|gzip_proxied|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_comp_level|gzip_comp_level|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_buffers|gzip_buffers|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_http_version|gzip_http_version|g' /etc/nginx/nginx.conf
sed -i 's|# gzip_types|gzip_types image/svg+xml|g' /etc/nginx/nginx.conf

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
  client_max_body_size 2m;
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
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}
EOF

mysql -u root -proot -e "CREATE DATABASE wordpress; GRANT ALL PRIVILEGES ON wordpress.* TO username@localhost IDENTIFIED BY 'password'"

cd /usr/share/nginx/html
rm 50x.html index.html
wget "https://wordpress.org/latest.zip"
unzip latest.zip
rm latest.zip
cp -a wordpress/. .
rm -rf wordpress

chown -R www-data:www-data /usr/share/nginx/html
find /usr/share/nginx/html -type d -exec chmod 755 {} +
find /usr/share/nginx/html -type f -exec chmod 644 {} +
chmod g+s /usr/share/nginx/html

service nginx reload

  SHELL
end
