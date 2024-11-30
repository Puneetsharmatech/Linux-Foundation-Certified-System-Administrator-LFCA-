#!/bin/bash

# ------- Function to print green and red colors
function print_color() {
    NC="\033[0m"
case $1 in
"green") COLOR="\033[0;32m" ;;
"red") COLOR="\033[0;31m" ;;
"*")COLOR="\033[[0m" ;;
esac
echo -e "${COLOR}$2 ${NC}"
}

#----------Function to print the status of service of all service ----------
function check_service_status(){
is_service_active=$(sudo systemctl is-active $1)
if [ $is_service_active = "active" ]
then 
    print_color "green" "$1 is active"
else 
    print_color "red" "$1 is not active"
    exit 1
fi 
}


# Installing Firewall
print_color "green" "Installing and configure firewall"
sudo apt install -y firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl is-active firewalld
check_service_status "firewalld"

# Installing MairaDB-server
print_color "green" "Installing and configure MariaDB server..."
sudo apt install -y mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
check_service_status "mariadb"

sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload

# creating a DB at localhost
print_color "green" "  configure DB..."
sudo cat > database-script.sql <<-EOF
CREATE DATABASE app;
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'appuser';
GRANT ALL PRIVILEGES ON *.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;
EOF

sudo mysql < database-script.sql

sudo cat > db-load-script.sql <<-EOF
USE app;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF




sudo mysql < db-load-script.sql


# Installing HTTPD webser and PHP
print_color "green" "Installing and configure HTTPD web server and PHP..."
sudo apt install -y httpd php php-mysqlnd


sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload



sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf


sudo systemctl start httpd
sudo systemctl enable httpd
check_service_status "httpd"

# installing Git
print_color "green" "Installing and cloning code from Github..."
sudo apt install -y git
sudo git clone https://github.com/Puneetsharmatech/app-ecommerce.git /var/www/html/

# Creating an .ENV file for DB
sudo cat > /var/www/html/.env <<-EOF
DB_HOST=localhost
DB_USER=appuser
DB_PASSWORD=appuser
DB_NAME=app
EOF



# Change code in index.php file
print_color "green" " replacing the DB ip with localhost in index.php file"
sudo sed -i 's#// \(.*mysqli_connect.*\)#\1#' /var/www/html/index.php
sudo sed -i 's#// \(\$link = mysqli_connect(.*172\.20\.1\.101.*\)#\1#; s#^\(\s*\)\(\$link = mysqli_connect(\$dbHost, \$dbUser, \$dbPassword, \$dbName);\)#\1// \2#' /var/www/html/index.php

print_color "green" "Updating index.php.."
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php

print_color "green" "---------------- Setup Web Server - Finished ------------------"



curl http://localhost

print_color "green" "All Set..... :)"
