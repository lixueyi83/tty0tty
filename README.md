## tty0tty - linux null modem emulator

Originally forked from: https://github.com/freemed/tty0tty

### tty0tty directory tree:

- module    :linux kernel module null-modem
- pts	    :null-modem using ptys (without handshake lines)


#### pts(unix98): 

When run connect two pseudo-ttys and show the connection names:

(/dev/pts/1) <=> (/dev/pts/2) 

the connection is:
  
- TX -> RX
- RX <- TX 	

#### module:

The module is tested in kernel 4.19.0-10-amd64 (debian). When loaded, it creates 10 ttys interconnected (5 pairs):

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
  
### Compile Requirements:

kernel-headers or kernel source is reqired, run following command to install:

```bash
    # to search using $ apt-cache search linux-headers
    $ sudo apt-get install linux-headers-$(uname -r)
```

- then go to ./pts and ./module separately to run `make` to build

### Compile/Build

- cd pts:
    - make        :to compile 
    - ./tty0tty   :to run 	

- cd module:
    - make        	    :to compile 
    - insmod tty0tty.ko :to load module (using root or sudo)	

### Install

- Copy the new kernel module into the kernel directory

```bash
    # mkdir ./misc if it does not exist
    $ sudo cp tty0tty.ko /lib/modules/$(uname -r)/drivers/misc/
```

- Load the module

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

- Persisting across boot:

```bash
    $ echo "tty0tty" >> /etc/modules # namely, adding line tty0tty to the end of the file
```

- Add user to the dialout group to have permissions on the tty devices

```bash
    $ sudo usermod -a -G dialout $USER
```

### Note

For any reason you want to build `dkms` package, run

```bash
    sudo apt-get update && sudo apt-get install -y dh-make dkms build-essential debuild -uc -us
```
