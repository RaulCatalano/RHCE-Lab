# -*- mode: ruby -*-
# vi: set ft=ruby :

boxes = [
  { :name => 'controller', :ip => '192.168.50.4' },
  { :name => 'managed1', :ip => '192.168.50.3' },
  { :name => 'managed2', :ip => '192.168.50.2' }
]

Vagrant.configure("2") do |config|
  config.vm.box = 'generic/rhel9'

  boxes.each do |box|

    if Vagrant.has_plugin?('vagrant-registration')
      config.registration.username = ENV['SUB_USERNAME']
      config.registration.password = ENV['SUB_PASSWORD']
      config.registration.unregister_on_halt = false
    end

    config.vm.define box[:name] do |boxconfig|
      boxconfig.ssh.insert_key = false
      boxconfig.vm.hostname = box[:name]
      boxconfig.vm.network :private_network, ip: box[:ip]
      boxconfig.vm.provider :virtualbox do |vb|
        vb.memory = 1024
        vb.cpus = 1

        # Add external disk to managed nodes
        if box[:name] != 'controller'

          file_to_disk = File.realpath( "." ).to_s + "/disk.#{box[:name]}"

          if ARGV[0] == 'up' && ! File.exist?(file_to_disk) 
            vb.customize [
                'createhd', 
                '--filename', file_to_disk, 
                '--format', 'VDI', 
                '--size', 10 * 1024 #10GB
                ] 
            vb.customize [
                'storageattach', :id,
                '--storagectl', 'SATA Controller', 
                '--port', 1, '--device', 0, 
                '--type', 'hdd', '--medium', 
                file_to_disk
                ]
          end
        end
        
        # Bootstrap controller node
        if box[:name] == 'controller'

          # Setup SSH key
          boxconfig.vm.provision 'file', source: '~/.vagrant.d/insecure_private_key', destination: '~/.ssh/id_rsa'
          boxconfig.vm.provision :shell, inline: 'chmod 600 ~/.ssh/id_rsa', privileged: false

          # Setup Ansible
          boxconfig.vm.provision :shell, inline: 'dnf -y install ansible', privileged: true
          boxconfig.vm.provision 'file', source: './ansible.cfg', destination: '~/ansible.cfg'
          boxconfig.vm.provision 'file', source: './hosts', destination: '~/hosts'
        end

      end
    end
  end
end
