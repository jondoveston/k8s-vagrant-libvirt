# -*- mode: ruby -*-
# vi: set ft=ruby :

N = 2
IP = "192.168.73"

Vagrant.configure("2") do |config|
  (0..N).each do |node_id|
    config.vm.define "px#{node_id}" do |node|
      node.vm.hostname = "px#{node_id}"
      node.vm.box = "generic/debian11"
      # node.ssh.forward_agent = true

      node.vm.provider :libvirt do |domain|
        # domain.driver = "kvm"
        # domain.host = 'localhost'
        # domain.uri = 'qemu:///system'
        domain.title = "px#{node_id}"
        domain.description = "K8s node #{node_id}"
        domain.cpus = 2
        domain.memory = 4096
        # domain.graphics_type = 'none'
        # domain.cpu_mode = 'host-passthrough'
        # domain.qemu_use_session = false
      end

      node.vm.network :private_network, ip: "#{IP}.#{node_id+100}"
      node.vm.network :public_network, dev: "br0", mode: "bridge", type: "bridge"

      node.vm.synced_folder ".", "/vagrant", disabled: true

      node.vm.provision :hosts, sync_hosts: true

      node.vm.provision :shell do |shell|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/authorized_keys").first.strip
        shell.inline = <<-SHELL
          mkdir -p /home/vagrant/.ssh
          touch /home/vagrant/.ssh/authorized_keys
          grep -qxF "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys || echo "#{ssh_pub_key}" >> /home/vagrant/.ssh/authorized_keys
          mkdir -p /root/.ssh
          touch /root/.ssh/authorized_keys
          grep -qxF "#{ssh_pub_key}" /root/.ssh/authorized_keys || echo "#{ssh_pub_key}" >> /root/.ssh/authorized_keys
        SHELL
      end

      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      if node_id == N
        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "playbook.yml"
          ansible.tags = ["wip"] if ARGV[0] == "provision"
          # ansible.verbose = true
        end
      end
    end
  end
end
