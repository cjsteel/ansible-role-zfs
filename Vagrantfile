# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Developed based on Larry Smith Jr.'s complex Vagrantfile examples
#
#   * [vagrant-complex-vagrantfile-configurations](https://everythingshouldbevirtual.com/virtualization/vagrant-complex-vagrantfile-configurations/)
#
# Virtual disk note
#
#  Virtual disk type vdi
#    throws error "agsize (2560 blocks) too small, need at least 4096 blocks"
#    when formatting with:
#      sudo /sbin/mkfs.xfs -f -K -d su=128k,sw=8 -l version=2,su=128k -isize=512 /dev/sdf
#
#  Virtualdisk type vdmk 
#    Works fine with above mkfs.xfs command

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# ---- Define number of nodes to spin up ----
N = 1

# ---- Define any custom memory/cpu requirement ----
# if custom requirements are desired...ensure to set
# custom_cpu_mem == "yes" otherwise set to "no"
# By default if custom requirements are defined and set below
# any node not defined will be configure
# which is 1vCPU/512mb...So if setting custom requirements
# only define any node which requires more than the defaults.

nodes = [
  {
    :node => "node0",
    :box => "ubuntu/bionic64",
    :cpu => 2,
    :mem => 1024
  }
]

# ---- Define variables below ----
add_controller = "no"
addiitonal_controller_ports = 3
additional_disks = "yes"  #Define if additional drives defined should be added (yes | no)
additional_disks_controller = "SCSI"
additional_disks_num = 14  #Define the number of additional disks to add
additional_disks_path = "/mnt/csteel/tmp/"
additional_disks_size = 5  #Define disk size in GB
additional_nics = "yes"  #Define if additional network adapters should be created (yes | no)
additional_nics_dhcp = "no"  #Define if additional network adapters should be DHCP assigned
additional_nics_num = 2  #Define the number of additional nics to add
ansible_groups = {
  "test-nodes" => [
    "node[0:#{N-1}"
  ]
}
box = "ubuntu/bionic64"  #Define Vagrant box to load
custom_cpu_mem = "yes"  #Define if custom cpu and memory requirements are needed (yes| no)...defined within nodes variable above
desktop = "no"  #Define if running desktop OS (yes | no)
enable_custom_boxes = "no"  #Define if custom boxes should be used...defined in nodes var..
enable_port_forwards = "yes"  #Define if port forwards should be enabled
linked_clones = "no"  #Defines if nodes should be linked from master VM (yes | no)
port_forwards = [
  {
    :node => "node0",
    :guest => 8000,
    :guest_ip => "localhost", 
    :host => 8090
    #:host_ip => ""
  },
]
provision_nodes = "yes"  #Define if provisioners should run (yes | no)
server_cpus = 2  #Define number of CPU cores...will be ignored if custom_cpu_mem == "yes"
server_memory = 1024  #Define amount of memory to assign to node(s)...will be ignored if custom_cpu_mem == "yes"
subnet = "192.168.30."
subnet_ip_start = 10  #Define starting last octet of the subnet range to begin addresses for node(s)
#subnet = "192.168.202."  #Define subnet for private_network (If not using DHCP)
#subnet_ip_start = 200   #Define starting last octet of the subnet range to begin addresses for node(s)

Vagrant.configure(2) do |config|

  #Iterate over nodes
  (1..N).each do |node_id|
    nid = (node_id - 1)

    config.vm.define "node#{nid}" do |node|
      if enable_custom_boxes == "yes"
        box_set = "no"  #Initially set to no so it can be set to true if found in custom box defined
        nodes.each do |cust_box|
          if cust_box[:node] == "node#{nid}"
            node.vm.box = cust_box[:box]
            box_set = "yes"
          end
        end
        if box_set == "no"
          node.vm.box = box
        end
      end
      if enable_custom_boxes == "no"
        node.vm.box = box
      end
      node.vm.provider "virtualbox" do |vb|
        if linked_clones == "yes"
          vb.linked_clone = true
        end
        if custom_cpu_mem == "no"
          vb.customize ["modifyvm", :id, "--cpus", server_cpus]
          vb.customize ["modifyvm", :id, "--memory", server_memory]
        end
        if custom_cpu_mem == "yes"
          nodes.each do |cust_node|
            if cust_node[:node] == "node#{nid}"
              vb.customize ["modifyvm", :id, "--cpus", cust_node[:cpu]]
              vb.customize ["modifyvm", :id, "--memory", cust_node[:mem]]
            end
          end
        end
        if desktop == "yes"
          vb.gui = true
          vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxvga"]
          vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
          vb.customize ["modifyvm", :id, "--ioapic", "on"]
          vb.customize ["modifyvm", :id, "--vram", "128"]
          vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
        end
        if add_controller == "yes"
          dnum = additional_disks_num
          dpath = additional_disks_path
          ddev = ("#{dpath}node#{nid}_Disk#{dnum}.vdmk")
          unless File.exist?("#{ddev}")
            vb.customize ['storagectl', :id, '--name', "#{additional_disks_controller}", '--add', 'sata', '--portcount', dnum]
          end
        end
        if additional_disks == "yes"
          (1..additional_disks_num).each do |disk_num|
            dnum = (disk_num + 1)
            dpath = additional_disks_path
            ddev = ("#{dpath}node#{nid}_Disk#{dnum}.vdmk")
            unless File.exist?("#{ddev}")
              vb.customize ['createhd', '--filename', ("#{ddev}"), '--variant', 'Fixed', '--size', additional_disks_size * 1024]
            end
            vb.customize ['storageattach', :id,  '--storagectl', "#{additional_disks_controller}", '--port', dnum, '--device', 0, '--type', 'hdd', '--medium', "#{ddev}"]
          end
        end
      end
      node.vm.hostname = "node#{nid}"

      ### Define additional network adapters below
      if additional_nics == "yes"
        if additional_nics_dhcp == "no"
          (1..additional_nics_num).each do |nic_num|
            #nnum = Random.rand(0..50)
            nnum = 5
            node.vm.network :private_network, ip: subnet+"#{subnet_ip_start + nid + nnum}"
          end
        end
        if additional_nics_dhcp == "yes"
          (1..additional_nics_num).each do |nic_num|
            node.vm.network :private_network, type: "dhcp"
          end
        end
      end

      ### Define port forwards below
      if enable_port_forwards == "yes"
        port_forwards.each do |pf|
          if pf[:node] == "node#{nid}"
            node.vm.network "forwarded_port", guest: pf[:guest], host: pf[:host] + nid
          end
        end
      end
      if provision_nodes == "yes"
        if node_id == N
          #node.vm.provision "ansible" do |ansible|  #runs bootstrap Ansible playbook
          #  ansible.limit = "all"
          #  ansible.playbook = "bootstrap.yml"
          #end
          node.vm.provision "ansible" do |ansible|  #runs Ansible playbook for installing roles/executing tasks
            ansible.limit = "node0"
            ansible.playbook = "playbook.yml"
            ansible.groups = ansible_groups
          end
        end
      end

    end
  end
#  if provision_nodes == "yes"
#    config.vm.provision :shell, path: "bootstrap.sh", keep_color: "true"  #runs initial shell script
#  end
end

