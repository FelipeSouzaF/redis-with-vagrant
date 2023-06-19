# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
    # The most common configuration options are documented and commented below.
    # For a complete reference, please see the online documentation at
    # https://docs.vagrantup.com.

    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://vagrantcloud.com/search.
    config.vm.box = "bento/ubuntu-22.04"

    config.vm.provider :virtualbox do |v|
      v.memory = 2048
      v.cpus = 1
    end

    # Define two VMs with static private IP addresses.
    boxes = [
        { :name => "redisn1", :ip => "192.168.56.111", :hostname => "redisn1" },
    ]

    # Provision each of the VMs.
    boxes.each do |opts|
        config.vm.define opts[:name] do |config|
            config.vm.hostname = opts[:hostname]
            config.vm.network :private_network, ip: opts[:ip]

            # config.vm.synced_folder "files/", "/opt/vagrant-files", create: true

            config.vm.provision "shell" do |s|
                # Read host ssh pub key
                ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_ed25519.pub").first.strip

                # Create alias for the ip addresses of the other boxes.
                hosts_file = ""
                boxes.each do |address|
                    # Skip the current box.
                    if address[:name] == opts[:name]
                        next
                    end
                    hosts_file += "#{address[:ip]} #{address[:hostname]}\n"
                end

                # Add the host ssh pub key into authorized_keys of the boxes.
                # Apply the other boxes alias to hosts file.
                s.inline = <<-SHELL
                echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys &&
                echo -e "#{hosts_file}" >> /etc/hosts

                curl -fsSL https://packages.redis.io/gpg | gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

                echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/redis.list

                apt update -y

                apt install redis zip -y
                SHELL
            end
        end
    end
end
