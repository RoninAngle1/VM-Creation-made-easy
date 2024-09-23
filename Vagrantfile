require 'highline/question'

Vagrant.configure("2") do |config|

  # User prompts for configuration
  num_machines = HighLine.new.ask("Enter the number of machines (default: 1): ") || 1
  num_machines = num_machines.to_i

  # Change the default machine path (if needed)
  Vagrant::Environment.instance_variable_set(:@home, "/run/media/alireza/data/machines/")

  (1..num_machines).each do |i|
    hostname = HighLine.new.ask("Enter hostname for machine #{i} (default: machine-#{i}): ") || "machine-#{i}"

    # Ensure user enters a base box name (fix for the error)
    base_box = HighLine.new.ask("Enter base box for machine #{i} (default: debian/bullseye64): ") || "debian/bullseye64"

    memory_size = HighLine.new.ask("Enter memory size for machine #{i} (default: 2048 MB): ") || "2048"
    cpu_cores = HighLine.new.ask("Enter number of CPU cores for machine #{i} (default: 2): ") || "2"

    config.vm.define hostname do |machine|
      machine.vm.box = base_box  # Set the provided base box name
      machine.vm.hostname = hostname

      machine.vm.provider "virtualbox" do |vb|
        vb.memory = memory_size
        vb.cpus = cpu_cores
      end

      # Provisioning script with interactive elements
      machine.vm.provision "shell", inline: <<-SHELL
        #!/bin/bash
        # Prompt for additional configuration
        echo "Retrieving NIC IP address for machine #{i}..."
        ip_address=$(ip addr show eth1 | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)
        echo "The server NIC IP address for machine #{i} (#{hostname}) is: $ip_address"
      SHELL

      # Setting up a private network (this will assign an IP to eth1)
      machine.vm.network "private_network", type: "dhcp"
    end
  end
