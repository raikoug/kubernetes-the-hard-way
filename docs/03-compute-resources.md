# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. In this lab you will provision the machines required for setting up a Kubernetes cluster.

## Machine Database

This tutorial will leverage a text file, which will serve as a machine database, to store the various machine attributes that will be used when setting up the Kubernetes control plane and worker nodes. The following schema represents entries in the machine database, one entry per line:

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

Each of the columns corresponds to a machine IP address `IPV4_ADDRESS`, fully qualified domain name `FQDN`, host name `HOSTNAME`, and the IP subnet `POD_SUBNET`. Kubernetes assigns one IP address per `pod` and the `POD_SUBNET` represents the unique IP address range assigned to each machine in the cluster for doing so.  

Here is an example machine database similar to the one used when creating this tutorial. Notice the IP addresses have been masked out. Your machines can be assigned any IP address as long as each machine is reachable from each other and the `jumpbox`.

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

Now it's your turn to create a `machines.txt` file with the details for the three machines you will be using to create your Kubernetes cluster. Use the example machine database from above and add the details for your machines. 

## Configuring SSH Access

SSH will be used to configure the machines in the cluster. Verify that you have `root` SSH access to each machine listed in your machine database. You may need to enable root SSH access on each node by updating the sshd_config file and restarting the SSH server.

### Enable root SSH Access

If `root` SSH access is enabled for each of your machines you can skip this section.

By default, a new `debian` install disables SSH access for the `root` user. This is done for security reasons as the `root` user is a well known user on Linux systems, and if a weak password is used on a machine connected to the internet, well, let's just say it's only a matter of time before your machine belongs to someone else. As mention earlier, we are going to enable `root` access over SSH in order to streamline the steps in this tutorial. Security is a tradeoff, and in this case, we are optimizing for convenience. On each machine login via SSH using your user account, then switch to the `root` user using the `su` command:

```bash
su - root
```

Edit the `/etc/ssh/sshd_config` SSH daemon configuration file and the `PermitRootLogin` option to `yes`:

```bash
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

Restart the `sshd` SSH server to pick up the updated configuration file:

```bash
systemctl restart sshd
```

### Generate and Distribute SSH Keys

In this section you will generate and distribute an SSH keypair to the `server`, `node-0`, and `node-1`, machines, which will be used to run commands on those machines throughout this tutorial. Run the following commands from the `jumpbox` machine.

Generate a new SSH key:

```bash
ssh-keygen -t ed25519
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

Generate the jumpbox `/etc/hosts` entries and a temporary file `/tmp/hosts`
assuming jumpbox is `172.16.100.100`:

```bash
echo "" >> /etc/hosts
echo "# Kubernetes The Hard Way" >> /etc/hosts
echo "172.16.100.100 jumpbox.internal jumpbox" > /tmp/hosts
while read IP FQDN HOST SUBNET; do
  echo "${IP} ${FQDN} ${HOST}" >> /etc/hosts
  echo "${IP} ${FQDN} ${HOST}" >> /tmp/hosts
done < machines.txt
```

Now `/etc/hosts` should looks like this (with you own IP addresses):

```text
127.0.0.1	localhost
172.16.0.100	jumpbox.internal	jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes The Hard Way
172.16.100.101 server.internal server
172.16.100.102 node-0.internal node-0
172.16.100.103 node-1.internal node-1
```

Test reachability with port 22 (SSH) to each machine:

```bash
while read IP FQDN HOST SUBNET; do 
  nc -zv ${FQDN} 22
done < machines.txt
```

Result should be:

```text
server.internal [172.16.100.101] 22 (ssh) open
node-0.internal [172.16.100.102] 22 (ssh) open
node-1.internal [172.16.100.103] 22 (ssh) open
```

Copy the SSH public key to each machine:

```bash
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${FQDN}
done < machines.txt
```

Once each key is added, verify SSH public key access is working:

```bash
while read IP FQDN HOST SUBNET; do 
  ssh -n root@${FQDN} uname -o -m
done < machines.txt
```

```text
x86_64 GNU/Linux
x86_64 GNU/Linux
x86_64 GNU/Linux
```

## Hostnames

In this section you will assign hostnames to the `server`, `node-0`, and `node-1` machines. The hostname will be used when executing commands from the `jumpbox` to each machine. The hostname also play a major role within the cluster. Instead of Kubernetes clients using an IP address to issue commands to the Kubernetes API server, those client will use the `server` hostname instead. Hostnames are also used by each worker machine, `node-0` and `node-1` when registering with a given Kubernetes cluster.

To configure the hostname for each machine, run the following commands on the `jumpbox`.

Set the hostname on each machine listed in the `machines.txt` file and copy the hosts:

```bash
while read IP FQDN HOST SUBNET; do 
    # Command to add the hostname to /etc/hosts
    CMD0="echo ''>> /etc/hosts; echo '# Kubernetes The Hard Way' >> /etc/hosts; echo '127.0.1.1 ${FQDN} ${HOST}' >> /etc/hosts"
    ssh -n root@${FQDN} "$CMD0"
    # Copy the temporary hosts file to each machine
    scp /tmp/hosts root@${FQDN}:/tmp/hosts
    ssh -n root@${FQDN} "cat /tmp/hosts >> /etc/hosts"
    # Set the hostname
    ssh -n root@${FQDN} hostnamectl hostname ${HOST}
done < machines.txt
```

Verify the hostname is set on each machine:

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${FQDN} hostname --fqdn
done < machines.txt
```

```text
server.internal
node-0.internal
node-1.internal
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
