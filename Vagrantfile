# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.vm.define "mdadm" do |mdadm|
    mdadm.vm.box = "generic/ubuntu2204"
    mdadm.vm.box_check_update = false
    mdadm.vm.hostname = "mdadm"
    #mdadm.vm.disk :disk, size: "20GB", primary: true
    mdadm.vm.provider "virtualbox" do |vb|
      vb.name = "mdadm"
      vb.memory = "2048"
      vb.cpus = "2"
      vb.default_nic_type = "virtio"
      file_to_disk = "mdadm.vmdk"
      unless File.exist?(file_to_disk)
          vb.customize [ "createmedium", "disk", "--filename", "vdisk1.vmdk", "--format", "vmdk", "--size", 1024 * 5 ]
          vb.customize [ "createmedium", "disk", "--filename", "vdisk2.vmdk", "--format", "vmdk", "--size", 1024 * 5 ]
          vb.customize [ "createmedium", "disk", "--filename", "vdisk3.vmdk", "--format", "vmdk", "--size", 1024 * 5 ]
          vb.customize [ "createmedium", "disk", "--filename", "vdisk4.vmdk", "--format", "vmdk", "--size", 1024 * 5 ]
          vb.customize [ "createmedium", "disk", "--filename", "vdisk5.vmdk", "--format", "vmdk", "--size", 1024 * 5 ]
      end
      vb.customize [ "storageattach", "mdadm" , "--storagectl", "SATA Controller", "--port", "2", "--device", "0", "--type", "hdd", "--medium", "vdisk1.vmdk" ]
      vb.customize [ "storageattach", "mdadm" , "--storagectl", "SATA Controller", "--port", "3", "--device", "0", "--type", "hdd", "--medium", "vdisk2.vmdk" ]       
      vb.customize [ "storageattach", "mdadm" , "--storagectl", "SATA Controller", "--port", "4", "--device", "0", "--type", "hdd", "--medium", "vdisk3.vmdk" ]
      vb.customize [ "storageattach", "mdadm" , "--storagectl", "SATA Controller", "--port", "5", "--device", "0", "--type", "hdd", "--medium", "vdisk4.vmdk" ]  
      vb.customize [ "storageattach", "mdadm" , "--storagectl", "SATA Controller", "--port", "6", "--device", "0", "--type", "hdd", "--medium", "vdisk5.vmdk" ]
    end
    #mdadm.vm.provision "shell", path: "disk.sh"
	mdadm.vm.provision "shell", inline: <<-SHELL
            mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
            mdadm --create --verbose --force /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
            cat /proc/mdstat
            mkdir /etc/mdadm/
            echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
            mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
            parted -s /dev/md0 mklabel gpt
            parted /dev/md0 mkpart primary ext4 0% 20%
            parted /dev/md0 mkpart primary ext4 20% 40%
            parted /dev/md0 mkpart primary ext4 40% 60%
            parted /dev/md0 mkpart primary ext4 60% 80%
            parted /dev/md0 mkpart primary ext4 80% 100%
            for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
            mkdir -p /raid/part{1,2,3,4,5}
            for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
            echo "#NEW DEVICE" >> /etc/fstab
            for i in $(seq 1 5); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /u0$i ext4 defaults 0 0 >> /etc/fstab; done
        SHELL
        
	end	
end
