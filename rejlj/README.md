IMAGE_NAME = "bento/debian-12" //instalace systému

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end

    config.vm.define "debian" do |debian|
        debian.vm.box = IMAGE_NAME
	debian.vm.network "forwarded_port", guest: 22, host: 2206, host_ip: "127.0.0.1"
	debian.vm.network "forwarded_port", guest: 80, host: 8006, host_ip: "127.0.0.1"
        debian.vm.hostname = "debian" //portování a port setup
	debian.vm.provision "shell", inline: "sudo apt-get update", run: "always"
	debian.vm.provision "shell", inline: "sudo apt install -y apache2", run: "always"
	debian.vm.provision "shell", inline: "sudo apt install -y mariadb-server", run: "always"
    end //instalace apt-balíčků

    config.vm.provision "file", source: "id_ed25519.pub", destination: "~/.ssh/me.pub" //automatické přihlášení přes id
    config.vm.provision "shell", inline: <<-SHELL
    cat /home/vagrant/.ssh/me.pub >> /home/vagrant/.ssh/authorized_keys

    SHELL

#     config.vm.provision "shell", path: "install-zabbix-agent2.sh" //zabbix agent + server install

#     config.vm.provision "shell", path: "configure-zabbix-agent2.sh" //zabix config

end

----------------------------------------------------------------------------------------------------------------------
//zabbix instalace

#!/usr/bin/env bash

sudo apt-get install -y net-tools

sudo wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
sudo dpkg -i zabbix-release_latest_7.0+debian12_all.deb
sudo apt update

sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

sudo apt install zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql
//instalace zabbix agent2 a serveru + pluginů

//tvoření databáze na ukládání zachicených dat
mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;

zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;

# EOF

----------------------------------------------------------------------------------------------------------------------
//configurace zabbix

#!/usr/bin/env bash

DBPassword=testword

# systemctl restart zabbix-server zabbix-agent2 apache2
# systemctl enable zabbix-server zabbix-agent2 apache2

# EOF