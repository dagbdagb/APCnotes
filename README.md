# APCnotes
Lessons learned for managing APC gear. These are my cliff-notes for handling of APC PDUs, UPSes and ATSes with NMC2 and NMC3. Hopefully, it acts as a shortcut for someone handed to task of bringing their APC stuff up to date.

## Disclaimer
I take no responsibility of any kind for the results of following instructions or trusting information found here. Instructions/information found on the Internet may be incomplete, incorrect, deceiving and/or malicious. Trust, but verify.

## !! Warnings !!
- The APC console cable is particular to APC. Historically, there have been incidents of APC UPSes immediately shutting down with no warning for using a non-suitable console cable.
- APC UPSes have a firmware (for the UPS-function, not the NMC) which:
    - is specific for the UPS model in question
    - depending on UPS model, **may toggle outputs** when installed
    - is not part of the NMC firmware package and must be downloaded separately
- APC ATSes have a particular procedure to follow for installing firmware. **Disregarding this may damage your ATS.**

## Useful CLI commands
```
eventlog     (events logged locally)
about        (will often show which NMC is installed as well as the current aos/app/bootmon versions)
upsabout     (for UPSes, duh)
detbat -all  (detailed UPS battery information)
alarmList    (shows active alarms)
reboot       (will restart NMC only, does not affect outputs)
```

## Firmware, NMC-generations, device-types.
We differentiate between the primary function (UPS, PDU, ATS)   and    the NMC (Network Management Card) of your APC device.

UPS and ATS (and PDU?) have firmwares for their primary function, which is distinct from the NMC firmware, and must be downloaded and installed independently.
Nevertheless, NMC firmware may require specific primary function firmware to work correctly.

APC has different generations of NMCs: 
- NMC  (gen1, End of Life)
- NMC2 (gen2, End of Life)
- NMC3 (gen3)
- NMC4 (gen4).

Each NMC generation may have multiple models/modelnumbers.  
- AP9630, AP9631, AP9635, OG-1238, AP9537SUM (gen2, not an exhaustive list)
- AP9640, AP9641, AP9643, ON-1570 (gen3, not an exhaustive list)

Some APC devices have the NMC as a module or plug-in card, others have it embedded.

Two different device types (say, a UPS and a PDU) may have the same NMC, but they will need different, non-interchangeable firmware packages for their NMC, even if they have the same NMC model or generation.

Two instances of the same device model (say, two UPSes of the same model) **may** have been delivered with different NMC generations.

Different NMC generations have different, incompatible firmware versions:

- 5.x.y, 6.x.y, 7.x.y.  NMC2. You want to have the latest available 7.x.y version.
  - typically 3 files: aos, $application and bootmon. bootmon may already be at the right version. 'about' in CLI will tell you what you already have.
  - installation with bundled windows app, or with scp or via web UI.

- 1.x.y.z, 2.x.y.z, 3.x.y.z.  NMC3. Your unit might have been delivered with 3.x.y.z. Updates to version 3 require a service agreement.
  - single file upload (*.nmc3)
  - installation with bundled windows app, or with scp or via web UI.

Figuring out the exact generation of your NMC may require some googling.


## Configuration
Most configuration and relevant operations for NMC can be done via CLI, but some exceptions exist.

- SYSLOG cannot be configured from CLI.
- Firewall can be enabled/disabled, but configuration requires using the web UI or uploading a specifically designed text file via SCP.


## SSH / SCP

If sshd does not respond, check if it responds to telnet. enabling ssh disables telnet (and vice versa?).
If neither works, enable ssh via http/https.
If http/https does not work, check section SSL / https below.
If everything fails, check relevant firewalls. If even that fails to explain anything, get an APC  console cable.

The sshd on NMC2 does not support public key authentication.
Modern ssh-clients may have to use a few extra flags:
- fw 6.x.x:
  ```
  ssh -c 3des-cbc -oHostKeyAlgorithms=+ssh-rsa  apc@target
  scp -O -c 3des-cbc -oHostKeyAlgorithms=+ssh-rsa sourcefile apc@target:targetfile
  ```
- fw 7.x.x:
  ```
  ssh -c aes128-ctr -oHostKeyAlgorithms=+ssh-rsa  apc@target
  scp -O -c aes128-ctr -oHostKeyAlgorithms=+ssh-rsa sourcefile apc@target:targetfile
  ```
If you use DNS, these extra flags may be encoded into your ```.ssh/config```. Do note that you must provide a target *filename*, not just the directory.


## SSL / https

The onboard SSL-certificate may have expired. If so, the onboard webserver refuses to start http*s*. And the client will report nothing useful. The fix is to delete the onboard cert and restart the NMC.
```
ssh target
cd ssl
dir
delete defaultcert.p15
reboot
```

That said, the NMC2 is end of life. It may be safer to just disable the webserver altogether:
```
web -h disable -s disable
```

## Firmware installation
Do not jump major versions when installing firmware. 
- 5 -> 6 -> 7.
- 1 -> 2 -> 3

Firmware can be installed via xmodem if something goes seriously wrong. See below.
Otherwise, the CLI way to upgrade NMC2 firmware is to first scp in the aos-file, allow the device to reboot, and then scp in the application file after which the NMC reboots again.
Sorry, I do not have notes for the bootmon file.

For NMC3, you just scp in the .nmc3 file.


## Firmware recovery

- Use cable part# 940-0144A
- 115200N81
- reset, hit the <enter> key a few times until you see the 'BM>'-prompt
- format flash and start xmodem:
```
  BM> format
WARNING: Formatting will erase all data.
Are you sure (Y/N)? Y

Formatting...
complete.
BM> help
?
help
boot
reset
sysinfo
mfginfo
xmodem
format

BM> xmodem
```

How to start transmitting the image depends on your terminal program. With Opengear ACM7xxx, the process is as follows:
- disconnect from pmshell with '~~~.' or '~~.' or '~.'. So 3, 2 or 1 times tilde, followed by a period.
- transmitting the file requires you to have copied the file to the ACM prior, Then do:
```lsz -X /var/mnt/storage.nvlog/apc_hw21_rpdu2g_2-5-2-5.nmc > /dev/port08 < /dev/port08```

Adapt the port number in the previous command as needed.



## Useful links/URLs
- ATS upgrade procedure - https://community.se.com/t5/EcoStruxure-IT-forum/Do-firmware-updates-require-reboots/td-p/210618#.Ye_6SJyVHLo.link
- CLI guide - https://www.apc.com/us/en/download/document/SPD_CCON-AYCELJ_EN/
- Determine NMC version 
  - https://www.se.com/sa/en/faqs/FA285026/
  - https://www.apc.com/us/en/faqs/FA228791/
  - https://community.se.com/t5/APC-UPS-Data-Center-Enterprise/How-to-determine-NMC-version-via-software/td-p/475968
- UPS firmware - https://www.apc.com/us/en/faqs/FAQ000242942/


## Sample bash script to create APC NMC config commands
- Save the script below as mkconfig.sh
- Modify parameters in header as needed
- Make it executable.
- Run it with the 5 parameters at the top of the header as options.
Like this: ```./mkconfig.sh whatever-ups 192.168.2.3 255.255.255.0 192.168.2.1 Wherever```

The output can be pasted straight into the NMC CLI.


```
#!/bin/sh
# script for generating CLI commands for configuring APC NMCs
# Copyright Dag Bakke (2025)

name=$1
ip=$2
mask=$3
gw=$4
loc=$5

domain="mydomain.com"
contact="noc@mydomain.com"
dns1="192.168.1.1"
dns2="192.168.1.2"
ntp="192.168.1.1"
traptargetip1="192.168.1.1"
community1="communityX"
community2="communityY"
traptargetip1="192.168.1.1"
traptargetip2="192.168.1.2"
snmppollerip1="192.168.1.1"
snmppollerip2="192.168.1.2"
authsecret1="whatever"
authsecret2="alsowhatever"
privacypw1="russia_is_a_terrorist_state"
privacypw2="slava_ukraini"

userapcpw="rellybadidea"


echo tcpip -S enable -i $ip -s $mask -g $gw -d $ domain -h $name
echo tcpip6 -S disable

echo dns -p $dns1 -s $dns2 -d $domain -h $name

echo system -n $name -c $contact -l $loc -s enable

echo ntp -p $ntp1 -s $ntp2 -e enable -u

echo snmptrap -r1 $traptargetip1 -t1 snmpV1 -a1 enable -u1 $community1 -g1 enable
echo snmptrap -r2 $traptargetip2 -t2 snmpV1 -a2 enable -u2 $community2 -g2 enable

echo snmpv3 -S enable -u1 authpriv -a1 $authsecret1 -c1 $privacypw1 -ap1 sha -pp1 aes -ac1 enable -au1 authpriv -n1 $snmppollerip1
echo snmpv3 -S enable -u2 authpriv -a2 $authsecret2 -c2 $privacypw2 -ap2 sha -pp2 aes -ac2 enable -au2 authpriv -n2 $snmppollerip2
echo ftp -S disable
echo web -h disable -s enable -mp TLS1.2 -lsp disable -lsd disable -cs 4
user -n apc -pw $userapcpw -cp apc
```
