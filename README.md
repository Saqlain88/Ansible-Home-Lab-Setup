# Ansible-Home-Lab-Setup

Set up your own Ansible lab for hands on practises.

# Pre-requisites
- Oracle VirtualBox v7.0.22
- Vagrant v2.3.7

# Tools required
- Putty 
- WinSCP

# Description

Below are the few steps that we'll be doing before jumping into any action.<br />
<br />
**Step 1:**  Install Vagrant and VirtualBox (version doesn't matter, as long as they are COMPATIBLE with one another). <br />
<br />
**Step 2:**  Install Putty and WinSCP. We are going to use them to configure our password less ssh conectivity. <br />
<br />
**Step 3:**  If you are using Windows OS, go to C:\Users\%username%\ and search for .ssh folder (its a hidden folder, make sure to check properly.) <br />
<br />
**Step 4:**  If you didn't find the .ssh folder in Step 3, open command prompt and run `ssh-keygen -t rsa`. Now it must me asking some questions over there (BORINGGGG). We will keep smashing enter, until all the question ends (YES we are inattentive and careless). <br />
<br />
**Step 5:**  Now that .ssh is there, open it and you must be having id_rsa files over there. <br />
<br />
**Step 6:**  We created a folder AnsibleLab (anywhere you want). Opened it. then we opened command prompt at this location and run `vagrant init`. Vagrantfile file was generated inside AnsibleLab folder. <br />
<br />
**Step 7:**  Opened the Vagrantfile with notepad and pasted below code: <br />

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
<br />
Oh, brilliant! Two web servers, a load balancer, a database, a queue, and of course, the almighty control machine—just to install Ansible. What efficiency! All for a <i>simple web application</i>.
<br />

**Step 8:** Now to create these vms in our VirtualBox, we went back to AnsibleLab folder and ran below in command prompt:<br />
<br />
`vagrant up`

After this all the vms will be up and running, we can verify the same by opening your VirtualBox. <br />

![image](https://github.com/user-attachments/assets/adc2604b-bd0b-4fd5-81fd-de77b46450cf)

**Step 9:** If we want to connect to any of the server then we can just run below:<br />
`vagrant ssh <VM_NAME>`

for example: `vagrant ssh control`

**Step 10:** Checked `C:\Users\%username%\.vagrant.d` for **insecure_private_key** file.<br />
**Step 11:** Opened PuTTYgen to create a key pair. Click load and select insecure_private_key file from Step 10. Saved private key and didn't passphrase it.<br />
**Step 12:** OPened PuTTY. Typed 127.0.0.1 with port 2215 for control server. Then under Connection > SSH > Auth > Credentials under 'Private keyfile for authentication' selected the saved private key from Step 11. Cameback to Session, and saved the session with name control. Repeated this step and created the session for other servers as well. Now we can easily open session without repetitively using vagrant ssh for all servers.<br />

![image](https://github.com/user-attachments/assets/b88d8aed-7dd1-4bbd-ad53-099cca28c214)
<br /><br />

**Step 12:** Time to setup ansible.<br />

Connected to control server and run below:<br />

```
sudo yum install epel-release 
sudo yum install ansible
```

if you run into any issues like ansible doesnot exist in package or something. Then update your yum. If you were unable to update yum. Check the end of this page and that might help you. <br /><br />

**Step 13:** Now in order to ansible to connect to other machines. We had to do little something.<br />

We used WinSCP and copied the `id_rsa` file from `C:\Users\%username%\.ssh\` folder (our local machine) to control server inside `/home/vagrant/.ssh/` directory. Now modified the permission of id_rsa file by running `chmod 700 id_rsa`. 

**Step 14:** We must configure our ansbile inventory now that password-less ssh is enabled.<br />

Updated the /etc/ansible/hosts file with below: <br />

```
[database]
192.168.56.101

[webservers]
192.168.56.102
192.168.56.103

[loadbalancer]
192.168.56.104

[queue]
192.168.56.105

[control]
192.168.56.106
```
<br />

**Step 15:** How we will validate ?? run below in control server to ping all the servers. <br />

```
ansible -m ping all
```
<br />
**Note:** Make sure you've already ran <i>vagrant up</i> and all servers are running. Check that by running <i>vagrant status</i>. And to stop all vms run <i>vagrant halt</i>. <br />

# Issue
<br />



