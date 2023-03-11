
Vagrant.configure("2") do |config|
    #config.vm.synced_folder ".", "/vagrant", disabled: false
    config.vm.synced_folder ".", "/vagrant", type: "sshfs"
    config.vbguest.auto_update = false
    #config.vm.box_url = "http://cloud.centos.org/centos/8/x86_64/images/CentOS-8-Vagrant-8.1.1911-20200113.3.x86_64.vagrant-virtualbox.box"
    config.vm.box = "centos8-kernel6"
   
end
