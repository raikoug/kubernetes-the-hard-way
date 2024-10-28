# Set Up The Jumpbox

In this lab you will set up one of the four machines to be a `jumpbox`. This machine will be used to run commands in this tutorial. While a dedicated machine is being used to ensure consistency, these commands can also be run from just about any machine including your personal workstation running macOS or Linux.

Think of the `jumpbox` as the administration machine that you will use as a home base when setting up your Kubernetes cluster from the ground up. One thing we need to do before we get started is install a few command line utilities. 

Log in to the `jumpbox`:

```bash
ssh root@jumpbox
```

All commands will be run as the `root` user. This is being done for the sake of convenience, and will help reduce the number of commands required to set everything up.

### Install requirements

Now that you are logged into the `jumpbox` machine as the `root` user, you will install the command line utilities that will be used to preform various tasks throughout the tutorial. 

```bash
apt -y install wget curl vim openssl git apt-transport-https ca-certificates gnupg cri-tools containerd
```

### Kebernetes Source List
Source: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux

```bash
# Download the key and add it to the keyring
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Add kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
chmod 644 /etc/apt/sources.list.d/kubernetes.list
# Update and install
apt update
```

#### Install the necessary Packages

```bash
apt install -y kubectl kubelet
```

Test `kubectl`
```bash
kubectl version --client
```

### Other Tools
#### Sync GitHub Repository

Now it's time to download a copy of this tutorial which contains the configuration files and templates that will be used build your Kubernetes cluster from the ground up. Clone the Kubernetes The Hard Way git repository using the `git` command:

```bash
git clone --depth 1 \
  https://github.com/raikoug/kubernetes-the-hard-way
```

Change into the `kubernetes-the-hard-way` directory:

```bash
cd kubernetes-the-hard-way
```

This will be the working directory for the rest of the tutorial. If you ever get lost run the `pwd` command to verify you are in the right directory when running commands on the `jumpbox`:

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```


### Download Binaries

In this section you will download the binaries for the various Kubernetes components. The binaries will be stored in the `downloads` directory on the `jumpbox`, which will reduce the amount of internet bandwidth required to complete this tutorial as we avoid downloading the binaries multiple times for each machine in our Kubernetes cluster.

From the `kubernetes-the-hard-way` directory create a `downloads` directory using the `mkdir` command:

```bash
mkdir downloads
```

The binaries that will be downloaded are listed in the `downloads.txt` file, which you can review using the `cat` command:

```bash
cat downloads.txt
```

Download the binaries listed in the `downloads.txt` file using the `wget` command:

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

Depending on your internet connection speed it may take a while to download the `584` megabytes of binaries, and once the download is complete, you can list them using the `ls` command:

```bash
ls -loh downloads
```

```text
total 584M
-rw-r--r-- 1 root  41M May  9 13:35 cni-plugins-linux-arm64-v1.3.0.tgz
-rw-r--r-- 1 root  34M Oct 26 15:21 containerd-1.7.8-linux-arm64.tar.gz
-rw-r--r-- 1 root  22M Aug 14 00:19 crictl-v1.28.0-linux-arm.tar.gz
-rw-r--r-- 1 root  15M Jul 11 02:30 etcd-v3.4.27-linux-arm64.tar.gz
-rw-r--r-- 1 root 111M Oct 18 07:34 kube-apiserver
-rw-r--r-- 1 root 107M Oct 18 07:34 kube-controller-manager
-rw-r--r-- 1 root  51M Oct 18 07:34 kube-proxy
-rw-r--r-- 1 root  52M Oct 18 07:34 kube-scheduler
-rw-r--r-- 1 root  46M Oct 18 07:34 kubectl
-rw-r--r-- 1 root 101M Oct 18 07:34 kubelet
-rw-r--r-- 1 root 9.6M Aug 10 18:57 runc.arm64
```

At this point the `jumpbox` has been set up with all the command line tools and utilities necessary to complete the labs in this tutorial.

Next: [Provisioning Compute Resources](03-compute-resources.md)
