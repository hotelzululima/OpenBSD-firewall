# OpenBSD firewall via scripts

## History

Earlier versions of this project were used at Eurospider by 
Mihai Barbos (https://github.com/mbarbos) to build 
corporate-style firewalls with Portwell hardware.

Newer versions run on Soekris hardware now.

I (https://github.com/andreasbaumann/) merely collected 
the ideas and updated them to new versions of OpenBSD
and cleaned up the repository a little bit. :-)

And I'm using it at home.

## Install

Check disk geometry of flash with:

```
disklabel wd0
```

Adapt disk geometry in `hardware/[machine]/flash_params`.

```
Run 'build.sh [machine] [flash_profile]'.
```

Transfer image to flash:

```
dd if=[machine].img of=/dev/wd0c
```

or remotely (after booting from floppy dongle or from hard disk):

```
dd if=[machine].img | ssh [machine] "dd of=/dev/wd1c"
```

## Directory layout

- build.sh: central build script
- doc: various documentation
- template: common files with variables being substituted and then copied to the image
- config: machine-specific configuration (e.g. pf.conf)
- hardware: flash disk geometry for specific machines

## News

15.7.2016:
- updated to OpenBSD 5.9

17.1.2016:
- updated to OpenBSD 5.8
- example shows how to use two nsd's and one unbound to replace a split horizon
  configuration formerly done with bind views

## Roadmap

- upgrade to OpenBSD 6.0
- improve update process, preferably an in-situ update via TFTP
- deal with logging
  - sensord
  - remote syslog
- various playgrounds
  - ospf, pfsync, carp
    
## Other Embedded OpenBSD projects

possible small OpenBSD makers (low level):

- CompactBSD: http://compactbsd.sourceforge.net/, back in 2002,
  looks like OpenBSD 3.x was the last version tested
- Flashboot: http://www.mindrot.org/projects/flashboot/
- Flashrd/Flashdist:
  - http://www.nmedia.net/flashrd/rlsnotes.html
  - https://github.com/yellowman/flashrd/
  - http://www.nmedia.net/~chris/soekris/: original page which has gone,
    flashdist is the older version of flashrd. The EIT
    firewalls where based on early scripts of Chris Cappuccio
    (early flashdist)
- Bowlfish:
  - http://www.kernel-panic.it/software/bowlfish/: latest version 2.1
    seems a little bit old (11.4.2013). The description about Embedded
    OpenBSD is very worthy to read, gives quite some insights how it works.
  - sort of a normal BSD install, not really automatic
  - seems to be for OpenBSD 4.9, not for 5.x
    ./install[332]: /usr/mdec/installboot: not found
    some files in etc missing
- Soekris256:
  - http://256.com/gray/docs/soekris_openbsd_diskless/

more high-level:

- http://opensoekris.sourceforge.net/
- http://compactbsd.sourceforge.net/

others:

- https://andrewmemory.wordpress.com/tag/flashrd/
- http://www.onlamp.com/pub/a/bsd/2004/03/11/Big_Scary_Daemons.html
- http://glozer.net/soekris/cf-install.html
- http://verb.bz/2011/06/12/openbsd-embedded-router/

## Hardware

At Eurospider we had Portwell NAR-2054 (3 and 5 ethernet port versions), some
have VGA ports and USBs, others only COMs, so make sure we always
get boot output on COM.

Now at Eurospider we run it on a Soekris net6501.

At home I'm running it on an ALIX.2D13 with 3 LAN ports and a WLAN card.

## VirtualBox build and test

Create a VMDK wrapper for the disk image built with 'build.sh firewall-test':

```
VBoxManage internalcommands createrawvmdk -filename firewall-test.vmdk -rawdisk firewall-test.image
```

Copy firewall-test.image from OpenBSD machine to the machine running Virtualbox.

Use COM1 and `/tmp/serial`, host pipe, create pipe in VirtualBox, then:

```
socat unix-connect:/tmp/serial stdio,raw,echo=0,icanon=0
```

The network devices is 'em0' not 'reX' on VirtualBox (as opposed to
the real box, at the time of writting there is no Realtek ethernet
card emulated in VirtualBox).

## Troubleshooting

### DMA issues

If you get something like

```
    pciide0:0:0: bus-master DMA error: missing interrupt, status=0x21
```

then change the access mode from DMA to PIO x
See man wd(4) for the values of flags

```
config -e -u -o /bsd.new /bsd

UKC> change wd
change (y/n) ? y
channel [-1] ? -1
flags [0] ? 0xff0
UKC> quit

mv -f /bsd.new /bsd
```

## Links to guides and documentation

- Manpages of OpenBSD.
- http://home.nuug.no/~peter/pf/en/long-firewall.html and his "Book of PF".
- limit handling in production (connection states): 
  http://www.skeptech.org/blog/2013/01/15/pf-limits-in-openbsd/
