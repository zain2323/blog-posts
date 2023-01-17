# How to create a virtual machine using KVM on Ubuntu 22.04 LTS

## Introduction

**Kernel Based Virtualization or KVM** is an open-source virtualization solution for Linux on x86 hardware containing virtualization extensions. It lets you turn Linux into a type-1 hypervisor that allows a host machine to run many instances of Virtual Machines which are all isolated from each other.

KVM is a feature built right into the Linux Kernel and we can run Virtual Machines without any third-party solution. Many third-party solutions let us do the same but KVM offers a high-speed interface to the Linux Kernel to run VMs at near-native speeds

**Prerequisites**

* To run KVM, your CPU must support virtualization technology. Most CPUs nowadays support this technology but often it is disabled in the BIOS section so you need to find out a way to enable this by yourself as this varies with every manufacturer.
    
* To check whether your CPU supports virtualization: `egrep -c '(vmx|svm)' /proc/cpuinfo` A result of *1 or more* means that the CPU support virtualization extensions. A result of *0* means it does not.
    
* Alternatively, you can also check whether your CPU supports KVM by: `kvm-ok` In my case the output of this command is:
    

![KVM-Ok](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188178600/fhJ_rDTfr.png align="left")

The output clearly indicates that my CPU supports KVM.

* The default directory for KVM Virtual Machine images is **/var/lib/libvirt/images**.
    
* Your root file system should have at least **10G** of free space to create a single VM.
    

**Setting up KVM Server** KVM is built-in into the Linux Kernel but we still need to install some dependencies. We’ll need to install **libvirt** and **QEMU**. Libvirt gives us access to the virtualization platforms while QEMU emulates the machine’s processor.

So let’s begin installing them `sudo apt install bridge-utils libvirt-clients libvirt-daemon-system qemu- system-x86` To verify if everything installed as expected check if the **libvirtd** service is running. `sudo systemctl status libvirtd`

![Libvirtd service status](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188180172/cm2DhfDa2.png align="left")

As you can see libvirt service is currently running but for now, we need to stop this as there is some configuration left. `sudo systemctl stop libvirtd`

Now we need to add two required groups on our machine which are **kvm** and **libvirt**. It might be possible that these groups have already been created by the packages we have just installed. To verify if the groups already exist:

```bash
cat /etc/group | grep 'kvm\|libbvirt'
```

![KVM and Libvirtd groups](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188182231/PcqelKZUA.png align="left")

In my case, both of the groups already exist so I don’t need to create them again.

If the output of the command is empty then this means the groups were not created and you have to now manually create them by running the following commands.

```bash
sudo groupadd kvm
sudo groupadd libvirt
```

The primary user of the system should be the part of these groups. If not, then you can add your user to the required groups. If you don’t know your username, you can use the output of the `whoami` command.

```bash
sudo usermod -aG kvm <username>
sudo usermod -aG libvirt <username>
```

To let the changes take effect log out and then log in again.

We need to ensure that the users of the kvm group have access to the **/var/lib/libvirt/images** directory. First we’ll add kvm group to this directory.

```bash
sudo chown :kvm /var/lib/libvirt/images
```

Now, we’ll set the permissions so that anyone in the kvm group will be able to modify its contents.

```bash
sudo chmod g+rw /var/lib/libvirt/images
```

Now we can start the libvirtd service again.

```bash
sudo systemctl start libvirtd
```

To verify if the service is running as expected check the status of the libvirtd service.

```bash
sudo systemctl status libvirtd
```

![Libvirtd service status after configuration](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188183969/QX_Dyd-PO.png align="left")

## Setting up Virt-Manager

Virtual Machine Manager is a GUI-based utility program that lets us manage our KVM Virtual Machines running either locally or on some remote server. To install this utility run the following command:

```bash
sudo apt install ssh-askpass virt-manager
```

Next open **virt-manager** either through the application windows or simply run `virt-manager` in your shell.

If there is some error while opening virt-manager you can simply ignore it.

![virt manager error while opening it first time](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188185510/MafjhQAsx.png align="left")

## Connect to VM on a remote machine over SSH

> If you are not making a VM on some remote server and using your own local machine then you can skip steps 1–6 and you can directly start by adding ISO in the storage group. If you have configured all of the above steps to set up VM on your local machine then in virt-manager the first connection “QEMU/KVM” is your local host and you can add the ISO in the same way as for the remote connection.

![virt manager main window](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188187524/Fo9sjlPD4.png align="left")

1\. To create a new connection, click on File and then **Add connection**. 2. A new window will appear where you can add your connection details. 3. Select the checkbox *Connect to a remote host over SSH* and fill in the required fields. 4. You will need to make sure that you are able to SSH into the server correctly otherwise this will not work. 5. You will also be prompted to enter your server machine password. 6. After this, you will have a new connection in your virt-manager application. 7. Now we need to add an ISO image in the storage group so we can install the Operating System. 8. To create *storage* group, double-click on your server connection and go to the storage tab. 9. To add ISO to our storage pool click the *plus* symbol in the bottom left corner.

![Click green plus icon to create storage group](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188188953/Mo_kk8c6y.png align="left")

10\. In the Name field type *ISO*. 11. In the *Target Path* field type /**var/lib/libvirt/images/ISO**.

![Creating storage pool](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188190454/4uoDutYtJ.png align="left")

12\. Click **Finish** to save all the changes. 13. If you followed all the steps correctly, you will see the storage pool that we just created in the left window pane.

![ISO storage pool in the left window pane](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188191900/P8uFjsTGz.png align="left")

14\. Now we need to update the permissions in the **/var/lib/libvirt/images/ISO** directory. `sudo chown root:kvm /var/lib/libvirt/images/ISO sudo chmod g+rw /var/lib/libvirt/images/ISO` 15. One last thing we need to do is just copy the ISO image we need to install in the **/var/lib/libvirt/images/ISO** directory. You can use the `scp` command to copy the ISO image from your local machine to some remote server. If you are following this article on your local machine then you can use the `cp` command to copy the ISO file.

## Creating a virtual machine

1\. In virt-manager, right click on your server connection and click on New to create a new VM.

![Creating new VM](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188193410/DROp-vne2.png align="left")

2\. The default selection will be **Local install media (ISO image or CDROM)**, leave this as is and click on **Forward**.

3\. In the next step, you can select your ISO image from the storage pool. Click on **browse** and on the new window select the **ISO** storage pool in the left pane and you should be able to see your ISO image. If you still are unable to see your ISO images, click on the **refresh** button.

![ISO storage pool](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188195165/fnfbdQBkk.png align="left")

4\. Select the ISO image you wish to install and then click on **Choose Volume**. After this click on **Forward**.

5\. In the next window you can configure the amount of **RAM** you want to allocate to the VM and the number of **CPU** cores. At a minimum, *allocate at least 1GB of RAM and 1 CPU core*.

![RAM and CPU allocation](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188196728/LQSoqssph.png align="left")

6\. In the next step you have to allocate the *amount of storage*. You can change this according to your requirements.

![Allocating storage](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188198593/T-T0o4uZB.png align="left")

7\. In the next step you have to give a *name* to your VM.

![Giving VM name](https://cdn.hashnode.com/res/hashnode/image/upload/v1673188199967/OoAMK6MPa.png align="left")

8\. Finally, click on **Finish**. After this, your VM will automatically boot into the installation menu of the Operating System. You can then follow the installation process of your VM’s operating system.

This is it for now. I hope you have found the article useful and if so please give it a like. If you have any confusion then feel free to ask.