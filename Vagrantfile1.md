```
# Create 3 VM // Get IP via DHCP // Uses VirtualBox
Vagrant.configure("2") do |config|

    config.vm.provider "virtualbox"
    # Create a public network via my laptop interface wlp6s0
    config.vm.network "public_network",
        bridge: "wlp6s0",
        use_dhcp_assigned_default_route: true

   # Create VM-1
   config.vm.define "vm1", primary: true, autostart: true do | vm1 |
      vm1.vm.box = "ubuntu/trusty64"
      vm1.vm.hostname = 'vm1'

      vm1.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", "vm1"]
      end
   end

   # Create VM-1
   config.vm.define "vm2", autostart: true do | vm2 |
      vm2.vm.box = "ubuntu/trusty64"
      vm2.vm.hostname = 'vm2'

      vm2.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", "vm2"]
      end
    end

   # Create VM-1
   config.vm.define "vm3", autostart: true do | vm3 |
      vm3.vm.box = "ubuntu/trusty64"
      vm3.vm.hostname = 'vm3'

      vm3.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", "vm3"]
      end
    end
end
```
