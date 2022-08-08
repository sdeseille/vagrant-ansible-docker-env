# Preparing Docker environment with Vagrant and ansible_local provisioner

I use Vagrant to build and experiment with my work. It allows me to try and fail often and quickly to find the best architecture for my projects. I learned how to build a minimal kubernetes cluster with it. I defined the best path to migrate an openldap infrastructure for 200K users. 

>Vagrant has been in my toolbox since 2014 or so. It has been a long time and I am still happy to use it. 

Recently, I have been working more with Ansible for the need of a project. Previously, I was one of those who wrote shell scripts to automate the installation steps. But hey, I can do better by adding Ansible in my tool belt.  

The main purpose of this article is to share how i prepared a docker environment with Vagrant and ansible_local provisioner. When this environment is ready, i can work on an opensource project i want to contribute to. 

## Prerequisites

Did I mention that we'll use Vagrant ? Certainly. So you will need a hypervisor. I use a Microsoft operating system, so I need a hypervisor. Currently I am using Oracle's VirtualBox. 
The guest system I have installed is Ubuntu Focal64. 
As i said, I went through several iterations in order to get a solution that works. I ran into errors at various times when i tried to build the environment. What I found is that we need the followings tools with the specified version to have our environment. 

- Microsoft Windows: 10 Version 21H2
- Oracle VirtualBox: 6.1.36 r152435 [url](https://download.virtualbox.org/virtualbox/6.1.36/VirtualBox-6.1.36-152435-Win.exe?source=:ow:o:p:nav:mmddyyVirtualBoxHero)
- Vagrant: 2.2.19 [url](https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.msi)
- Vagrant Box [ubuntu/focal64]: 20220804 [url](https://app.vagrantup.com/ubuntu/boxes/focal64/versions/20220804.0.0)

## Vagrantfile initialisation

Our Docker environment will be based on Ubuntu Focal64. We use following command to init Vagrant.

>vagrant init ubuntu/focal64

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant init ubuntu/focal64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Content of the Vagrantfile

```powershell
PS D:\vagrant_projects\ubuntu_env> type .\Vagrantfile
```

```ruby
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
  config.vm.box = "ubuntu/focal64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

## Provisioner ansible_local

I decided to use Ansible to set the environment. Vagrant offers two provisioners. The classical [**ansible**] which required the host system to be linux based or the other choice when it is not the case [**ansible_local**]. When you use this provisioner, Vagrant will automaticaly install Ansible on the guest system and run the playbooks directly on it. 
See the Official [documentation](https://www.vagrantup.com/docs/provisioning/ansible_local). 

The minimal configuration for ansible_local is to defined a [**playbook.yml**] file that is read from the current project directory.

```ruby
  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
  end
```

After some iterations, I found that it is necessary to add some specifics options. They are needed to make the installation of pip with python 2.7. You should got following configuration.

```ruby
  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.install_mode = "pip"
    ansible.pip_install_cmd = "curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | sudo python"
  end
```

## Playbook for Docker

I followed the current [article](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04) from [Digital Ocean](https://www.digitalocean.com/) in order to build the playbook. The article is well explained, thanks to authors [Tony Tran](https://www.digitalocean.com/community/users/tonytran) and [Erika Heidi](https://www.digitalocean.com/community/users/erikaheidi).

The whole playbook worked well until the task [**Pull default Docker image**]. I had an error about the community.docker collection being messing. I made some additional research and studied the [article](https://steampunk.si/blog/getting-started-with-ansible/) from [XLAB SteamPunk](https://steampunk.si/) as well as the detailed documentation for [Vagrant](https://www.vagrantup.com/docs/provisioning/ansible_common) and [Ansible](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html#installing-collections-with-ansible-galaxy). With all this information, I finally figured out how to define my Vagrantfile.

```ruby
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.install_mode = "pip"
    ansible.pip_install_cmd = "curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | sudo python"
    ansible.galaxy_role_file = "requirements.yml"
    ansible.galaxy_command = "ansible-galaxy collection install -r %{role_file}"
  end
```

## Working configuration

When everything is set up, you will be happy to get the following results with the command: 

> vagrant up

```shell
PS D:\vagrant_projects\perl_projects> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/focal64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/focal64' version '20220804.0.0' is up to date...
==> default: Setting the name of the VM: perl_projects_default_1659985304505_52969
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 3000 (guest) => 3000 (host) (adapter 1)
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection reset. Retrying...
    default: Warning: Connection aborted. Retrying...
    default: Warning: Connection reset. Retrying...
    default: Warning: Connection aborted. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
==> default: Mounting shared folders...
    default: /vagrant => D:/vagrant_projects/perl_projects
==> default: Running provisioner: ansible_local...
    default: Installing Ansible...
    default: Installing pip... (for Ansible installation)
    default: Running ansible-galaxy...
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the
controller starting with Ansible 2.12. Current version: 2.7.18 (default, Jul  1
 2022, 12:27:04) [GCC 9.4.0]. This feature will be removed from ansible-core in
 version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
/usr/local/lib/python2.7/dist-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-docker-2.7.0.tar.gz to /home/vagrant/.ansible/tmp/ansible-local-8337dY3egI/tmpOS1yA1/community-docker-2.7.0-wm2luu
Installing 'community.docker:2.7.0' to '/home/vagrant/.ansible/collections/ansible_collections/community/docker'
community.docker:2.7.0 was installed successfully
    default: Running ansible-playbook...
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the
controller starting with Ansible 2.12. Current version: 2.7.18 (default, Jul  1
 2022, 12:27:04) [GCC 9.4.0]. This feature will be removed from ansible-core in
 version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
/usr/local/lib/python2.7/dist-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 20.04 on host default should use
/usr/bin/python3, but is using /usr/bin/python for backward compatibility with
prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information.
 This feature will be removed in version 2.12. Deprecation warnings can be
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [default]

TASK [Install aptitude] ********************************************************
changed: [default]

TASK [Install required system packages] ****************************************
changed: [default]

TASK [Add Docker GPG apt Key] **************************************************
changed: [default]

TASK [Add Docker Repository] ***************************************************
changed: [default]

TASK [Update apt and install docker-ce] ****************************************
changed: [default]

TASK [Install Docker Module for Python] ****************************************
changed: [default]

TASK [Pull default Docker image] ***********************************************
changed: [default]

PLAY RECAP *********************************************************************
default                    : ok=8    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

PS D:\vagrant_projects\perl_projects>
```

## Validate our Docker environment

You connect to your vagrant guest with ssh and start the well known [**Hello World**] container example.

### Connect to Vagrant guest

The connection with vagrant is made with the command:

> vagrant ssh

```powershell
PS D:\vagrant_projects\perl_projects> vagrant ssh
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Aug  8 22:06:38 UTC 2022

  System load:  0.08              Processes:                119
  Usage of /:   8.0% of 38.70GB   Users logged in:          0
  Memory usage: 28%               IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                IPv4 address for enp0s3:  10.0.2.15


2 updates can be applied immediately.
1 of these updates is a standard security update.
To see these additional updates run: apt list --upgradable
```

### Run the container hello-world

You just have to use following command:

> sudo docker run hello-world

```bash
vagrant@ubuntu-focal:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.4 LTS
Release:        20.04
Codename:       focal
vagrant@ubuntu-focal:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:53f1bbee2f52c39e41682ee1d388285290c5c8a76cc92b42687eecf38e0af3f0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

vagrant@ubuntu-focal:~$
```

## Conclusion

All files are available from my GitHub Account [Here]().
I will soon work on improving the settings to eliminate the warnings about the version of Python used.