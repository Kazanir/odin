# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "odin"
  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network", ip: "192.168.2.33"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "../www", "/var/www"
  config.vm.synced_folder "../code", "/home/vagrant/code"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]  
    vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", privileged: false, inline: <<-SHELL

# Debian packaging prep
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password mysql'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password mysql'

echo "deb http://archive.ubuntu.com/ubuntu trusty multiverse
deb http://archive.ubuntu.com/ubuntu trusty-updates multiverse
deb http://security.ubuntu.com/ubuntu trusty-security multiverse" | sudo tee -a /etc/apt/sources.list > /dev/null

sudo apt-get update -y

# Install every package under the sun but not Perl
sudo apt-get install -y accountsservice adduser apache2-mpm-worker apache2-utils apparmor apt apt-transport-https apt-utils autoconf automake bash bash-completion build-essential bzip2 ca-certificates cmake coreutils default-jre dos2unix dpkg e2fsprogs ed eject findutils gcc-4.8 geoip-database git git-flow gitk glances gnupg gpgsm gpgv grep iperf less libapache2-mod-fastcgi libmcrypt-dev libmemcached-dev libmysqlclient-dev libtool makedev man-db manpages mawk memcached mime-support mlocate module-init-tools mount mountall mtr multiarch-support mysql-common mysql-server ncdu ncurses-base ncurses-bin ocaml ocaml-native-compilers openntpd openssh-client openssh-server openssl php5 php5-cli php5-curl php5-dbg php5-fpm php5-gd php5-ldap php5-mcrypt php5-mysql php5-memcache php5-pgsql php5-redis php5-xdebug putty-tools redis-server resolvconf rsync rsyslog samba screen sed siege solr-tomcat tcpdump tmux traceroute udev ufw unixodbc-dev vagrant vim vim-common vim-tiny wget xml-core zsh 

# A bit of permissions stupidity
sudo usermod -a -G www-data vagrant
sudo chown -R www-data:www-data /var/log/apache2
sudo chown www-data:www-data /var/log/php5-fpm.log

# Easy MySQL access
mysql -u root -pmysql -e "CREATE USER 'vagrant'@'localhost';"
mysql -u root -pmysql -e "GRANT ALL ON *.* TO 'vagrant'@'localhost';"
mysql -u root -pmysql -e "FLUSH PRIVILEGES;"

# Set up Apache
sudo a2dismod mpm_event
sudo a2enmod alias actions vhost_alias rewrite mpm_worker ssl socache_shmcb proxy
sudo cp /vagrant/apache2/odin.conf /etc/apache2/sites-available
sudo a2ensite odin
sudo apache2ctl restart

# Set up PHP
sudo cp /vagrant/php5/php.ini /etc/php5/fpm/php.ini
sudo php5enmod mcrypt pdo_mysql mysql mysqli
sudo service php5-fpm restart

# Restart Apache for good measure. No, I don't know why.
sudo apache2ctl restart

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Drupal build tools
composer global require drush/drush:6.*
composer global require platformsh/cli:1.*
echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> /home/vagrant/.bashrc
echo '. "$HOME/.composer/vendor/platformsh/cli/platform.rc" 2>/dev/null' >> /home/vagrant/.bashrc

# Install my Vim config because it is awesome
git clone https://github.com/Kazanir/dotvim.git ~/.vim
ln -s ~/.vim/.vimrc ~/.vimrc
cd ~/.vim
git submodule init
git submodule update

SHELL
end
