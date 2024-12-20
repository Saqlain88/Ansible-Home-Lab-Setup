# Ansible-Home-Lab-Setup

Set up your own Ansible lab for hands on practises.

# Pre-requisites
- Oracle VirtualBox v7.0.22
- Vagrant v2.3.7

# Tools required
- Putty 
- WinSCP

# Description

Below are the few steps that we'll be doing before jumping into any action.

Step 1: Install Vagrant and VirtualBox (version doesn't matter, as long as they are COMPATIBLE with one another).
Step 2: Install Putty and WinSCP. We are going to use them to configure our password less ssh conectivity.
Step 3: If you are using Windows OS, go to C:\Users\%username%\ and search for .ssh folder (its a hidden folder, make sure to check properly.) 
Step 4: If you didn't find the .ssh folder in Step 3, open command prompt and run `ssh-keygen -t rsa`. Now it must me asking some questions over there (BORINGGGG). We will keep smashing enter, until all the question ends (YES we are inattentive and careless).
Step 5: Now that .ssh is there, open it and you must be having id_rsa files over there.
Step 6: We created a folder AnsibleLab (anywhere you want). Opened it. then we opened command prompt at this location and run `vagrant init`. Vagrantfile file was generated inside AnsibleLab folder.
Step 7: Opened the Vagrantfile with notepad and pasted below code:

```vagrant
Vagrant.configure("2") do |config|
  servers=[
    {
      :hostname => "database",
      :box => "centos/7",
      :ip => "192.168.56.101",
      :ssh_port => '2210'
    },
    {
      :hostname => "webserver-1",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.102",
      :ssh_port => '2211'
    },
    {
      :hostname => "webserver-2",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.103",
      :ssh_port => '2212'
    },
    {
      :hostname => "loadbalancer",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.104",
      :ssh_port => '2213'
    },
    {
      :hostname => "queue",
      :box => "centos/7",
      :ip => "192.168.56.105",
      :ssh_port => '2214'
    },
    {
      :hostname => "control",
      :box => "centos/7",
      :ip => "192.168.56.106",
      :ssh_port => '2215'
    }

  ]

  servers.each do |machine|

    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
    
      node.vm.network :private_network, ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 512]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end
    end
  end

  id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))

  config.vm.provision :shell,
        :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

  config.ssh.insert_key = false
end
```
