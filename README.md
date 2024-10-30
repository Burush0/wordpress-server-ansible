# wordpress-server-ansible
Deployment of a WordPress server on Ubuntu Server using an Ansible playbook.

This is a continuation of the DevOps series, now deploying the server using Ansible. [Part 1](https://github.com/Burush0/wordpress-ubuntu-server) (raw step by step), [Part 2](https://github.com/Burush0/wordpress-server-bash) (bash script).

# Prerequisites:
- An [Ubuntu Server](https://ubuntu.com/download/server) installation, preferably fresh. I did mine inside a VMWare Workstation virtual machine.
- A way to generate SSH keys on your host machine. In my case of Windows 10, I had [Git](https://gitforwindows.org/) installed, that can also be achieved with [PuTTY](https://www.putty.org/).
- A working [Ansible](https://www.ansible.com/) installation. Since there's no native Windows support, I resorted to using another Ubuntu VM and installing Ansible over there.

# Setup:
## 1. Setup SSH connection between guest (VM) and host (your PC AND the Ansible VM):

### 1.1. Install OpenSSH Server:

```
sudo apt install openssh-server -y
```

### 1.2. Generate an SSH key on your host machine:
```
ssh-keygen -t rsa -b 4096
```

### 1.3. Send the public part of the key to the guest. 

```
ssh-copy-id <remote-user>@<server-ip>
```

## 2. Specify a local domain name

In `/etc/hosts` file on your system (For Windows, that would be `C:/Windows/System32/drivers/etc/hosts`), add in a line that looks like this:
```
<server-ip> <domain-name>
```
To give a more specific example, my local server ip was `192.168.139.134` and the domain name I chose was `testdomain.com`, so the line looked like:
```
192.168.139.134 testdomain.com
```
Save changes to the file, needs administrator privileges (which means you might need to run Notepad as admin).

## 3. Run the Ansible playbook

### 3.1. Clone the repo and navigate to the folder
```
git clone https://github.com/Burush0/wordpress-server-ansible.git
cd wordpress-server-ansible
```
### 3.2. Edit the `inventory.ini` file to match your server IP

In my case, it looks like this:
```
[wordpress_server]
192.168.139.134 ansible_user=burush
```
You'll have to match the IP and the username to your own.

### 3.3. Edit the variables in the `site.yaml` playbook
```
vars:
    domain_name: testdomain.com
    db_password: qwe123
```
Change those to the ones you have set previously. The password being stored like that is unsecure, but in this demo I decided against using Ansible Vault. If you want your password to be stored more securely, Google is your friend. 

Essentially, password is stored in a secret file which is referenced in the playbook and then as a variable everywhere else. In my case, it is also referenced as a variable everywhere else, but I just set it up as plaintext in the playbook.

### 3.4. Run the playbook
```
ansible-playbook -i inventory.ini play.yaml -kK 
```
I had an issue where it would get stuck at the Gathering Facts step, I was able to fix it with running the command without the `-kK` flag, the playbook aborting due to not getting sudo rights, and then re-running it with the `-kK` flag worked as intended.

Yet again, the SSL certificate generation is with my own information, if you want to customize it, head over to `roles/tasks/nginx/main.yaml`, find the entry where it gets generated with `openssl` and edit the part with the `-subj` flag.

And that's it! You should have a working Wordpress deployment at `https://<SERVER_NAME>/wordpress/`
