# HI3518E_camera

## Ports
The ports that are open on this device are:
```
PORT      STATE SERVICE
80/tcp    open  http
554/tcp   open  rtsp
9527/tcp  open  H264DVR server v1.0
9530/tcp  open  unknown
34567/tcp open  DIVR-IP Web server
```
The `H264DVR` server can be accessed using telnet and it gives access to a text console where commands can be inspected with a help function.

### Web port (80)
Command for taking picture snapshots:
```
http://192.168.2.46/webcapture.jpg?command=snap&channel=0
```

## Firmware
Firmware is contained in the `General_HZXM_IPC_HI3518E_50H10L_S38_V4.02.R12.20140926_ALL.bin` file that is basically a zip archive with the following files:
* Install
* InstallDesc
* custom-x.cramfs.img
* romfs-x.cramfs.img
* u-boot.bin.img
* user-x.cramfs.img
* web-x.cramfs.img

## Inspecting images
After trying to read a first image file, `web-x.cramfs.img` I found a problem because it seems not an CRAMFS image as specifid in the file name. In fact trying to mount it as `cramfs` I got a `cramfs: wrong magic` error.
In reality by using the following `file` command:
I saw the following output:
```
file web-x.cramfs.img                                                                                                                    
```
I got this output:
```
web-x.cramfs.img: u-boot legacy uImage, linux, Linux/ARM, Standalone Program (gzip), 1286144 bytes, Fri Sep 26 07:09:51 2014, Load Address: 0x00630000, Entry Point: 0x00770000, Header CRC: 0xAD796646, Data CRC: 0xB77010E9
```
So it mention really a `u-boot legacy uImage`... So after searching I found a [useful article] (https://boundarydevices.com/hacking-ram-disks/) about the fact that `mkimage (the command used to create the image) add a 64 bytes header (U-boot header) in front of the image containing the CRC value.
So, using the following `dd` command I stripped off the first 64 bytes:
```
 dd bs=1 skip=64 if=web-x.cramfs.img of=web-x.img 
 ```
 Now issuing the same `file` command:
 ```
 file web-x.img
 ```
 I found this interesting output:
```
web2.img: Squashfs filesystem, little endian, version 4.0, 1282652 bytes, 139 inodes, blocksize: 262144 bytes, created: Fri Sep 26 07:09:51 2014
```
As it turns out the image is really a `squashfs`. Then I tried to mount it as `squashfs` file system:
```
mkdir web-x
sudo mount -o looop -t squashfs web-x.img web-x
```
And here is the content of the root dir in the mount point `web-x`:
```
-rwxr-xr-x 1      556      556  510 Sep 26  2014 1.jpg
-rwxr-xr-x 1      556      556  525 Sep 26  2014 11.jpg
-rwxr-xr-x 1      556      556  536 Sep 26  2014 16.jpg
-rwxr-xr-x 1      556      556  531 Sep 26  2014 161.jpg
-rwxr-xr-x 1      556      556  547 Sep 26  2014 25.jpg
-rwxr-xr-x 1      556      556  552 Sep 26  2014 251.jpg
-rwxr-xr-x 1      556      556  560 Sep 26  2014 36.jpg
-rwxr-xr-x 1      556      556  573 Sep 26  2014 361.jpg
-rwxr-xr-x 1      556      556  549 Sep 26  2014 4.jpg
-rwxr-xr-x 1      556      556  561 Sep 26  2014 41.jpg
-rwxr-xr-x 1      556      556  577 Sep 26  2014 64.jpg
-rwxr-xr-x 1      556      556  577 Sep 26  2014 641.jpg
-rwxr-xr-x 1      556      556  557 Sep 26  2014 9.jpg
-rwxr-xr-x 1      556      556  561 Sep 26  2014 91.jpg
-rwxr-xr-x 1      556      556  26K Sep 26  2014 DVR.htm
-rwxr-xr-x 1      556      556  851 Sep 26  2014 English.js
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 ForbitPlay.gif
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 ForbitRecord.gif
-rwxr-xr-x 1      556      556  644 Sep 26  2014 ForbitSubPlay.gif
-rwxr-xr-x 1      556      556  12K Sep 26  2014 Login.htm
-rwxr-xr-x 1      556      556  909 Sep 26  2014 SimpChinese.js
-rwxr-xr-x 1      556      556  956 Sep 26  2014 Talk.gif
-rwxr-xr-x 1      556      556  492 Sep 26  2014 addPreSet.jpg
-rwxr-xr-x 1      556      556  486 Sep 26  2014 addPreSet1.jpg
-rwxr-xr-x 1      556      556  490 Sep 26  2014 audio.jpg
-rwxr-xr-x 1      556      556  544 Sep 26  2014 audio1.jpg
-rwxr-xr-x 1      556      556 2.9K Sep 26  2014 back.GIF
-rwxr-xr-x 1      556      556  393 Sep 26  2014 bg.jpg
-rwxr-xr-x 1      556      556 1.5K Sep 26  2014 board.gif
-rwxr-xr-x 1      556      556  603 Sep 26  2014 bt.gif
-rwxr-xr-x 1      556      556  817 Sep 26  2014 config.js
-rwxr-xr-x 1      556      556  465 Sep 26  2014 delPreSet.jpg
-rwxr-xr-x 1      556      556  463 Sep 26  2014 delPreSet1.jpg
-rwxr-xr-x 1      556      556  323 Sep 26  2014 dlr.jpg
-rwxr-xr-x 1      556      556  586 Sep 26  2014 dm.jpg
-rwxr-xr-x 1      556      556  520 Sep 26  2014 editCruise.jpg
-rwxr-xr-x 1      556      556  497 Sep 26  2014 editCruise1.jpg
-rwxr-xr-x 1      556      556 1.5K Sep 26  2014 err.htm
-rwxr-xr-x 1      556      556 1.3K Sep 26  2014 err.jpg
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 failed.htm
-rwxr-xr-x 1      556      556  688 Sep 26  2014 full.jpg
-rwxr-xr-x 1      556      556  571 Sep 26  2014 full1.jpg
-rwxr-xr-x 1      556      556  466 Sep 26  2014 goCruise.jpg
-rwxr-xr-x 1      556      556  467 Sep 26  2014 goCruise1.jpg
-rwxr-xr-x 1      556      556  498 Sep 26  2014 goPreSet.jpg
-rwxr-xr-x 1      556      556  497 Sep 26  2014 goPreSet1.jpg
-rwxr-xr-x 1      556      556  86K Sep 26  2014 index.htm
-rwxr-xr-x 1      556      556 4.9K Sep 26  2014 l_bgm.gif
-rwxr-xr-x 1      556      556 4.1K Sep 26  2014 l_bgmd.gif
-rwxr-xr-x 1      556      556  366 Sep 26  2014 labg.jpg
-rwxr-xr-x 1      556      556  593 Sep 26  2014 lbt.jpg
-rwxr-xr-x 1      556      556  689 Sep 26  2014 logOut.gif
-rwxr-xr-x 1      556      556  905 Sep 26  2014 logo.gif
-rwxr-xr-x 1      556      556  348 Sep 26  2014 lr.jpg
-rwxr-xr-x 1      556      556  13K Sep 26  2014 m.css
-rwxr-xr-x 1      556      556  31K Sep 26  2014 m.jsp
-rwxr-xr-x 1      556      556  600 Sep 26  2014 m_dral.gif
-rwxr-xr-x 1      556      556  496 Sep 26  2014 m_dral.jpg
-rwxr-xr-x 1      556      556  150 Sep 26  2014 m_dram.gif
-rwxr-xr-x 1      556      556  364 Sep 26  2014 m_dram.jpg
-rwxr-xr-x 1      556      556  592 Sep 26  2014 m_drar.gif
-rwxr-xr-x 1      556      556  496 Sep 26  2014 m_drar.jpg
-rwxr-xr-x 1      556      556  319 Sep 26  2014 m_inTop.jpg
-rwxr-xr-x 1      556      556 1.2K Sep 26  2014 m_top.jpg
-rwxr-xr-x 1      556      556  411 Sep 26  2014 mb_bg.jpg
-rwxr-xr-x 1      556      556  386 Sep 26  2014 mb_bg2.jpg
-rwxr-xr-x 1      556      556  287 Sep 26  2014 mc.jpg
-rwxr-xr-x 1      556      556 2.2K Sep 26  2014 ml.jpg
-rwxr-xr-x 1      556      556 2.7K Sep 26  2014 mr.jpg
-rwxr-xr-x 1      556      556  26K Sep 26  2014 mt.js
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 noPlay.gif
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 noRecord.gif
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 noSubPlay.gif
-rwxr-xr-x 1      556      556  952 Sep 26  2014 noTalk.gif
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 play.gif
-rwxr-xr-x 1      556      556  306 Sep 26  2014 plcb11.jpg
-rwxr-xr-x 1      556      556  394 Sep 26  2014 plcbl.jpg
-rwxr-xr-x 1      556      556  394 Sep 26  2014 plcbr.jpg
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 record.gif
-rwxr-xr-x 1      556      556  814 Sep 26  2014 recordAll.jpg
-rwxr-xr-x 1      556      556  824 Sep 26  2014 recordAll1.jpg
-rwxr-xr-x 1      556      556  926 Sep 26  2014 sal.gif
-rwxr-xr-x 1      556      556  927 Sep 26  2014 sal1.gif
-rwxr-xr-x 1      556      556  925 Sep 26  2014 sar.gif
-rwxr-xr-x 1      556      556  929 Sep 26  2014 sar1.gif
-rwxr-xr-x 1      556      556  845 Sep 26  2014 sas.gif
-rwxr-xr-x 1      556      556  925 Sep 26  2014 sks.gif
-rwxr-xr-x 1      556      556  752 Sep 26  2014 snap.jpg
-rwxr-xr-x 1      556      556  772 Sep 26  2014 snap1.jpg
-rwxr-xr-x 1      556      556  796 Sep 26  2014 startAll.jpg
-rwxr-xr-x 1      556      556  808 Sep 26  2014 startAll1.jpg
-rwxr-xr-x 1      556      556  595 Sep 26  2014 stopAll.jpg
-rwxr-xr-x 1      556      556  601 Sep 26  2014 stopAll1.jpg
-rwxr-xr-x 1      556      556  441 Sep 26  2014 stopCruise.jpg
-rwxr-xr-x 1      556      556  449 Sep 26  2014 stopCruise1.jpg
-rwxr-xr-x 1      556      556  594 Sep 26  2014 stopRecordAll.jpg
-rwxr-xr-x 1      556      556  590 Sep 26  2014 stopRecordAll1.jpg
-rwxr-xr-x 1      556      556 1.1K Sep 26  2014 subPlay.gif
-rwxr-xr-x 1      556      556  336 Sep 26  2014 t.jpg
-rwxr-xr-x 1      556      556  436 Sep 26  2014 t1t.jpg
-rwxr-xr-x 1      556      556  481 Sep 26  2014 t2t.jpg
-rwxr-xr-x 1      556      556  851 Sep 26  2014 top.jpg
-rwxr-xr-x 1      556      556  452 Sep 26  2014 tx1.jpg
-rwxr-xr-x 1      556      556  418 Sep 26  2014 tx2.jpg
-rwxr-xr-x 1      556      556  448 Sep 26  2014 tx3.jpg
-rwxr-xr-x 1      556      556  655 Sep 26  2014 tx4.jpg
-rwxr-xr-x 1      556      556 1.2M Sep 26  2014 web.cab
-rwxr-xr-x 1      556      556 1.2K Sep 26  2014 yt+.gif
-rwxr-xr-x 1      556      556 1.2K Sep 26  2014 yt+1.gif
-rwxr-xr-x 1      556      556 1.2K Sep 26  2014 yt-.gif
-rwxr-xr-x 1      556      556 1.2K Sep 26  2014 yt-1.gif
-rwxr-xr-x 1      556      556  519 Sep 26  2014 yt1.jpg
-rwxr-xr-x 1      556      556  539 Sep 26  2014 yt11.jpg
-rwxr-xr-x 1      556      556  574 Sep 26  2014 yt2.jpg
-rwxr-xr-x 1      556      556  587 Sep 26  2014 yt21.jpg
-rwxr-xr-x 1      556      556  519 Sep 26  2014 yt3.jpg
-rwxr-xr-x 1      556      556  530 Sep 26  2014 yt31.jpg
-rwxr-xr-x 1      556      556  610 Sep 26  2014 yt4.jpg
-rwxr-xr-x 1      556      556  620 Sep 26  2014 yt41.jpg
-rwxr-xr-x 1      556      556  734 Sep 26  2014 yt5.jpg
-rwxr-xr-x 1      556      556  713 Sep 26  2014 yt51.jpg
-rwxr-xr-x 1      556      556  481 Sep 26  2014 yt51a.jpg
-rwxr-xr-x 1      556      556  484 Sep 26  2014 yt51b.jpg
-rwxr-xr-x 1      556      556  483 Sep 26  2014 yt5a.jpg
-rwxr-xr-x 1      556      556  509 Sep 26  2014 yt5b.jpg
-rwxr-xr-x 1      556      556  601 Sep 26  2014 yt6.jpg
-rwxr-xr-x 1      556      556  606 Sep 26  2014 yt61.jpg
-rwxr-xr-x 1      556      556  494 Sep 26  2014 yt7.jpg
-rwxr-xr-x 1      556      556  501 Sep 26  2014 yt71.jpg
-rwxr-xr-x 1      556      556  539 Sep 26  2014 yt8.jpg
-rwxr-xr-x 1      556      556  544 Sep 26  2014 yt81.jpg
-rwxr-xr-x 1      556      556  488 Sep 26  2014 yt9.jpg
-rwxr-xr-x 1      556      556  492 Sep 26  2014 yt91.jpg
-rwxr-xr-x 1      556      556  387 Sep 26  2014 yta1.jpg
-rwxr-xr-x 1      556      556  423 Sep 26  2014 yta11.jpg
-rwxr-xr-x 1      556      556  330 Sep 26  2014 ytabg.jpg
-rwxr-xr-x 1      556      556  484 Sep 26  2014 yy1.jpg
-rwxr-xr-x 1      556      556  528 Sep 26  2014 yy11.jpg
```

