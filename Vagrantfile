# -*- mode:  ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {

:ns01 => {
        :box_name => "centos/8",
        :net => [
                {ip: '192.168.50.10', netmask: '255.255.255.0', adapter: 2, virtualbox__intnet: 'dns'},
              ]
},
:ns02 => {
        :box_name => "centos/8",
        :net => [
                {ip: "192.168.50.11", netmask: "255.255.255.0", adapter: 2, virtualbox__intnet: "dns"},
              ]
},
:client => {
        :box_name => "centos/8",
        :net => [
            {ip: "192.168.50.15", netmask: "255.255.255.0", adapter: 2, virtualbox__intnet: "dns"},
              ]
},
:client2 => {
        :box_name => "centos/8",
        :net => [
            {ip: "192.168.50.16", netmask: "255.255.255.0", adapter: 2, virtualbox__intnet: "dns"},
              ]
},

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder "./", "/vagrant", type: "rsync", rsync__auto: true, rsync__exclude: ['./hddvm, README.md']
    config.vm.define boxname do |box|
    config.vm.provider "virtualbox" do |vrt|
        vrt.memory = 512
    end
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
          sed -i.bak 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
          systemctl restart sshd

        SHELL

        box.vm.provision :ansible_local do |ansible|
          ansible.galaxy_role_file = 'requirements.yml'
          ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file}"
          ansible.verbose = "vv"
          ansible.install = "true"
          #ansible.limit = "all"
          ansible.tags = boxname.to_s
          # ansible.tags = "facts"
          ansible.inventory_path = "./ansible/inventory/"
          ansible.playbook = "./ansible/playbooks/dns.yml"
        end
    end
  end
end
