Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  config.vm.box = "generic/rhel9"

  #SHELL PARA SUSCRIBIR, INSTALAR PAQUETES, CREAR USUARIO STUDENT Y PERMITIR LOGIN CON CLAVE
  config.vm.provision "shell", inline: <<-SHELL
    if ! subscription-manager status; then
      sudo subscription-manager register --username='sisnetporta' --password='j7r&4yNBw' --auto-attach
    fi
    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    sudo yum module install -y python3
    sudo useradd student
    echo student | passwd --stdin student
    echo "student ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/student
  SHELL
  
  #CREACION DE NODOS ADMINISTRADOS
  #NODO A
  config.vm.define "servera.lab.example.com" do |node|
    node.vm.hostname = "servera.lab.example.com"
    node.vm.network "private_network", ip: "192.168.50.211"
    node.vm.provider "virtualbox" do |v|
	  v.name = "servera.lab.example.com"
    end    
  end
  #NODO B
  config.vm.define "serverb.lab.example.com" do |node|
    node.vm.hostname = "serverb.lab.example.com"
    node.vm.network "private_network", ip: "192.168.50.212"
    node.vm.provider "virtualbox" do |v|
	  v.name = "serverb.lab.example.com"
    end
  end
  #NODO C
  config.vm.define "serverc.lab.example.com" do |node|
    node.vm.hostname = "serverc.lab.example.com"
    node.vm.network "private_network", ip: "192.168.50.213"
    node.vm.provider "virtualbox" do |v|
	  v.name = "serverc.lab.example.com"
    end    
  end
  #NODO D
  config.vm.define "serverd.lab.example.com" do |node|
    node.vm.hostname = "serverd.lab.example.com"
    node.vm.network "private_network", ip: "192.168.50.214"
    node.vm.provider "virtualbox" do |v|
	  v.name = "serverd.lab.example.com"
    end
  end
    
  #CREACION DE NODO DE CONTROL
  config.vm.define "workstation.lab.example.com" do |controller|
    controller.vm.synced_folder "lab", "/home/student/", type: "rsync"
    controller.vm.hostname = "workstation.lab.example.com"
    controller.vm.network "private_network", ip: "192.168.50.210"
    controller.vm.provider "virtualbox" do |v|
      v.name = "workstation.lab.example.com"
    end
	
	#LLENAR ARCHIVO DE HOSTS, COPIAR LABORATORIOS CON RSYNC, CREAR RELACION DE CONFIANZA E INSTALAR ANSIBLE-NAVIGATOR
	controller.vm.provision "shell", inline: <<-SHELL
	  sudo echo "192.168.50.210 workstation.lab.example.com" >> /etc/hosts
	  sudo echo "192.168.50.211 servera.lab.example.com" >> /etc/hosts
	  sudo echo "192.168.50.212 serverb.lab.example.com" >> /etc/hosts
	  sudo echo "192.168.50.213 serverc.lab.example.com" >> /etc/hosts
	  sudo echo "192.168.50.214 serverd.lab.example.com" >> /etc/hosts
	        
	  sudo echo "sudo su - student" >> /home/vagrant/.bash_profile
	  sudo -u student /bin/sh << 'STUDENT_USER'
		sudo chown -R student:student /home/student
        cd /home/student
        mkdir -v .ssh
        sudo yum install -y vim sshpass
        cd /home/student/.ssh/
        ssh-keygen -N "" -f id_rsa        
        sshpass -p 'student' ssh-copy-id -o StrictHostKeyChecking=no -i id_rsa.pub student@servera.lab.example.com
        sshpass -p 'student' ssh-copy-id -o StrictHostKeyChecking=no -i id_rsa.pub student@serverb.lab.example.com
		sshpass -p 'student' ssh-copy-id -o StrictHostKeyChecking=no -i id_rsa.pub student@serverc.lab.example.com
		sshpass -p 'student' ssh-copy-id -o StrictHostKeyChecking=no -i id_rsa.pub student@serverd.lab.example.com
        sudo yum install -y ansible podman python3-pip		
        python3 -m pip install ansible-navigator --user
		echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.profile
		source ~/.profile
      STUDENT_USER
	SHELL
  end
end
