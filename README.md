Something about RAID! 
=================

Doc is mostly baised towards **HARDWARE RAID**.

#### Which RAID are you using?

There are many ways to check that, But you are in luck at the moment, I am going to explain it as simple as I can so,


Software RAID
-------------------

- Go to you sever's terminal (linux) and type `cat /proc/mdstat`
- Here you can have multiple type of results but, we are going to check for 2 of them

1:
```
.Personalities : [raid1] [raid6] [raid5] [raid4]
md3 : active raid6 sda4[0] sdo4[14] sdn4[13] sdm4[12] sdl4[11] sdk4[10] sdj4[9] sdi4[8] sdh4[7] sdg4[6] sdf4[5] sde4[4] sdd4[3] sdc4[2] sdb4[1]
      287086592 blocks super 1.2 level 6, 512k chunk, algorithm 2 [15/15] [UUUUUUUUUUUUUUU]

md1 : active raid1 sda2[0]
      523712 blocks super 1.2 [15/1] [U______________]

md0 : active raid1 sda1[0] sdo1[14] sdn1[13] sdm1[12] sdl1[11] sdk1[10] sdj1[9] sdi1[8] sdh1[7] sdg1[6] sdf1[5] sde1[4] sdd1[3] sdc1[2] sdb1[1]
      4190208 blocks super 1.2 [15/15] [UUUUUUUUUUUUUUU]

unused devices: <none>
```
2:
```
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
unused devices: <none>
```

> **Note:**
> Besides you can simply run df command and if you see md devices it means you have software RAID.
> command => `df -h`, You will see the md devices in partition table.

In case of 1:, You have a software RAID, that's it **CASE CLOSED**.

In case of 2:, You have a HARDWARE RAID, how to confirm and go through this? Lets do it, together..


HARDWARE RAID
--------------

- Go to you turminal an type `lspci | grep -i raid`
> There are many vendors in case of HARDWARE RAID such as
```
[root@sorage3 ~]# lspci | grep -i raid
06:00.0 RAID bus controller: Hewlett-Packard Company Smart Array Controller (rev 04)
[root@storage3 ~]#
[root@storage4 ~]# lspci | grep -i raid
01:00.0 RAID bus controller: Adaptec AAC-RAID (rev 09)
[root@storage4 ~]#
[root@storage5 ~]# lspci | grep -i raid
01:00.0 RAID bus controller: LSI Logic / Symbios Logic MegaRAID SAS 1078 (rev 04)
[root@storage5 ~]#
```

Installing MegaCLI
------------------

- We are going to discuss about MegaRAID SAS anyhow, first of all you need, MegaCLI to run few MegaRAID commands to get the information you need,
1: go to your terminal and run `nano megacli.sh`.
2: Paste the code below in your file,

```
#!/usr/bin/env sh

### Download and install megaraidcli for Ubuntu;
 
FILE="megacli_8.07.14.orig.tar.gz"
LINK="http://hwraid.le-vert.net/ubuntu/sources/$FILE"
wget $LINK -O /tmp/$FILE
 
(
	cd /tmp
	tar -zxvf $FILE && cd megacli-*/opt
	cp -R MegaRAID/ /opt
)

apt-get install libsysfs-dev libsysfs2 sysfsutils
ln -s /lib/libsysfs.so.2.0.1 /usr/lib/libsysfs.so.2.0.2


exit 0
```
3: run this command in your terminal, `chmod +x megacli.sh`, this will make the script executable.
4: run this command as `./megacli`, It will install and setup MegaCLI for you in this `/opt/MegaRAID/MegaCli/` path.

CHECK RAID LEVEL
----------------

First of all, there is a little explanation for RAID levels as
```
['Primary-0, Secondary-0, RAID Level Qualifier-0'] = RAID-0
['Primary-1, Secondary-0, RAID Level Qualifier-0'] = RAID-1
['Primary-5, Secondary-0, RAID Level Qualifier-3'] = RAID-5
['Primary-6, Secondary-0, RAID Level Qualifier-3'] = RAID-6
['Primary-1, Secondary-3, RAID Level Qualifier-0'] = RAID-10
```

Now its your turn to check your RAID level.
1: Go to your terminal and run this command, with your MegaCLI path as `/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -L0 -a0`, You should see some infomation as below

```
Adapter 0 -- Virtual Drive Information:
Virtual Drive: 0 (Target Id: 0)
Name                :
RAID Level          : Primary-6, Secondary-0, RAID Level Qualifier-3
Size                : 70.947 TB
Sector Size         : 512
Is VD emulated      : Yes
Parity Size         : 10.915 TB
State               : Optimal
Strip Size          : 256 KB
Number Of Drives    : 15
Span Depth          : 1
Default Cache Policy: WriteBack, ReadAhead, Direct, Write Cache OK if Bad BBU
Current Cache Policy: WriteBack, ReadAhead, Direct, Write Cache OK if Bad BBU
Default Access Policy: Read/Write
Current Access Policy: Read/Write
Disk Cache Policy   : Disk's Default
Encryption Type     : None
Bad Blocks Exist: No
Is VD Cached: No

Exit Code: 0x00
```

As in the above result, You can see RAID LEVEL as
```
RAID Level          : Primary-6, Secondary-0, RAID Level Qualifier-3
```
and from the above explanation of RAID levels, it seems you have RAID Level 6, In any case if you want to see just that above infomattion, then run this command `/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL | grep RAID`, you will see results as

```
RAID Level          : Primary-6, Secondary-0, RAID Level Qualifier-3
```
So this is RAID LEVEL 6.


CHECK DISK FAILURES && ETC
-------------------

1:

As you have megacli installed already, Now you can have many benefits from that, with one liners as

Run this command to get your disks state and errors

```
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aAll | egrep "Enclosure Device ID:|Slot Number:|Inquiry Data:|Error Count:|state"
```
This will result into
```
Enclosure Device ID: 245
Slot Number: 0
Media Error Count: 0
Other Error Count: 0
Firmware state: Online, Spun Up
Inquiry Data: NCGNJYHS            HGST HUS726060ALE610                    APGNT7J0
...../...../....
Enclosure Device ID: 245
Slot Number: 14
Media Error Count: 0
Other Error Count: 0
Firmware state: Online, Spun Up
Inquiry Data: NCGG42NS            HGST HUS726060ALE610                    APGNT7J0

```
The firm state can tell you, if your that specific slot is working or not, or defected.

2:

run this command `/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aALL` , it will result into a big result, where you can have many many useful information about you RAID but the most important to me or for you is

```
                Device Present
                ================
Virtual Drives    : 1 
  Degraded        : 0 
  Offline         : 0 
Physical Devices  : 16 
  Disks           : 15 
  Critical Disks  : 0 
  Failed Disks    : 0 

```

3:

run this command, if you only need to see the count for failed disks `/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aALL | grep "Failed"` , this will results into,

```
Security Key Failed              : No
  Failed Disks    : 0 
Deny Force Failed                       : No
```


Disk FAILED > 0
===============

The status of each drive can be checked using this command via SSH root:

1.

```
/opt/MegaRAID/MegaCli/MegaCli64 -ShowSummary -aALL
```

The output will be similar to:
```
Connector : ##########<Internal><Encl Pos 1 >: Slot 2
Vendor Id : ##########
Product Id : ##########
State : Unconfigured Good
Disk Type : SAS,Hard Disk Device 
Capacity : 3.637 TB 
Power State : Active 
```

2.
You need to get the correct enclosure position and slot of the drive by using the following command:

```
opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL
```

Output will be similar to:

```
Adapter #0 
[...]
Enclosure Device ID: 8
Slot Number: 2
[...]
Firmware state: Unconfigured(good), Spun Up
```

3.
To get the right Array and Row position of the drive in the RAID array, execute:

```
/opt/MegaRAID/MegaCli/MegaCli64 -pdgetmissing -aALL
```

The output will be similar to:

```
Adapter 0 - Missing Physical drives 
No. Array Row Size Expected 
0 0 3 3814697 MB
```
4.

Place the missing drive into the RAID array and run this command:

```
/opt/MegaRAID/MegaCli/MegaCli64 -PdReplaceMissing -PhysDrv [8:2] -Array0 -row3 -a0
```

Where 8 and 2 in [8:2] are the Enclosure Device ID and slot number respectively, 0 in Array0 is the array number, 3 in row3 is the number of the drive in the RAID array, and 0 in a0 represents the adapter.

Output should be similar to:

```
Adapter: 0: Missing PD at Array 0, Row 3 is replaced.
```

5.

To start rebuilding the drive run this command:

```
/opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -Start -PhysDrv [8:2] -a0
```

Output should be similar to this:

```
started rebuild progress on device encl 8 slot 2
```

6.

To check status on the rebuilding status, run this command:

```
/opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -ShowProg -PhysDrv [8:2] -a0
```

the output of the above command should be similar to this.

```
Rebuild Progress on Device at Enclosure 8, Slot 2 Completed 0% in x Minutes.
```

NOTE: Rebuild in RAID6 is going to take time, for refernce please, see.
 - http://www.computerweekly.com/feature/RAID-rebuilds-How-do-RAID-rebuilds-work-and-which-is-fastest
 - https://community.spiceworks.com/topic/585368-raid-6-with-2-failed-drives-one-is-rebuilding


Cautions and Checks
===================

1.

If you have 2 drives failed in case of RAID6, then its equal to one failure, also never replace 2 disk together but one at a time, for reference please see https://serverfault.com/questions/306526/raid-6-better-to-replace-two-dead-drives-at-the-same-time-or-one-at-a-time


----------------------

So this was it for now, I am going to share some useful links down below for read and apply, And I will also add more and more commands and information to this doc, In anycase, If you are reading this and founding anything wrong, or want to support the doc by adding infotmation or correction, please make a PR.

Useful links
-----------
- https://hwraid.le-vert.net/wiki/LSIMegaRAIDSAS
