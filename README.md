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


## Setup NFS
a In master, setup NFS Server

i
```g
 sudo apt-get install nfs-kernel-server
```
ii 
```g
 mkdir /home/song/storage
```
  storage is the shared folder, to be mounted in all slaves
iii edit /etc/exports and add a new entry for /home/song/storage as shown:

1 open /etc/exports
```g
sudo nano /etc/exports
```
2 add the following:
```g
 /home/song/storage *(rw,sync,no root squash,no subtree check)
```
   Here, instead of * you can specifically give out the IP address to which you want to share this folder to. But, this will just make our job easier.

   -rw: This is to enable both read and write option. ro is for read-only.

   -sync: This applies changes to the shared directory only after changes are committed.

   -no_subtree_check: This option prevents the subtree checking. When a shared directory is the subdirectory of a larger filesystem, nfs performs scans of every directory above it, in order to verify its permissions and details. Disabling the subtree check may increase the reliability of NFS, but reduce security.

   -no_root_squash: This allows root account to connect to the folder.

  3 save & exit

iv then run the following:

```g
sudo exportfs -a
```
v restart nfs server
```g
sudo service nfs-kernel-server restart
```

b In slaves, setup NFS client

i 
```g
sudo apt-get install nfs-common
```
ii
```g
 mkdir /home/song/storage
```
storage is the shared folder, to be mounted from master

iii exit (from mpiuser)
iv add the following entry to /etc/fstab so that the mounting of master’s storage folder to the client will be done everytime the system boots.
1 sudo nano /etc/fstab
2 master:/home/song/storage /home/song/storage nfs
3 save & exit
vi Restart the slave machine


## Testing MPI programs
Before we need to install MPI in all nodes, here we install PM2: https://gforge.inria.fr/frs/?group_id=30&release_id=10507

a login to master as song
b cd storage
c create a program, say helollo.c
```g
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialize the MPI environment
    MPI_Init(NULL, NULL);

    // Get the number of processes
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Get the rank of the process
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Get the name of the processor
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    // Print off a hello world message
    printf("Hello world from processor %s, rank %d out of %d processors\n",
           processor_name, world_rank, world_size);

    // Finalize the MPI environment.
    MPI_Finalize();
}
```
d mpicc hello.c -o hello
e mpirun  --hosts master,slave1 ./hello

Or 

Create a file called "machinefile" in the program folder:
```g
master 
slave1 
slave2
slave1
slave2 
... 
```
mpirun -f machinefile ./hello


