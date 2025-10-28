# PrimePi
Script to find prime numbers under a certain limit made to performance test computers. The below example is for an 8-node Raspberry Pi cluster running Raspberry Pi OS and MPI (Message Passing Interface). The setup uses one master (host) node and seven worker nodes.

🧩 Hardware Setup

1x Master Node: Raspberry Pi with Raspberry Pi OS (Full)

7x Worker Nodes: Raspberry Pis with Raspberry Pi OS Lite

Network: All connected via Ethernet

Assumed IP Range: 192.168.0.1 – 192.168.0.8

⚙️ Step 1: Prepare Each Node for SSH Access

Boot each Pi and run the following to update the system:

sudo apt -y update
sudo apt -y upgrade


Open Raspberry Pi configuration:

sudo raspi-config


Change the password

Set a unique hostname (e.g., Node1, Node2, etc.)

Enable SSH under Interface Options

Assign a static IP address to each Pi:

sudo nano /etc/dhcpcd.conf


Add the following lines to the end of the file (adjust the IP for each node):

interface eth0
static ip_address=192.168.0.1/24


Reboot each Pi:

sudo reboot


Verify all nodes are online from the master node:

nmap 192.168.0.1-8

🔑 Step 2: Set Up SSH Key-Based Authentication

This allows passwordless communication between the master and worker nodes.

Create SSH keys on the master node:

ssh-keygen -t rsa


Press Enter for all prompts (no passphrase needed).

Repeat the same on each worker node:

ssh-keygen -t rsa


Copy each node’s SSH key to the master node:

ssh-copy-id 192.168.0.1


From the master node, copy its SSH key to each worker node:

ssh-copy-id 192.168.0.2
ssh-copy-id 192.168.0.3
ssh-copy-id 192.168.0.4
ssh-copy-id 192.168.0.5
ssh-copy-id 192.168.0.6
ssh-copy-id 192.168.0.7
ssh-copy-id 192.168.0.8


You should now be able to SSH into any node from the master without entering a password:

ssh 192.168.0.2

📦 Step 3: Install MPI (Message Passing Interface)

MPI allows the cluster to distribute and coordinate computational tasks.

On the master node:

sudo apt install mpich python3-mpi4py


Repeat the same installation on each worker node via SSH.

Verify MPI communication across all nodes:

mpiexec -n 8 --host 192.168.0.1,192.168.0.2,192.168.0.3,192.168.0.4,192.168.0.5,192.168.0.6,192.168.0.7,192.168.0.8 hostname


You should see each node’s hostname listed in the output.

🧮 Step 4: Copy and Test the Prime Calculation Script

Copy the prime.py script to each node:

scp ~/prime.py 192.168.0.2:
scp ~/prime.py 192.168.0.3:
scp ~/prime.py 192.168.0.4:
scp ~/prime.py 192.168.0.5:
scp ~/prime.py 192.168.0.6:
scp ~/prime.py 192.168.0.7:
scp ~/prime.py 192.168.0.8:


Test locally on any node:

mpiexec -n 1 python3 prime.py 1000


If it runs successfully, you’re ready to run your first cluster test.

🚀 Step 5: Run the Cluster Test

Run the prime number calculation across all 8 nodes:

mpiexec -n 8 --host 192.168.0.1,192.168.0.2,192.168.0.3,192.168.0.4,192.168.0.5,192.168.0.6,192.168.0.7,192.168.0.8 python3 prime.py 10000


The cluster should complete the task in less than a second — demonstrating parallel processing performance.

🧠 Notes

Ensure all Pis are on the same subnet and can ping each other.

If any node doesn’t respond, recheck SSH keys and IP configuration.

You can expand the cluster by repeating the same setup for additional nodes.
