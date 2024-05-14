# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Configurar el sistema operativo
  config.vm.box = "ubuntu/focal64"

  # Configurar el hostname
  config.vm.hostname = "levantamiento.test"

  # Configurar la red privada
  config.vm.network "private_network", ip: "192.168.33.15"

  # Configurar el folder sincronizado
  config.vm.synced_folder "./html", "/var/www/html", mount_options: ["dmode=777", "fmode=777"]

  # Configurar la cantidad de RAM y CPUs
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 6
  end

  # Provisionar la VM
  config.vm.provision "shell", inline: <<-SHELL
    # Actualizar paquetes
    apt-get update
    apt-get upgrade -y

    # Instalar Apache
    apt-get install -y apache2

    # Configurar Apache para permitir Override en .htaccess
    sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

    # Habilitar mod_rewrite
    a2enmod rewrite

    # Instalar PHP 8.2 y extensiones necesarias
    add-apt-repository ppa:ondrej/php -y
    apt-get update
    apt-get install -y php8.2 php8.2-cli php8.2-fpm php8.2-mysql php8.2-xml php8.2-mbstring php8.2-zip php8.2-curl php8.2-gd php8.2-bcmath libapache2-mod-php8.2 php8.2-pdo php8.2-pdo-mysql

    # Instalar Composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

    # Instalar Node.js y npm
    curl -sL https://deb.nodesource.com/setup_18.x | bash -
    apt-get install -y nodejs

    # Instalar MariaDB
    apt-get install -y mariadb-server mariadb-client

    # Configurar MariaDB para escuchar en todas las interfaces
    sed -i 's/^bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
    systemctl restart mariadb

    # Configurar MariaDB
    mysql -u root -e "CREATE DATABASE vagrant;"
    mysql -u root -e "GRANT ALL PRIVILEGES ON vagrant.* TO 'vagrant'@'%' IDENTIFIED BY 'vagrant';"
    mysql -u root -e "FLUSH PRIVILEGES;"

    # Ajustar el firewall para permitir el acceso a MariaDB (Puerto 3306)
    ufw allow 3306

    # Instalar y configurar Tailwind CSS
    npm install -g tailwindcss

    # Instalar unzip y 7z
    apt-get install -y unzip p7zip-full

    # Crear el directorio para el proyecto Laravel
    mkdir -p /var/www/html/levantamiento/public

    # Configurar VirtualHost
    cat <<EOF > /etc/apache2/sites-available/levantamiento.test.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName levantamiento.test
    DocumentRoot /var/www/html/levantamiento/public

    <Directory /var/www/html/levantamiento/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOF

    # Habilitar el nuevo VirtualHost
    a2ensite levantamiento.test.conf

    # Deshabilitar el sitio por defecto
    a2dissite 000-default.conf

    # Reiniciar Apache para aplicar los cambios
    systemctl restart apache2

    # Instalar la extensión PDO MySQL
    apt-get install -y php8.2-pdo php8.2-pdo-mysql

    # Reiniciar Apache nuevamente para cargar las nuevas extensiones PHP
    systemctl restart apache2
  SHELL
end
