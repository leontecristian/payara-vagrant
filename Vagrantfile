# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "generic/ubuntu1804"
  

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 8080, host: 18080
  config.vm.network "forwarded_port", guest: 8081, host: 18081
  
  config.vm.network "forwarded_port", guest: 5432, host: 15432

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  config.vm.network "forwarded_port", guest: 4848, host: 14848, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./data", "/vagrant_data", create: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  
	 vb.name = "payara_vm"
	
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
  
     # Customize the amount of memory on the VM:
     vb.memory = "2560"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    
	PAYARA_HOME=/opt/payara/
	#
	PAYARA_ED=https://repo1.maven.org/maven2/fish/payara/distributions/payara/5.2021.7/payara-5.2021.7.zip
	#PAYARA_ED=https://repo1.maven.org/maven2/fish/payara/distributions/payara/5.2020.7/payara-5.2020.7.zip
	#PAYARA_ED=https://repo1.maven.org/maven2/fish/payara/distributions/payara/5.201/payara-5.201.zip
	#PAYARA_ED=https://repo1.maven.org/maven2/fish/payara/distributions/payara/5.191/payara-5.191.zip
	
	echo "Configuring operating system"
    add-apt-repository -y ppa:openjdk-r/ppa
	echo "running update..."
	apt-get -qqy update
	echo "installing unzip"
	apt-get -qqy install unzip
	echo "installing OpenJDK-8"
	apt-get -qqy install openjdk-8-jdk
	
	echo "Provisioning Payara5 $PAYARA_ED to $PAYARA_HOME"
	echo "Downloading Payara"
	
	wget --progress=bar:force $PAYARA_ED -O temp.zip
	mkdir -p $PAYARA_HOME
	echo "Unzip ..."
	unzip -qq temp.zip -d $PAYARA_HOME
	rm temp.zip
	
	echo "Enabling secure admin mode for domains (u/p = admin/payara0payara)"
	PWDFILE=/tmp/pwdfile
	TMPFILE=/tmp/tmpfile
	DOMAINS_DIR="${PAYARA_HOME}/payara5/glassfish/domains"
	ADMIN_USER=admin
	ADMIN_PASSWORD=payara0payara
	USER_NAME=vagrant
	USER_PASSWORD=vagrant_1234
	echo "AS_ADMIN_PASSWORD=\nAS_ADMIN_NEWPASSWORD=${ADMIN_PASSWORD}" > ${TMPFILE}
	echo "AS_ADMIN_PASSWORD=${ADMIN_PASSWORD}\nAS_ADMIN_USERPASSWORD=${USER_PASSWORD}" > ${PWDFILE}
	
	for DOMAIN in domain1; do
           # echo "admin;{SSHA256}Rzyr/2/C1Zv+iZyIn/VnL0zDYESs8nTH8t/OMlOpazehMGn5L9ejkg==;asadmin" > "${DOMAINS_DIR}/${DOMAIN}/config/admin-keyfile"
           ${PAYARA_HOME}/payara5/bin/asadmin --user ${ADMIN_USER} --passwordfile "${TMPFILE}" change-admin-password --domain_name=${DOMAIN}
		   ${PAYARA_HOME}/payara5/bin/asadmin start-domain ${DOMAIN}
           ${PAYARA_HOME}/payara5/bin/asadmin --user admin --passwordfile "${PWDFILE}" enable-secure-admin
		   ${PAYARA_HOME}/payara5/bin/asadmin --user admin --passwordfile "${PWDFILE}" create-file-user ${USER_NAME}
           ${PAYARA_HOME}/payara5/bin/asadmin stop-domain ${DOMAIN}
    done
	#rm "${PWDFILE}"
	
	echo "Setting ownership of ${PAYARA_HOME} content"
	chown -R vagrant:vagrant $PAYARA_HOME         # Make sure vagrant owns dir
	
	echo "use the following command to start server"
	echo "${PAYARA_HOME}/payara5/bin/asadmin start-domain domain1"
	echo "use this link from the host machine http://localhost:14848/"
	
	echo "Install POSTGRES 13"
	sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
	wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
	sudo apt-get update
	sudo apt-get -y install postgresql-13

	echo "listen_addresses = '*'" | sudo tee -a /etc/postgresql/13/main/postgresql.conf
	echo "host  all  all 0.0.0.0/0 md5" | sudo tee -a /etc/postgresql/13/main/pg_hba.conf
	sudo service postgresql restart

	echo "Creating vagrant user and database"
	echo "ALTER USER postgres PASSWORD 'Welcome1'" | sudo -u postgres psql -a -f -
	
  SHELL
end
