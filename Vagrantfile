# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 30022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "irouter-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 31022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "dir-net"},
                   {adapter: 3, virtualbox__intnet: "irouter-net"},
                   {adapter: 4, virtualbox__intnet: "irouter2-net"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 32022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "dir-net"},
                ]
  },
  :inetRouter2 => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 33022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "irouter2-net"},
                   {adapter: 3, ip: '192.168.18.13', netmask: "255.255.255.0"},
                ]
  },
  :provision => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 34022},
        :net => [
                   {adapter: 2, ip: '192.168.18.2', netmask: "255.255.255.252"}
                ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.network "forwarded_port", boxconfig[:forward]

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
          sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
          systemctl restart sshd
        SHELL
        case boxname.to_s
        when "provision"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install epel-release -y
            yum install ansible -y
            cp -r /vagrant/* /root/
            cd /root
            ansible-playbook filtering.yml
            SHELL
        end        
#        case boxname.to_s
#        when "inetRouter"
#          box.vm.provision "shell", run: "always", inline: <<-SHELL
#            sysctl net.ipv4.conf.all.forwarding=1
#            iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
#            ip route add 192.168.0.0/16 via 192.168.255.2
#            SHELL
#        when "inetRouter2"
#          box.vm.provision "shell", run: "always", inline: <<-SHELL
#            sysctl net.ipv4.conf.all.forwarding=1
#            iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
#            ip route add 192.168.0.0/16 via 192.168.255.14
#            SHELL
#        when "centralRouter"
#          box.vm.provision "shell", run: "always", inline: <<-SHELL
#            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#            systemctl restart network
#            sysctl net.ipv4.conf.all.forwarding=1
#            ip route add default via 192.168.255.1
#            SHELL
#        when "centralServer"
#          box.vm.provision "shell", run: "always", inline: <<-SHELL
#            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#            systemctl restart network
#            ip route add default via 192.168.0.1
#            SHELL
#
#        end

      end

  end
  
  
end
