# Graphical OS Tutorial

This tutorial will guide you through building a simple graphical OS using Linux. We will cover the installation of dependencies, kernel setup, creating a basic GUI, and running the OS using QEMU.

## 1. Install Dependencies

We will need several dependencies to set up the Linux kernel and tools for building the OS. Run the following command to install all required packages:

```bash
sudo apt install wget bzip2 libncurses-dev flex bison bc libelf-dev libssl-dev xz-utils autoconf gcc make libtool git vim libpng-dev libfreetype-dev g++ extlinux nano
```bash
2. Set Up Linux Kernel
Create a new directory for your Linux kernel, download the kernel source, and configure it:

bash
Kopiuj
mkdir linux && cd linux
sudo apt update
sudo apt install wget bzip2 libncurses-dev flex bison bc libelf-dev libssl-dev xz-utils autoconf gcc make libtool git vim libpng-dev libfreetype-dev g++ extlinux nano
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.9.4.tar.xz
tar xf linux-6.9.4.tar.xz
cd linux-6.9.4
make menuconfig
2.1 Kernel Configuration
In the menuconfig interface, configure the kernel with the following settings:

Device Drivers > Graphics Support > Cirrus [*]
Device Drivers > Graphics Support > DRM Driver for VMware Virtual GPU [*]
Device Drivers > Graphics Support > Laptop Hybrid Graphics [*]
Device Drivers > Graphics Support > Intel Xe Graphics [M]
Device Drivers > Graphics Support > Virtual Box Graphics Card [*]
(Deprecated) Device Drivers > Graphics Support > Simple framebuffer driver [*]
Device Drivers > Graphics Support > Frame Buffer Devices > Support for frame buffer device drivers [*]
Device Drivers > Graphics Support > Bootup logo [*]
Device Drivers > Generic Input Layer (mousedev) > Mouse interface [*]
Once you're done, save and exit.

3. Build the Kernel
Now, let's build the kernel. This may take around 20-40 minutes depending on your system.

bash
Kopiuj
make -j $(nproc)
4. Set Up Kernel Directory
Create a directory for your distribution, and copy the kernel image to it:

bash
Kopiuj
sudo mkdir /distro
cd ~/linux/linux-6.9.4/arch/x86_64/boot
sudo cp bzImage /distro/bzImage
cd /distro
5. Set Up Busybox
Busybox is needed to provide basic command-line utilities in your OS. Install it by following these steps:

bash
Kopiuj
sudo wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
sudo tar xf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
sudo make menuconfig
5.1 Busybox Configuration
In the menu, go to Settings and select Build static binary [*].
Under Networking Utilities, uncheck tc.
6. Build Busybox
Build Busybox, which should take around 1-2 minutes:

bash
Kopiuj
sudo make -j $(nproc)
sudo make CONFIG_PREFIX=/distro install
7. Set Up NanoX
NanoX is required to run graphical applications. Without it, the OS will not compile. Follow these steps to set it up:

bash
Kopiuj
cd /distro
sudo git clone https://github.com/ghaerr/microwindows.git
cd microwindows/src
sudo cp Configs/config.linux-fb config
sudo nano config
7.1 Update Configuration
Search for NX11 and change its value from N to Y. This ensures that X11 headers are correctly set up.

After editing, exit the file, and then:

bash
Kopiuj
sudo nano nx11/Makefile
Find X11_INCLUDE and uncomment x11-local, while commenting x11hdrlocation. Then exit.

8. Build NanoX
Now, you can build NanoX with the following command:

bash
Kopiuj
sudo make
9. Copy Important Libraries and Create Sample App
Next, copy the necessary libraries for NanoX to work and create a simple GUI app.

bash
Kopiuj
sudo make install
sudo mkdir /x11-demo
cd /x11-demo
sudo nano gui.c
9.1 Create a Simple GUI App
Paste the following code into gui.c:

c
Kopiuj
#include <X11/Xlib.h>
int main() {
	XEvent event;
	Display* display = XOpenDisplay(NULL);
	Window w = XCreateSimpleWindow(display,DefaultRootWindow(display),50,50,250,250,1,BlackPixel(display,0),WhitePixel(display,0));
	XMapWindow(display,w);
	XSelectInput(display,w,ExposureMask);
	for (;;) {
		XNextEvent(display,&event);
		if (event.type == Expose) {
			XDrawString(display,w,DefaultGC(display,0),100,100,"Thanks for reading!",20);
		}
	}
	return 0;
}
9.2 Compile the GUI App
Compile the application:

bash
Kopiuj
sudo gcc gui.c -lNX11 -lnano-X -I /distro/microwindows/src/nx11/X11-local/
9.3 Copy Executable
Once compiled, copy the executable:

bash
Kopiuj
sudo cp a.out /distro/a.out
9.4 Copy Libraries
Now, copy the necessary libraries to make sure NanoX works correctly:

bash
Kopiuj
cd /distro/microwindows
cd src/bin
sudo mkdir -p /distro/lib/x86_64-linux-gnu
sudo mkdir -p /distro/lib64/
sudo cp /lib/x86_64-linux-gnu/libpng16.so.16 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libz.so.1 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libfreetype.so.6 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libm.so.6 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libbz2.so.1.0 /distro/lib/x86_64-linux-gnu/
sudo cp /lib/x86_64-linux-gnu/libbrotlidec.so.1 /distro/lib/x86_64-linux-gnu/
sudo cp /lib64/ld-linux-x86-64.so.2 /distro/lib64/
sudo cp /lib/x86_64-linux-gnu/libbrotlicommon.so.1 /distro/lib/x86_64-linux-gnu/
9.5 Copy Other Binaries
bash
Kopiuj
cd /distro
cd /distro/microwindows/src
sudo cp -r bin /distro/nanox
sudo cp runapp /distro/nanox
10. Create boot.img
Now let's create the boot.img with a size of 200MB.

bash
Kopiuj
cd /distro
sudo truncate -s 200MB boot.img
sudo mkfs boot.img
10.1 Mount boot.img
Create a mount point and mount the boot.img:

bash
Kopiuj
sudo mkdir /distro/mnt
sudo mount boot.img mnt
10.2 Install extlinux
Install the bootloader:

bash
Kopiuj
sudo apt install extlinux
cd /distro
sudo extlinux -i mnt
10.3 Copy Files to mnt
Copy the necessary files to mnt:

bash
Kopiuj
cd /distro
sudo cp -a bin bzImage lib lib64 linuxrc nanox/ a.out sbin/ usr/ mnt
cd mnt
sudo mkdir var etc root tmp dev proc
cd ..
sudo umount mnt
11. Run the OS!
Now that everything is set up, you can run the OS using QEMU. First, install QEMU:

bash
Kopiuj
sudo apt install qemu-system
Then, run the OS:

bash
Kopiuj
cd /distro
sudo qemu-system-x86_64 boot.img -vga cirrus
11.1 Configuration for Booting
When prompted, type:

bash
Kopiuj
/bzImage rw root=/dev/sda
To prevent this prompt in the future, mount mnt again and create an extlinux.conf file with the following contents:

bash
Kopiuj
TIMEOUT 50
DEFAULT linux

LABEL linux
    MENU LABEL Linux
    LINUX /bzImage  # Path to your bzImage in root directory
    APPEND root=/dev/sda rw
Now you're ready to boot your graphical OS! 🎉

