Vagrant.configure("2") do |config|
  # Configuration de la machine virtuelle
  
  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  config.vm.define "debian1" do |host_config|
    host_config.vm.hostname = "debian1"
    host_config.vm.box = "generic/debian12"
    host_config.vm.network "private_network", ip: "192.168.60.10"
  end

  config.vm.define "rocky1" do |host_config|
    host_config.vm.hostname = "rocky1"
    host_config.vm.box = "generic/rocky9"
    host_config.vm.network "private_network", ip: "192.168.60.20"

    # Configuration de la provision avec Ansible
    host_config.vm.provision "ansible" do |ansible|
      ansible.limit = "all"
      ansible.playbook = "playbooks/install.yml"        # Chemin vers le playbook Ansible
      ansible.inventory_path = "inventories/vagrant/inventory"    # Chemin vers l'inventaire Ansible spécifique
      ansible.compatibility_mode = "2.0"
    end
  end
end
