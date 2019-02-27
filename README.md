# MPI-cluster-learn

## Setting up an MPI Cluster in Ubuntu
The following is done with two same system (Ubuntu 18.04).

#### 1. Rename Hosts to ensure unique names in the cluster.
Master machine may be named master and slaves as slave1,slave2,.. The following shows the
modifications in master system. The same should be done on all slaves.

a sudo nano hostname
b replace the current hostname with master, save and exit
c sudo nano /etc/hosts
```g
  i Give name and IP address of all nodes in the cluster
    For example, in master
         127.0.0.1 localhost
         192.168.1.69 master
         192.168.1.53 slave1
         192.168.1.50 slave2

    And in slave, say slave1
         127.0.0.1 localhost
         192.168.1.69 master
         192.168.1.53 slave1
```
 ii Comment out (#) any other entries starting with 127.
    e.g. #127.0.1.1 administrator-system-product-name

 iii keep other entries in tact
d Restart the system


#### 2 Add the same user mpiuser in all nodes (optional)
a sudo adduser mpiuser
b Set the password
c Just press Enter for all other fields
We did not do this step, because all systems are same (clone), having same user.

#### 3. Installing SSH Server
Run this command in all nodes in order to install OpenSSH Server
```g
sudo apt­-get install openssh-server
```
If we want to unistall openssh-server, we can use command:
```g
sudo apt-get --purge remove openssh-server
```
#### 4. Setting up passwordless SSH for communication between nodes
i. we generate an RSA key pair
```g
ssh­-keygen ­-t rsa
```
Use default locations and keep pass phrase Blank(just press Enter)
ii. Copy key to slaves
```g
ssh-copy-id slave1
ssh-copy-id slave2
```
iii. ssh all machines ones (They get added to list of known hosts)
```g
ssh slave1
ssh slave2
```
iv. Keychain
If you are asked to enter a passphrase every time, you need to set up a keychain. This is done easily by installing... Keychain.

sudo apt-get install keychain
And to tell it where your keys are and to start an ssh-agent automatically edit your ~/.bashrc file to contain the following lines (where id_rsa is the name of your private key file):
```g
if type keychain >/dev/null 2>/dev/null; then
  keychain --nogui -q id_rsa
  [ -f ~/.keychain/${HOSTNAME}-sh ] && . ~/.keychain/${HOSTNAME}-sh
  [ -f ~/.keychain/${HOSTNAME}-sh-gpg ] && . ~/.keychain/${HOSTNAME}-sh-gpg
fi
```
Exit and login once again or do a source ~/.bashrc for the changes to take effect.
Now your hostname via ssh command should return the other node's hostname without asking for a password or a passphrase. Check that this works for all the slave nodes.