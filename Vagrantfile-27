Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_check_update = false

  # Jenkins VM
  config.vm.define "jenkins" do |jenkins|
    jenkins.vm.hostname = "jenkins.example.com"
    jenkins.vm.network "private_network", ip: "192.168.56.210"
    jenkins.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # 2GB RAM
      vb.cpus = 1 # 1 CPU
    end

    # Install Git on Jenkins machine
    jenkins.vm.provision "shell", inline: <<-SHELL
      sudo yum update -y
      sudo yum install git -y
    SHELL

    # Enable password authentication for SSH and restart sshd
    jenkins.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      sudo systemctl restart sshd
    SHELL
  end

  # Ansible VM
  config.vm.define "ansible" do |ansible|
    ansible.vm.hostname = "ansible.example.com"
    ansible.vm.network "private_network", ip: "192.168.56.211"
    ansible.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # 2GB RAM
      vb.cpus = 1 # 1 CPU
    end

    # Install Ansible on Ansible machine
    ansible.vm.provision "shell", inline: <<-SHELL
      sudo yum update -y
      sudo yum install epel-release -y
      sudo yum install ansible -y
    SHELL

    # Add ansible user with password "redhat123" and grant sudo admin rights
    ansible.vm.provision "shell", inline: <<-SHELL
      sudo useradd ansible --password $(openssl passwd -1 redhat123)
      sudo usermod -aG wheel ansible
    SHELL

    # Enable password authentication for SSH and restart sshd
    ansible.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      sudo systemctl restart sshd
    SHELL
  end

  # Webserver VM
  config.vm.define "webserver" do |webserver|
    webserver.vm.hostname = "webserver.example.com"
    webserver.vm.network "private_network", ip: "192.168.56.212"
    webserver.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # 2GB RAM
      vb.cpus = 1 # 1 CPU
    end

    # Install Apache HTTP Server on Webserver machine
    webserver.vm.provision "shell", inline: <<-SHELL
      sudo yum update -y
      sudo yum install httpd -y
      sudo systemctl start httpd
      sudo systemctl enable httpd
    SHELL

    # Enable password authentication for SSH and restart sshd
    webserver.vm.provision "shell", inline: <<-SHELL
   #   sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
   #   sudo systemctl restart sshd
    SHELL

    # Add ansible user with password "redhat123" and grant sudo admin rights (same as Ansible machine)
    webserver.vm.provision "shell", inline: <<-SHELL
      sudo useradd ansible --password $(openssl passwd -1 redhat123)
      sudo usermod -aG wheel ansible
    SHELL
  end

  # Set up /etc/hosts entries on all servers
  config.vm.provision "shell", inline: <<-SHELL
    echo "192.168.56.210 jenkins.example.com" | sudo tee -a /etc/hosts
    echo "192.168.56.211 ansible.example.com" | sudo tee -a /etc/hosts
    echo "192.168.56.212 webserver.example.com" | sudo tee -a /etc/hosts
    echo "search example.com" | sudo tee -a /etc/resolv.conf
    sudo useradd -m ansible
    echo "redhat123" | sudo passwd --stdin ansible
    echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/ansible
    sudo sed -i 's/PasswordAuthentication no/#PasswordAuthentication no/' /etc/ssh/sshd_config
    sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    sudo yum install git -y
  SHELL
end

