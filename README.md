## tty0tty - Linux Null Modem Emulator

Originally forked from: https://github.com/freemed/tty0tty

## 1. tty0tty directory tree:

```bash
    module  :linux kernel module null-modem
    pts	    :null-modem using ptys (without handshake lines)
```

#### pts(unix98): 

```bash
    # When run connect two pseudo-ttys and show the connection names:
    (/dev/pts/1) <=> (/dev/pts/2) 
```

  
```bash
    # The connection is:
    TX -> RX
    RX <- TX 	
```

#### module:

The module is tested in kernel 4.19.0-10-amd64 (debian). When loaded, it creates 10 (5 pairs) ttys:

```bash
    /dev/tnt0  <=>  /dev/tnt1 
    /dev/tnt2  <=>  /dev/tnt3 
    /dev/tnt4  <=>  /dev/tnt5 
    /dev/tnt6  <=>  /dev/tnt7 
    /dev/tnt8  <=>  /dev/tnt9 
```

The connection is:

```bash
    TX   ->  RX
    RX   <-  TX 	
    RTS  ->  CTS
    CTS  <-  RTS
    DSR  <-  DTR
    CD   <-  DTR
    DTR  ->  DSR
    DTR  ->  CD
```
  
## 2. Build Requirements:

kernel-headers or kernel source is reqired, run following command to install:

```bash
    # to search using $ apt-cache search linux-headers
    $ sudo apt-get install linux-headers-$(uname -r)
```

Then go to ./pts and ./module separately to run `make` to build.

## 3. Compile/Build


```bash
    $ cd pts:
    $ make        # to compile 
    $ ./tty0tty   # to run 	
```

```bash
    $ cd modules
    $ make        	        # to compile 
    $ insmod tty0tty.ko     # to load module (using root or sudo)	
```

## 4. Install

### 4.1 Copy the new kernel module into the kernel directory

```bash
    # mkdir ./misc if it does not exist under /lib/modules/4.19.0-10-amd64/kernel/
    $ sudo cp tty0tty.ko /lib/modules/$(uname -r)/kernel/misc/  # sudo cp tty0tty.ko /lib/modules/4.19.0-10-amd64/kernel/misc/
```

### 4.2 Load the module

```bash
    sudo depmod
    sudo modprobe tty0tty
```

Now you should see serial ports in `/dev/tnt[0-9]` and give permissions to the new serial ports:

```bash
    $ sudo chmod 666 /dev/tnt*
```

You can now access the serial ports. Note that the consecutive ports are interconnected. For example,
`/dev/tnt0` and `/dev/tnt1` are connected as if using a direct cable.

### 4.3 Persisting across boot:

```bash
    $ echo "tty0tty" >> /etc/modules # namely, adding line tty0tty to the end of the file
```

### 4.4 Add user to the dialout group to have permissions on the tty devices

```bash
    $ sudo usermod -aG dialout $USER
```
```bashr
    user@debian:~$ ls -l /dev/tnt*
    crw-rw---- 1 root dialout 245, 0 Oct 20 00:51 /dev/tnt0
    crw-rw---- 1 root dialout 245, 1 Oct 20 00:51 /dev/tnt1
    crw-rw---- 1 root dialout 245, 2 Oct 20 00:51 /dev/tnt2
    crw-rw---- 1 root dialout 245, 3 Oct  7 03:42 /dev/tnt3
    crw-rw---- 1 root dialout 245, 4 Oct  7 03:42 /dev/tnt4
    crw-rw---- 1 root dialout 245, 5 Oct  7 03:42 /dev/tnt5
    crw-rw---- 1 root dialout 245, 6 Oct 20 00:51 /dev/tnt6
    crw-rw---- 1 root dialout 245, 7 Oct 20 00:51 /dev/tnt7
    crw-rw---- 1 root dialout 245, 8 Oct 20 01:06 /dev/tnt8
    crw-rw---- 1 root dialout 245, 9 Oct 20 01:06 /dev/tnt9
```

If /dev/tnt* are within `root` group, use `sudo chgrp dialout /dev/tnt*` to change them to `dialout` group, and then add username to dialout group with above command.

## Note

For any reason you want to build `dkms` package, run

```bash
$ sudo apt-get update && sudo apt-get install -y dh-make dkms build-essential debuild -uc -us
```
