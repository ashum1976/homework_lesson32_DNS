# -*- mode:  ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:srv01 => {
        # Сервер  для старта нескольких openvpn серверов, для проверки режимов работы L2 и L3 уровне, интерфейсы соответственно tap и tun. Первый адаптер - для настройки доступа через vpn  с хостовой машины. Adapter 5 создаётся без IP для организации моста, тип сети внешний
        :box_name => "centos/8",
        :net => [
                   {ip: '192.168.128.2',  netmask: "255.255.255.248", adapter: 2, virtualbox__extnet: "vpn-net_1"},
                   {ip: '192.168.160.2',  netmask: '255.255.255.248', adapter:3, virtualbox__intnet: "vpn-net_2"},
                   {ip: '192.168.192.2',  netmask: "255.255.255.248", adapter: 4, virtualbox__extnet: "vpn-net_3"},
                   { ip: '192.168.224.4', netmask: "255.255.255.240", adapter: 5, virtualbox__extnet: "router-net_5", mac: "08002795bf0e"},
                   # { ip: '0.0.0.0', netmask: "255.255.255.240", adapter: 5, virtualbox__extnet: "router-net_5", mac: "08002795bf0e"}
                ]
  },
  :cln01 => {
        # Клиент для запуска тунеля на L3 уровне, интерфейс tun
        :box_name => "centos/8",
        :net => [
                   {ip: '192.168.160.3',  netmask: "255.255.255.248", adapter: 2, virtualbox__intnet: "vpn-net_2"},
                ]
  },
  :cln02 => {
        # Клиент для запуска тунеля на L2 уровне, интерфейс tap. Adapter 3 создаётся без IP для организации моста, тип сети  внутренний
        :box_name => "centos/8",
        :net => [
                   {ip: '192.168.192.3',  netmask: "255.255.255.248", adapter: 2, virtualbox__extnet: "vpn-net_3"},
                   # { ip: '0.0.0.0', netmask: "255.255.255.240", adapter: 3, virtualbox__intnet: "router-net_4", mac: "080027ca8609"},
                   { ip: '192.168.224.6', netmask: "255.255.255.240", adapter: 3, virtualbox__intnet: "router-net_4", mac: "080027ca8609"},
                   # {adapter: 3, virtualbox__intnet: "router-net_4"}
                ]
  },
  :cln03 => {
        # Клиент для теста в сети, доступной через тунель на L2 уровне, интерфейс tap,  тип сети "внутренний - intnet". Подключен в сеть c cln02
        :box_name => "centos/8",
        :net => [
                   {ip: '192.168.224.7',  netmask: "255.255.255.240", adapter: 2, virtualbox__intnet: "router-net_4"},
                ]
  },

  :cln04 => {
        # Клиент для теста в сети, доступной через тунель на L2 уровне, интерфейс tap, тип сети "внешний - extnet". Подключен в сеть с srv01
        :box_name => "centos/8",
        :net => [
                   {ip: '192.168.224.5',  netmask: "255.255.255.240", adapter: 2, virtualbox__extnet: "router-net_5"},
                ]
  },


}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder "./", "/vagrant", type: "rsync", rsync__auto: true, rsync__exclude: ['./hddvm, README.md']
    config.vm.define boxname do |box|

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
       #Установка  коллекции community.general, для использования community.general.nmcli (nmcli) управление сетевыми устройствами.
          # ansible.galaxy_command = 'ansible-galaxy collection install community.general'
          ansible.galaxy_role_file = 'requirements.yml'
          ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file}"
          ansible.verbose = "vv"
          ansible.install = "true"
          #ansible.limit = "all"
          ansible.tags = boxname.to_s
          # ansible.tags = "facts"
          ansible.inventory_path = "./ansible/inventory/"
          ansible.playbook = "./ansible/playbooks/openvpn.yml"
          end

      end

  end


end
