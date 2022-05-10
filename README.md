**Keep calm and build an SSI cluster**


To learn more about Linux:

- https://www.youtube.com/watch?v=wBp0Rb-ZJak
- https://www.youtube.com/watch?v=ZtqBQ68cfJc
- https://www.youtube.com/watch?v=ROjZy1WbCIA

To learn more about ARM and x86 machine architectures:

- https://www.arm.com/architecture
- https://en.wikipedia.org/wiki/ARM_architecture_family
- https://www.techopedia.com/definition/5334/x86-architecture
- https://en.wikipedia.org/wiki/X86
- https://stackoverflow.com/questions/14794460/how-does-the-arm-architecture-differ-from-x86
- https://en.wikipedia.org/wiki/64-bit_computing

How does git work? Version control for code
- https://rogerdudler.github.io/git-guide/

To learn more about the academic research project our project is powered by:
- https://popcornlinux-doc.readthedocs.io/en/latest/build_kernel.html 
- http://www.popcornlinux.org/index.php/overview
- https://github.com/ssrg-vt/popcorn-kernel
- (Papers) http://www.popcornlinux.org/index.php/publications


To learn more about Linux kernel development
- Learn the fundamentals of the C programming language - https://www.tutorialspoint.com/cprogramming/index.htm
- https://www.youtube.com/watch?v=_JQAve05o_0
- https://www.youtube.com/watch?v=juGNPLdjLH4
- https://www.oreilly.com/library/view/linux-kernel-development/9780768696974/ (Book)
- https://www.cs.utexas.edu/~rossbach/cs380p/papers/ulk3.pdf (Book)
- https://www.youtube.com/watch?v=598Xe7OsPuU

Thread migration:
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Compiler-Setup#initiate-thread-migration
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Compiler-Setup#build-your-own-applications


To learn more about raspberry pis:
- https://www.raspberrypi.com/documentation/computers/linux_kernel.html
The specs we would have in total on all our machines:

```
            .-/+oossssoo+/-.               aloha@makers
        `:+ssssssssssssssssss+:`           -----------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04.4 LTS x86_64/ARM
    .ossssssssssssssssssdMMMNysssso.       Host: SSI Cluster
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 5.13.0-40-generic
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 10 hours
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: bash 5.0.17
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Resolution: 1920x1080
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Terminal: /dev/pts/0
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   CPU: Very powerful/benchmark it (20 cores)
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Memory: 906MiB / 17GB
.ssssssssdMMMNhsssssssssshNMMMdssssssss.    
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/    
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.




```

For the x86 ("normal" Intel/AMD machine in the cluster):


- Grab a desktop ISO image from https://releases.ubuntu.com/20.04/ 
- Use Balena Etcher or the dd command on Linux/mac to flash the ISO to a USB stick
- Plug the USB Stick on your x86 machine and enter the boot menu on the PC (to find the function key for that just Google "pc model + enter boot menu") and go through the installation process
- Remove the USB Stick and reboot
- Update the system


```
sudo apt-get update && sudo apt-get upgrade && sudo apt install vim

```
- Set a static IP address (use nmcli or your distro's equivalent tool - avoid netplan)

```
sudo nmcli  [list interfaces/network interface cards]
sudo nmcli con  [list the available connections]
sudo nmcli con mod "NAME_OF_DESIRED_CONNECTION (in our case wifi (not infiniband :( ) name - STUDENTS)" ipv4.gateway 10.40.7.1 ipv4.addresses staticipaddress/24 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli con up "NAME_OF_DESIRED_CONNECTION" && sudo systemctl restart NetworkManager

```
- Building popcorn dependencies install (refer https://github.com/ssrg-vt/popcorn-kernel/wiki/Kernel-debugging-tips-settings)

```
sudo apt-get install quilt dkms make gcc coreutils pciutils grep perl procps lsof python-libxml2 libssl-dev libncursesw5-dev bison flex git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev scurl bc build-essential libelf-dev 

```
Downgrade gcc to gcc v8 in order to get popcorn to build 


```
sudo apt-get install gcc-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 10
```
Install the popcorn enabled kernel (could take long to compile depending on how performant your machine is) 

```
git clone --depth=1 -b main --single-branch https://github.com/ssrg-vt/popcorn-kernel.git
cd popcorn-kernel
cp /boot/config-...... (closest to 5.2) .config
chmod +x ./update_config.sh
./update_config.sh
```
Edit the .config file using your favourite text editor and make sure that the following options are set as follows:

CONFIG_ARCH_SUPPORTS_POPCORN=yes

```
  make -j12 bindeb-pkg LOCALVERSION=-popcorn (going to ask you some popcorn questions, answer them correctly)
  sudo update-grub
  sudo update-initramfs -u -k all
```
Go up one directory:

```
cd ..
ls (you should see popcorn named .deb files)
sudo apt install ./*.deb (going to create a repo to host the .deb files)
```
Edit grub like 

```
sudo vim /etc/default/grub

```

Make sure the first 3 lines are like 

```
GRUB_DEFAULT=0
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=-1
```

Now reboot your system to boot the popcorn kernel. When rebooting you'll see a "GNU Grub" menu. Select advanced options and select the popcorn named kernel and hit ENTER. Now you have a popcorn enabled kernel in your system.


To test whether popcorn works 


```
sudo mkdir /etc/popcorn
sudo touch /etc/popcorn/nodes
sudo vim /etc/popcorn/nodes

```

In the "nodes" file the IPs should be listed as 

x86's IP address
...... others

....... others


Now 

```
sudo modprobe msg_socket 
```

If you get no error, you're good to go!


For each target arm64 machine (in our case the Raspberry Pi 4Bs):

- Get raspberry pi Linux kernel using the rpi-5.2.y branch and applying the popcorn kernel patch there. (the latest popcorn kernel was based on 5.2.21)
- wrt creating the patch: can first find the base Linux 5.2.21 code commit: https://github.com/ssrg-vt/popcorn-kernel/commit/e91ef5bcdeda8956eb9f1972ed90198b698dca0f
- Then, git diff and git apply to create a patch and patch the RPI-Linux code
- Might need to have me fix errors when apply patch since the raspberry kernel might update the same files as popcorn


```
git clone -b rpi-5.2.y --single-branch https://github.com/raspberrypi/linux

```




Compiler setup:

- Download https://drive.google.com/file/d/1oyFep7igjRaJvl3nx7BRREphcF8Y1dvD/view?usp=drive_open

```
sudo apt-get install build-essential flex bison subversion cmake zip x86_64-linux-gnu-g++
git clone -b main --single-branch https://github.com/ssrg-vt/popcorn-compiler.git
cd popcorn-compiler
ls
sudo mkdir -p /usr/local/popcorn
id 
sudo chown <ID OUTPUTTED BY id command> /usr/local/popcorn
./install_compiler.py --install-all --threads 8
```
The compiler, including supporting libraries and tools, is now installed at <POPCORN PATH>. You can use the the provided Makefile in "popcorn-compiler/util/Makefile.pyalign.template" to build progrms. The makefile template expects that the entire application's source is contained in a single folder.


Command issues you might face:
- Segfault in the Migration Library Due to Executable Stack
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Procedure-calls-in-migration-point


Benchmarking:
- NPB

```
tar -xzf npb......
cd npb....
ls
make A (If you have problems in the linking stage, you probably need to update the Makefile to use the new x86_64-popcorn-linux-gnu-ld.gold instead of ld.gold.)
make
cd ep (or any other)
scp ep_aarch64 ep_x86-64 popcorn@x86
scp ep_aarch64 ep_aarch64 popcorn@armmachinepis
```
On each node

```
cp ep_aarch64 ep 
cp ep_x86-64 ep

```
Now on the main x86  machine do ./ep after chmod +x ./ep (making it executable)

- 

Contact me:

- Email

