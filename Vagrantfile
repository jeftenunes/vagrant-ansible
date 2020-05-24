$script_mysql = <<-SCRIPT
  apt-get update && \
  apt-get install -y mysql-server-5.7 && \
  mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT

$script_ansible = <<-SCRIPT
  apt-get -y update
  apt-get -y install software-properties-common
  apt-add-repository --yes --update ppa:ansible/ansible
  apt-get install -y ansible
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.define "php" do |web|
    web.vm.network "forwarded_port", guest: 8888, host: 8888
    web.vm.network "public_network", ip: "192.168.1.25"
    web.vm.provision "shell", inline: "sudo apt-get update && sudo apt-get install -y puppet"
    web.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "./configs/manifests"
      puppet.manifest_file = "web.pp" 
    end
  end

  config.vm.define "mysqlserver" do |server|
    server.vm.network "public_network", ip: "192.168.1.28"
    server.vm.provision "shell", inline: "cat /vagrant/configs/id_bionic.pub >> .ssh/authorized_keys"
  end

  config.vm.define "ansible" do |ansible|
    ansible.vm.network "public_network", ip: "192.168.1.29"

    ansible.vm.provision "shell",
      inline: "cp /vagrant/id_bionic  /home/vagrant && \
              chmod 600 /home/vagrant/id_bionic && \
              chown vagrant:vagrant /home/vagrant/id_bionic"
    
    ansible.vm.provision "shell", inline: $script_ansible
    ansible.vm.provision "shell", inline: "ansible-playbook -i /vagrant/configs/ansible/hosts \
      /vagrant/configs/ansible/playbook.yml"
  end
end