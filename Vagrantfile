# Default box image. 
BOX_IMAGE = "debian/bullseye64"

# Number of nodes besides master to deploy
# make sure your computer support this number of nodes (default: 3)
NODE_COUNT = 3

# Set the amount of memory in MB for all the machines 
# make sure your computer support allocating the necessary memory (default: 4096 or 4GB)
MEMORY = 2048

# Set the number of cpu cores fro all machines,
# make sure your computer support this number of cpu cores (default: 4)
CPUS = 2

Vagrant.configure("2") do |config|

  config.vm.define "master" do |subconfig|
    subconfig.vm.box = BOX_IMAGE
    subconfig.vm.hostname = "master.k8s.test"
    subconfig.vm.network :private_network, ip: "10.0.0.10"
    subconfig.vm.provider :libvirt do |libvirt|
      libvirt.host = "vm1-qemu"
      libvirt.driver = "kvm"
      libvirt.uri = 'qemu+unix:///system'
      libvirt.memory = MEMORY
      libvirt.cpus = CPUS
    end
  end

  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |subconfig|
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "node#{i}.k8s.test"
      subconfig.vm.network :private_network, ip: "10.0.0.#{i + 10}"
      subconfig.vm.provider :libvirt do |libvirt|
        libvirt.host = "vm#{1 + i}-qemu"
        libvirt.driver = "kvm"
        libvirt.uri = 'qemu+unix:///system'
        libvirt.memory = MEMORY
        libvirt.cpus = CPUS
      end
    end
  end

  config.vm.synced_folder "shared", "/vagrant", disabled: false

  config.ssh.insert_key = false
end

# vi: set ft=ruby :