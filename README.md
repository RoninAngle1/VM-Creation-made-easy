# VM-Creation-made-easy

A simple Vagrant file that help you create multiple VMs easily 

# Pre requirements

1- Vagrant installed on host

2- Virtualbox installed on host

3- this list of plugins be installed

```
1- hoe-highline
2- erb
```
## Install Plugins

NOTE: If you run into some errors like 

```
# This plugin is example
vagrant plugin install libvirt
Installing the 'libvirt' plugin. This can take a few minutes...
Vagrant failed to properly resolve required dependencies. These
errors can commonly be caused by misconfigured plugin installations
or transient network issues. The reported error is:

conflicting dependencies vagrant (= 1.5.0) and vagrant (= 2.4.1)
  Activated vagrant-2.4.1
  which does not match conflicting dependency (= 1.5.0)

  Conflicting dependency chains:
    vagrant (= 2.4.1), 2.4.1 activated

  versus:
    vagrant (= 1.5.0)

  Gems matching vagrant (= 1.5.0):
    vagrant-1.5.0

```

just do as below

```
VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install hoe-highline
VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install erb
```

## Change some ruby configurations in the user

Acutally when you try to run an interactive shell while running a Vagrantfile you will see this error

```
agrant up
/home/alireza/.vagrant.d/gems/3.2.5/gems/highline-1.7.10/lib/highline.rb:624:in `encode': no implicit conversion of Hash into String (TypeError)

    out = (indentation+statement).encode(Encoding.default_external, { :undef => :replace  } )
                                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
	from /home/alireza/.vagrant.d/gems/3.2.5/gems/highline-1.7.10/lib/highline.rb:624:in `say'
	from /home/alireza/.vagrant.d/gems/3.2.5/gems/highline-1.7.10/lib/highline.rb:261:in `ask'
	from /run/media/alireza/data/k8s-vagrant/Vagrantfile:183:in `block in <top (required)>'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/v2/loader.rb:40:in `load'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/loader.rb:129:in `block (2 levels) in load'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/loader.rb:122:in `each'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/loader.rb:122:in `block in load'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/loader.rb:119:in `each'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/config/loader.rb:119:in `load'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/vagrantfile.rb:34:in `initialize'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/environment.rb:835:in `new'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/environment.rb:835:in `vagrantfile'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/environment.rb:1016:in `process_configured_plugins'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/lib/vagrant/environment.rb:200:in `initialize'
	from /opt/vagrant/embedded/gems/gems/vagrant-2.4.1/bin/vagrant:211:in `new'

```

The sulotion is to change ```out = (indentation+statement).encode(Encoding.default_external, { :undef => :replace  } ``` line number is 624

change it to be like:
```
vim /home/alireza/.vagrant.d/gems/3.2.5/gems/highline-1.7.10/lib/highline.rb
########
out = (indentation+statement).encode(Encoding.default_external, { :undef => :replace  }
#######
```
then do run
```
vagrant up
```

# Vagrant File Explenation

This Vagrantfile defines a configuration for creating and managing virtual machines (VMs) with an interactive user experience. Here's a breakdown of the code:

1. Requiring Highline Gem:

```Ruby

require 'highline/question'
```

This line imports the highline/question library, which provides functionality for creating interactive prompts and collecting user input.

2. Vagrant Configuration Block:

```Ruby

Vagrant.configure("2") do |config|
  # ... configuration goes here ...
end
```

This block defines the Vagrant configuration. The "2" specifies that this configuration is intended for Vagrant version 2.

3. User Prompts for Configuration:

```Ruby

num_machines = HighLine.new.ask("Enter the number of machines (default: 1): ") || 1
num_machines = num_machines.to_i

# ... other prompts for hostname, base box, etc.
```

These lines use the HighLine class to ask the user for several configuration parameters:

```Ruby
    num_machines: The number of VMs to create (defaults to 1).
    hostname: The hostname for each VM (defaults to machine_#).
    base_box: The base box image to use for the VMs (defaults to debian/bullseye64).
    memory_size: The amount of memory allocated to each VM (defaults to 2048 MB).
    cpu_cores: The number of CPU cores for each VM (defaults to 2).
```

The || 1 part ensures that even if the user enters an empty value, the variable will still be set to 1 (for number of machines). Similarly, other prompts have default values.

4. Machine Definition Loop:

```Ruby

(1..num_machines).each do |i|
  # ... machine configuration for each VM ...
end
```

This loop iterates a number of times based on the user-provided num_machines value. Inside the loop, a new VM is defined for each iteration with the following configuration:

5. Defining Each Virtual Machine:

```Ruby

config.vm.define hostname do |machine|
  machine.vm.box = base_box  # Set the base box from user input
  machine.vm.hostname = hostname  # Set the hostname from user input

  # ... other machine configuration ...
end
```

config.vm.define hostname do |machine|: This line defines a new VM with the user-provided hostname.
  
machine.vm.box = base_box: This line sets the base box image for the VM based on the user's input or the default.
  
machine.vm.hostname = hostname: This line sets the hostname for the VM based on the user's input.

6. VirtualBox Provider Configuration:

```Ruby

machine.vm.provider "virtualbox" do |vb|
  vb.memory = memory_size
  vb.cpus = cpu_cores
end
```

This section configures the VirtualBox provider for the VM. It sets the following parameters based on user input:

vb.memory: Specifies the memory allocated to the VM.

vb.cpus: Defines the number of CPU cores allocated to the VM.

7. Interactive Shell Provisioning:

```Ruby

machine.vm.provision "shell", inline: <<-SHELL
  #!/bin/bash
  # Prompt for additional configuration
  echo "Retrieving NIC IP address for machine #{i}..."
  ip_address=$(ip addr show eth1 | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)
  echo "The server NIC IP address for machine #{i} (#{hostname}) is: $ip_address"
SHELL
```

This section defines a shell provisioning script that runs inside the VM after it's created. This script:

Prints a message indicating it's retrieving the IP address.

Uses shell commands to get the IP address of the eth1 network interface.

Prints the retrieved IP address for the VM along with its hostname.

8. Setting Up Private Network:

```Ruby
machine.vm.network "private_network", type: "dhcp"
```

This line configures a private network for the VM. The type: "dhcp" specifies that the VM will obtain an IP address automatically through DHCP.

Summary:

This Vagrantfile provides an interactive way to create multiple virtual machines with user-defined settings. It includes functionalities like setting base box image, memory, CPU cores, and retrieving the IP address after creation.

# Input sample and results

```
vagrant up
Enter the number of machines (default: 1): 1
Enter hostname for machine 1 (default: machine-1): machine-1
Enter base box for machine 1 (default: debian/bullseye64): debian/bullseye64
Enter memory size for machine 1 (default: 2048 MB): 2048
Enter number of CPU cores for machine 1 (default: 2): 2
Bringing machine 'machine-1' up with 'virtualbox' provider...
==> machine-1: Importing base box 'debian/bullseye64'...
==> machine-1: Matching MAC address for NAT networking...
==> machine-1: Checking if box 'debian/bullseye64' version '11.20240905.1' is up to date...
==> machine-1: Setting the name of the VM: k8s-vagrant_machine-1_1727095303687_22452
==> machine-1: Clearing any previously set network interfaces...
==> machine-1: Preparing network interfaces based on configuration...
    machine-1: Adapter 1: nat
    machine-1: Adapter 2: hostonly
==> machine-1: Forwarding ports...
    machine-1: 22 (guest) => 2222 (host) (adapter 1)
==> machine-1: Running 'pre-boot' VM customizations...
==> machine-1: Booting VM...
==> machine-1: Waiting for machine to boot. This may take a few minutes...
    machine-1: SSH address: 127.0.0.1:2222
    machine-1: SSH username: vagrant
    machine-1: SSH auth method: private key
    machine-1: 
    machine-1: Vagrant insecure key detected. Vagrant will automatically replace
    machine-1: this with a newly generated keypair for better security.
    machine-1: 
    machine-1: Inserting generated public key within guest...
    machine-1: Removing insecure key from the guest if it's present...
    machine-1: Key inserted! Disconnecting and reconnecting using new SSH key...
==> machine-1: Machine booted and ready!
==> machine-1: Checking for guest additions in VM...
    machine-1: The guest additions on this VM do not match the installed version of
    machine-1: VirtualBox! In most cases this is fine, but in rare cases it can
    machine-1: prevent things such as shared folders from working properly. If you see
    machine-1: shared folder errors, please make sure the guest additions within the
    machine-1: virtual machine match the version of VirtualBox you have installed on
    machine-1: your host and reload your VM.
    machine-1: 
    machine-1: Guest Additions Version: 6.0.0 r127566
    machine-1: VirtualBox Version: 7.0
==> machine-1: Setting hostname...
==> machine-1: Configuring and enabling network interfaces...
==> machine-1: Mounting shared folders...
    machine-1: /vagrant => /run/media/alireza/data/k8s-vagrant
==> machine-1: Running provisioner: shell...
    machine-1: Running: inline script
    machine-1: Retrieving NIC IP address for machine 1...
    machine-1: The server NIC IP address for machine 1 (machine-1) is: 192.168.56.17

==> machine-1: Machine 'machine-1' has a post `vagrant up` message. This is a message
==> machine-1: from the creator of the Vagrantfile, and not from Vagrant itself:
==> machine-1: 
==> machine-1: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports
```
# Post VM Creation

```
eval "$(ssh-agent -s)"

ssh-add .vagrant/machines/machine-1/virtualbox/private_key 
Identity added: .vagrant/machines/machine-1/virtualbox/private_key (vagrant)
```

Then you can ssh the new VM easily

```
ssh vagrant@192.168.56.17

The authenticity of host '192.168.56.17 (192.168.56.17)' can't be established.
ED25519 key fingerprint is SHA256:yemZXwd0rjiddHpcGef+r1f+y8ugcp2lVTkr2e8Rk/c.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.17' (ED25519) to the list of known hosts.
Linux machine-1 5.10.0-32-amd64 #1 SMP Debian 5.10.223-1 (2024-08-10) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
vagrant@machine-1:~$ 
vagrant@machine-1:~$ 
```
