
# Create Ubuntu 

## I. Setup Host

### 1. Install tools
	sudo apt-get install binutils debootstrap squashfs-tools xorriso grub-pc-bin grub-efi-amd64-bin mtools schroot 

### 2.	Cấu hình Ubuntu sử dụng debootstrap
	export HOME=/home/phong/
	mkdir $HOME/customize-live-ubuntu-cd
	// cat /etc/os-release bionic
	sudo debootstrap --arch=amd64 --variant=minbase bionic $HOME/customize-live-ubuntu-cd/chroot http://vn.archive.ubuntu.com/ubuntu/
	
	sudo mount --bind /dev $HOME/customize-live-ubuntu-cd/chroot/dev
	sudo mount --bind /run $HOME/customize-live-ubuntu-cd/chroot/run

	   
	- Debootstrap là công cụ được sử dụng để tạo ra hệ thống Debian từ ban đầu mà không yêu cầu dpkg hoặc apt. 
	Debootstrap sẽ thực hiện download bản sao từ upstream và giải nén về local.
	- Sau khi có được bản sao Debian. Chúng ta sẽ thực hiện chạy nó trong môi trường chroot.
	Ta có thể coi chroot như là một Virtual Machine tuy nhiên nó nhẹ hơn và không yêu cầu cài đặt kernel để có thể chạy. 
	Host sẽ chia sẻ nhân kernel với chroot.
	
## II. Basic configuration for chroot
### 1.	Đăng nhập vào chroot
	sudo chroot $HOME/customize-live-ubuntu-cd/chroot
	
	mount none -t proc /proc
	mount none -t sysfs /sys
	mount none -t devpts /dev/pts
	export HOME=/root
	export LC_ALL=C

	The none just means that there is no physical disk partition linked to the mount point you see when issuing the mount command. 
	It is used for virtual filesystems like shm, ramfs, proc and tmpfs.
	
### 2. Update hostname
	echo "thonv12" > /etc/hostname	

### 3. Update apt/source.list
	// update apt source-list
	cat <<EOF > /etc/apt/sources.list
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic main restricted
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic-updates main restricted
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic universe
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic-updates universe
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic multiverse
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic-updates multiverse
	deb http://vn.archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse
	deb http://security.ubuntu.com/ubuntu bionic-security main restricted
	deb http://security.ubuntu.com/ubuntu bionic-security universe
	deb http://security.ubuntu.com/ubuntu bionic-security multiverse
	EOF
	
	// update
	apt-get update
	
### 4. install systemD service

	SystemV is older, and goes all the way back to original Unix.
	SystemD is the new system that many distros are moving to.
	SystemD was designed to provide faster booting, better dependency management, and much more.
	SystemD handles startup processes through .service files.
	SystemV handles startup processes through shell scripts in /etc/init*.
	
	// file /sbin/init
	apt-get install -y libterm-readline-gnu-perl systemd-sysv	

### 5. Install packages necessary needed by system
	apt-get install -y \
	    sudo \
	    ubuntu-standard \
	    casper \
	    lupin-casper \
	    discover \
	    laptop-detect \
	    os-prober \
	    network-manager \
	    resolvconf \
	    net-tools \
	    wireless-tools \
	    wpagui \
	    locales \
	    grub-common \
	    grub-gfxpayload-lists \
	    grub-pc \
	    grub-pc-bin \
	    grub2-common \
	
	apt-get install -y --no-install-recommends linux-generic
	
### 6. Graphic Installer

	apt-get install -y \
	ubiquity \
	ubiquity-casper \
	ubiquity-frontend-gtk \
	ubiquity-slideshow-ubuntu \
	ubiquity-ubuntu-artwork

### 7. Install window manager
	
	apt-get install -y \
    	plymouth-theme-ubuntu-logo \
    	ubuntu-gnome-desktop \
    	ubuntu-gnome-wallpapers
	
### 8. Generate locales
	dpkg-reconfigure locales
 Bấm nút space để chọn
 ![image](https://github.com/VANTHO15/custom_ubuntu/assets/56969447/277e86d0-b925-4957-bfda-2512c8dfa120)
 ![image](https://github.com/VANTHO15/custom_ubuntu/assets/56969447/134327e8-9f9d-4d67-809e-e900cdc1b892)



### 9. Reconfigure resolvconf
	dpkg-reconfigure resolvconf

## III. Starting customize
	
### 1. Install Packages
	apt-get install -y \
		clamav-daemon \
		terminator \
		apt-transport-https \
		software-properties-common \
		wget \
		curl \
		vim \
		nano \
		less \
		ibus-unikey \
  		tree \
    		openssh-server
### 2. Install Applications
	// visual code
	sudo apt update && sudo apt upgrade -y
	wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
	sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
	sudo apt install code
	
	// chrome
	sudo apt-get install libxss1 libappindicator1 libindicator7
	wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
	sudo apt install ./google-chrome*.deb
	
	// gparted
	sudo apt-get install gparted
	
	// beyond compare
	sudo dpkg --add-architecture i386
	sudo apt update
	sudo apt install ./bcompare-4.4.2.26348_amd64.deb
	
	// notepad++
	sudo apt-get install snapd snapd-xdg-open

### 3. Remove unnecessary packages
#### 3.1 Remove apps
	
	sudo apt-get remove package-name
		<Any of the above commands will remove the specified package, but they will leave behind configuration files, 
		and in some cases, other files that were associated with the package.>
	sudo apt-get purge package-name
	
	// remove firefox
	sudo apt-get purge firefox
	
	// remove gnome-games and non-language
	sudo apt-get remove --purge gnome-games*
	sudo apt-get remove --purge `dpkg-query -W --showformat='${Package}\n' | grep language-pack | egrep -v '\-en'`
	
	// show pakages
	dpkg-query -W --showformat='${Package}\n | less
	
	
	
#### 3.2 Remove unused package	
	apt-get autoremove -y
	
	autoremove is used to remove packages that were automatically
	installed to satisfy dependencies for other packages and are now no
	longer needed.
	
### 4. Clean tempotary files and quit
	apt-get clean
	rm -rf /tmp/* ~/.bash_history
	umount /proc
	umount /sys
	umount /dev/pts
	exit	
	
	sudo umount $HOME/customize-live-ubuntu-cd/chroot/dev
	sudo umount $HOME/customize-live-ubuntu-cd/chroot/run

## IV. Create LiveCD

### 1. mout iSO image to customize-live-ubuntu-cd/livecd
	mkdir /tmp/livecd
	sudo mount -o loop ubuntu-18.04.6-desktop-amd64.iso /tmp/livecd 

### 3. Remove rootfs file
	mkdir livecd
	sudo cp -rf /tmp/livecd/* livecd && sync
	sudo rm -rf livecd/casper/filesystem.squashfs

### 4. Compress chroot -> filesystem.squashfs
	sudo mksquashfs chroot livecd/casper/filesystem.squashfs
	
### 5. Copy linux -> livecd/casper/
	sudo cp -rf chroot/boot/vmlinuz-4.15.0-180-generic livecd/casper/vmlinuz
	sudo cp -rf chroot/boot/initrd.img-4.15.0-180-generic livecd/casper/initrd
	
### 6. Recreate manifest file
	sudo chroot chroot/ dpkg-query -W --showformat='${Package} ${Version}\n' | sudo tee livecd/casper/filesystem.manifest
	
### 7. Write the filesystem.size
	printf $(sudo du -sx --block-size=1 chroot | cut -f1) > filesystem.size && sudo cp -f filesystem.size livecd/casper/
	
### 8. Recreate md5 checksum
	find livecd -type f -print0 | xargs -0 md5sum > /tmp/md5sum.txt && cp -f /tmp/md5sum.txt livecd
	
### 9. Create ISO image
	cd livecd && sudo mkisofs -r -V "phonglt15-Ubuntu-Live-Custom" -b isolinux/isolinux.bin -c isolinux/boot.cat -cache-inodes -J -l -no-emul-boot \
	-boot-load-size 4 -boot-info-table -o ../phonglt15-Ubuntu-Live-Custom.iso .
