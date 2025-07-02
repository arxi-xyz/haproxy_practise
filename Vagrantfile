Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.define "master" do |master|
    master.vm.hostname = "debian-vm3"
    master.vm.network "private_network", ip: "192.168.56.13", adapter: 2, type: "static", virtualbox__intnet: false
    master.vm.synced_folder "./haproxy/haproxy_config", "/etc/haproxy", type: "virtualbox", mount_options: ["dmode=775", "fmode=664"], create: true
    master.vm.synced_folder "./haproxy/haproxy_source", "/usr/local/src/haproxy", type: "virtualbox", mount_options: ["dmode=775", "fmode=664"], create: true
    master.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  nodes = [
    { name: "node_1", hostname: "debian-node-1", ip: "192.168.56.10" },
    { name: "node_2", hostname: "debian-node-2", ip: "192.168.56.11" },
    { name: "node_3", hostname: "debian-node-3", ip: "192.168.56.12" }
  ]

  nodes.each do |node|
    config.vm.define node[:name] do |node_config|
      node_config.vm.hostname = node[:hostname]
      node_config.vm.network "private_network", ip: node[:ip], adapter: 2, type: "static", virtualbox__intnet: false
    end
  end

  # Ansible provisioning
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "./ansible/user_creator/playbook.yml"
    ansible.vault_password_file = "./ansible/user_creator/vault_pass.txt"
    ansible.groups = {
      "all_vms" => ["master"] + nodes.map { |n| n[:name] }
    }
    ansible.limit = "all" # Ensure provisioning runs on all VMs
  end
end