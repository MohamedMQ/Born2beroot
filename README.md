# Born2beroot

	The 42-Borntoberoot project's goal is to install a Virtual Machine and set up a small server.

Welcome to an introduction of the virtualization’s world. This project aims to present some initial concepts of system administration. Besides doing the proper arrangements for the VM, with its partition scheme, we will install the Debian OS, configure the root user, set security barriers (AppArmor, UFW, and password security policy), establish the SSH connection (through OpenSSH) and make a monitoring script for us to use the “wall” command and “cron” service. I hope you enjoy it!
		
First, we better understand what a Virtual Machine (also known as VM) is. Basically, the difference between a virtual machine and a physical one is that the first gets its physical resources (like a CPU, RAM, HD, etc.) from a host. So, a VM is like installing a computer in another existent one, a virtual computer within a physical server. A hypervisor is responsible for running the virtual machines and managing their resources, like memory and storage.

Just for the record, let me tell you a bit more about the benefits of using a VM. After all, you might be wondering why someone would need a virtual machine. Although many reasons could be listed, I can briefly say that with it you are able to run a different OS than the physical computer in a VM without having to restart its hard drive. You can also install several virtual machines on a limited number of physical computers, saving space and probably some electricity. And a final reason to use it could be the possibility of installing an old OS that doesn’t have a compatible hardware where to put it. Of course, there are drawbacks on using a virtual machine, especially regarding performance (once it shares with the host its physical resources) and security (as there are more vulnerabilities than if you were using a single OS).
		
Anyway, let’s get back to the project. We were told to choose between two different OSs to install in the VM that we created using Virtual Box. Debian or CentOS might be chosen. I decided to go with Debian because I once started to learn something about Ubuntu, and so I thought it would be easier for me to use Debian (just imagine yourself using yum rather than apt hahaha it’s not that intuitive to me). Also, it seems that the Debian community is more widespread.
	
To create the Virtual Machine, I’ve used Oracle Virtual Box. The downloaded Debian disk image (.iso) is added in the initialization of a new VM at the Virtual Box, and some configuration is done. In <type> we must select Linux, and Debian in <version>. I’ve allocated 1024MB for RAM and 30GB of storage (although probably that’s too much, I’ve tried to do the same as it appears in the subject).
	
After a few more settings adjusted, we now must correctly mount the partitions which we’ll be using. The purpose of having different partitions is basically preventing an overall corruption. If any part gets compromised, it could be that other does not. A 500 MB non-encrypted partition is reserved to boot (it keeps the static files to the boot loader). The others will be encrypted, and they are destinated for: root (the home directory for the root user), home (hosting the user home directories), swap (a special partition, like a workspace for the OS in case of RAM overflow), var (containing variable data), srv (with data for services provided by the system), temp (to the temporary files) and var/log (for the log files). Sda1, sda2 and sda5 are now mounted and we can configure the Logical Volume Manager creating the Volume Group and arranging the respective logical volumes as we need.  After this, the installation offers different services, but we won’t install any of them because we’ll install the ones we need later.

To properly set our Debian we used several commands that will be listed below:

	GENERAL
	
To check the OS information: cat /etc /os-release
	
To check our partitions: lsblk
	
To check if the packet manager is installed: apt –version
	
To quit the current session: exit
	
To connect as a different user: su other_username
	
To restart the system: sudo systemctl reboot

	SUDO

To install sudo: apt install sudo (as root)
	
To add a user to sudo group: sudo usermod -aG sudo <username>
	
To change sudo’s configuration: sudo visudo
	

	APPARMOR
	
To see if AppArmor is intalled: sudo aa-status
	

	UFW
	
To install ufw: apt install ufw
	
To turn ufw on: sudo ufw enable
	
To see ufw status: sudo ufw status verbose
	
To make a rule: sudo ufw default deny/allow outgoing
	
To make a port rule: sudo ufw allow/deny 4242
	
To remove a rule: sudo ufw delete allow/deny 8080
	

	SSH
	
To install OpenSSH: sudo apt install openssh-server
	
To check if it’s running: sudo systemctl status ssh
	
To change ssh configurations: sudo nano /etc/ssh/sshd_config
	
To restart ssh service: sudo systemctl restart ssh
	
To access the virtual machine via ssh: ssh <username_server>@<server_IP_adress> -p <ssh-port>
	
To end a ssh session: exit
	

	STRONG PASSWORD POLICY
	
To change password rules: sudo nano /etc/login.defs
	
To modify the parameters of root and preexisting users: sudo chage -M/-m/-W <username/root>
	
To install a library for password verification: sudo apt intall libpam-pwquality
	
To edit the configuration file: sudo nano /etc/security/pwquality.conf
	
To change the password: sudo passwd <user/root>

	
	HOSTNAME
	
To change hostname: sudo hostnamectl set-hostname <newhostname>
	
To check the current hostname: hostname ctl status
	
	
	USER MANAGEMENT
	
To create a new user: useradd <username>
	
To change the users parameters: usermod -l (for the username) -c (for the full name)
	
To del a user: userdel -r <username>
	
To display all the users on the machine: cat /etc/passwd | cut -d “:” -f1
	

	
	GROUP MANAGEMENT
	
To create a new group: groupadd <groupname>
	
To add a user to a group: gpasswd -a <username>
	
To remove a user from a group: gpasswd -d <username>
	
To delete a group: groupdel <group_name>
	
To display a list of all users in a group: getent group <groupname>
	

	
	CRON / COMANDS TO MONITORING SCRIPT
	
To set a routine within cron: crontab -e
	
To see the architecture of a OS: uname -a
	
To see the number of physical processors: grep "physical id" /proc/cpuinfo | sort | uniq | wc -l
	
To see the number of virtual processors: grep "^processor" /proc/cpuinfo | wc -l
	
To see the ram available: free -m | awk '$1 == "Mem:" {print $2}'
	
To see the total ram: free -m | awk '$1 == "Mem:" {print $3}'
	
To calculate the percentage of used ram: (free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}'
	
To see the available memory on the server: df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $4} END {print ft}'
	
To see the total memory: df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}'
	
To see the CPU usage: top -bn1 | grep '^%Cpu' | cut -c 9- | awk '{printf("%.1f%%"), $1 + $2 + $3}'
	
To see the date and time of last boot: who -b | awk '$1 == "system" {print $3 " " $4}'
	
To see if LVM is active or not:  if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi
	
To see how many TCP connections are established: cat /proc/net/sockstat{,6} | awk '$1 == "TCP:" {print $3}'
	
To see the number of logged users: users | wc -w
	
To see the IP of this machine: hostname -I
	
To see the MAC address: ip link show | awk '$1 == "link/ether" {print $2}'
	
To count how many commands with sudo were used in the current section: journalctl _COMM=sudo | grep COMMAND | wc -l
	
	The submission we made was only the sha1 of .vdi file of the corresponding VM. That is the content of Signature.txt.
	In order to get that we did the command: certUtil -hashfile Borntoberoot.vdi sha1 in %HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\.
