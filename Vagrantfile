BOX_IMAGE = "debian/bullseye64"
NODE_COUNT = 3


Vagrant.configure("2") do |config|

  config.vm.define "master" do |subconfig|
    subconfig.vm.box = BOX_IMAGE
    subconfig.vm.hostname = "master.k8s.test"
    subconfig.vm.network :private_network, ip: "10.0.0.10"
    subconfig.vm.provider :libvirt do |libvirt|
      libvirt.host = "vm1-qemu"
      libvirt.driver = "kvm"
      libvirt.uri = 'qemu+unix:///system'
      libvirt.memory = 4096
      libvirt.cpus = 4
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
        libvirt.memory = 4096
        libvirt.cpus = 4
      end
    end
  end

  config.vm.synced_folder "shared", "/vagrant", disabled: false

  config.ssh.insert_key = false
end
