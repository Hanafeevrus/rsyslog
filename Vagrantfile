# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.vm.box_check_update = true
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
    set -x
    	yum update
	sudo yum install rsyslog -y
	sudo yum install audispd-plugins.x86_64 -y
	sudo yum install net-tools -y
       sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config	
sudo service sshd restart
  SHELL

  config.vm.define "web", primary: true do |s|
    s.vm.hostname = 'web-server'
    s.vm.network "private_network", ip: "192.168.111.15"
    s.vm.provision "shell", inline: <<-SHELL
	yum install epel-release -y
	sudo yum install nginx -y  
	sudo cp /vagrant/nginx.conf /etc/nginx/nginx.conf
	sudo systemctl start nginx
	sudo systemctl enable nginx
	sudo cp /vagrant/audit.rules /etc/audit/rules.d/audit.rules
	sudo cp /vagrant/audisp_client/audisp-remote.conf  /etc/audisp/audisp-remote.conf
	sudo cp /vagrant/audisp_client/au-remote.conf /etc/audisp/plugins.d/au-remote.conf
	sudo cp /vagrant/crit.conf /etc/rsyslog.d/crit.conf
	sudo cp /vagrant/rsyslog_client/rsyslog.conf /etc/rsyslog.conf
	sudo service auditd restart
        sudo service rsyslog restart
	sudo systemctl enable auditd
        sudo systemctl enable rsyslog
    SHELL
  end

  config.vm.define "log" do |c|
    c.vm.hostname = 'log-server'
    c.vm.network "private_network", ip: "192.168.111.16"
    c.vm.provision "shell", inline: <<-SHELL
	sudo firewall-cmd --permanent --add-port=514/{tcp,udp}
       	sudo firewall-cmd --reload
	sudo cp /vagrant/rsyslog_server/rsyslog.conf /etc/rsyslog.conf
	sudo cp /vagrant/auditd.conf /etc/audit/auditd.conf
	sudo service auditd restart
	sudo service rsyslog restart
	sudo systemctl enable rsyslog
	sudo systemctl enable auditd
    SHELL
   end
  end
