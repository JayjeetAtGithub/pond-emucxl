### emucxl setup instructions

`emucxl` uses the pond techniques for cxl emulation. This README provides instructions for setting up `emucxl` on a 
regular 2-socket (2S) system.

#### Prerequisites
- Ubuntu 22.04
- Qemu Latest

#### Steps
1. Check the ubuntu and kernel version
    ```bash
    $ hostnamectl
    $ uname -r
    ```
2. Check the number of numa nodes. If the system has more than one numa node, then the system is a multi-socket system.
    ```bash
    $ numactl -H
    ```
3. check the qemu version and install the latest version if not installed
    ```bash
    # Qemu installation from the source code
    ## Install the required dependencies. More details can be found at https://wiki.qemu.org/Hosts/Linux
    $ sudo apt-get install -y libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev python3-venv ninja-build
    $ sudo apt-get install libibverbs-dev libjpeg8-dev libncurses5-dev libnuma-dev libaio-dev libslirp-dev libaio1 flex bison
    # Now, download the latest version and build the qemu
    $ wget https://download.qemu.org/qemu-8.2.2.tar.xz
    $ tar xvJf qemu-8.2.2.tar.xz
    $ cd qemu-8.2.2
    $ mkdir build && cd build
    $ ../configure --prefix=/opt/qemu-8.2.2 # Install the qemu in /opt/qemu-8.2.2 to avoid conflicts with the system installed qemu
    $ make -j all
    $ make install
    ```
4. configure host networking

    a. confirm IP forwarding is enabled on the host (0: disabled, 1: enabled)
    ```bash
    $ cat /proc/sys/net/ipv4/ip_forward
    $ sudo cat /proc/sys/net/ipv6/conf/default/forwarding
    # If the output is 0, then enable the IP forwarding temporarily by running the following command
    $ sudo echo 1 > /proc/sys/net/ipv4/ip_forward
    $ sudo echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
    ```
5. download an ubuntu 22.04 server image
    ```bash
    $ mkdir -p ~/images/
    $ cd ~/images
    $ wget https://releases.ubuntu.com/22.04/ubuntu-22.04.4-live-server-amd64.iso
    # create the qcow2 file that will serve as the hard drive for the VM:
    $ qemu-img create -f qcow2 emucxl.qcow2 40G
    $ bash ./create_vm.sh
    ```
6. After the VM setup is done, edit the /etc/default/grub file and update the grub `sudo update-grub`.
    ```bash
    GRUB_CMDLINE_LINUX="ip=dhcp console=ttyS0,115200 console=tty console=ttyS0"
    GRUB_TERMINAL=serial
    GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1"
    ```
    edit the /boot/grub/menu.lst file with the following content
    ```bash
    serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1
    terminal --timeout=15 serial console
    ```
7. Now, shut down the emucxlVM 
    ```bash
    $ sudo shutdown -h now
    ```
8. Start the emucxlVM
    ```bash
    $ bash ./start_vm.sh
    ```
    If you correctly setup the aforementioned configurations, you should be able to see text-based VM login in the same terminal where you issue the running script. You can play around the start_vm.sh script to change remote and local memory configurations.
9. After login into the VM, confirm the cxl setup by running the following command
    ```bash
    $ sudo apt-get install numactl
    $ sudo numactl --hardware
    ```
    It should show the Node 0 and Node 1 with the respective memory and CPU details.
