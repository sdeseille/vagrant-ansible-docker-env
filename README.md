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

After some iterations, I found that it is necessary to add some specifics options. They are needed to make the installation possible. You should got following configuration.

```ruby
  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.install_mode = "pip"
    ansible.pip_install_cmd = "curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | sudo python"
  end
```


## Working configuration

When all is in place you will be pleased to obtain following results.

```ansible
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