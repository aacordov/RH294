NUMBER_OF_NODES = ENV['NUMBER_OF_NODES'] = '2'
NUMBER_OF_NODES_TO_INT = NUMBER_OF_NODES.to_i
Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", type: "rsync"
  config.vm.box_check_update = false

  config.vm.provision "shell", inline: <<-SHELL
  if ! subscription-manager status; then
	sudo subscription-manager register --username='sisnetporta' --password='j7r&4yNBw' --auto-attach
  fi
  sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  sudo yum module install -y python36
  sudo useradd student
  echo student | passwd --stdin student
  echo "student ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/student
  SHELL

  (1..NUMBER_OF_NODES_TO_INT).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.box = "generic/rhel9"
      node.vm.hostname = "node0#{i}"
      node.vm.network "private_network", ip: "192.168.50.#{i + 210}"
      file_for_disk = "./large_disk#{i}.vdi" # Create & attach a 5GiB disk to each node machine
      node.vm.provider "virtualbox" do |v|
		v.name = "node0#{i}"
        # If the disk already exists don't create it
        unless File.exist?(file_for_disk)
            v.customize ['createhd', '--filename', file_for_disk, '--size', 5120]
        end
        v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_for_disk]
      end
    end
  end

  config.vm.define "controller" do |controller|
    controller.vm.box = "generic/rhel9"
    controller.vm.hostname = "controller"
    controller.vm.network "private_network", ip: "192.168.50.210"
	controller.vm.provider "virtualbox" do |v|
		v.name = "node0#{i}"
	end
    controller.vm.provision "shell", inline: <<-SHELL
    sudo echo "192.168.50.210 controller" >> /etc/hosts
	export NUMBER_OF_NODES=#{ENV['NUMBER_OF_NODES']}        
    	for((i=1; i<=$NUMBER_OF_NODES; i++));
    do
      sudo echo "192.168.50.21$i node0$i" >> /etc/hosts
    done
	    
	sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    # Use ansible user instead of vagrant
    sudo echo "sudo su - student" >> /home/vagrant/.bash_profile
    # [1] [5]
    sudo -u student /bin/sh << 'STUDENT_USER'
      export NUMBER_OF_NODES=#{ENV['NUMBER_OF_NODES']}        
      cd /home/student
      mkdir -v .ssh
      sudo yum install -y vim sshpass
      cd /home/student/.ssh/
      ssh-keygen -N "" -f id_rsa # Generate public and private key pairs (id_rsa, id_rsa.pub)
      # Add public key to all managed servers [2]
      for((i=1; i<=$NUMBER_OF_NODES; i++));
      do
        sshpass -p 'student' ssh-copy-id -o StrictHostKeyChecking=no -i id_rsa.pub student@node0$i
      done
	  sudo yum install -y ansible podman python3-pip
	  python3 -m pip install ansible-navigator --user
STUDENT_USER
  SHELL
  end
end