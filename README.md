# APCnotes
Lessons learned for managing APC gear. These are my cliff-notes for handling of APC PDUs, UPSes and ATSes with NMC2 and NMC3.

## Disclaimer
I take no responsibility of any kind for the results of following hints or instructions here. Instructions may be incomplete, incorrect, deceiving and/or malicious.

## Warnings
- The APC console cable is particular to APC. Historically, there have been incidents of APC UPSes shutting down with no warning for using a non-suitable console cable.
- APC UPSes have a firmware (for the UPS-function, not the NMC) which, depending on the model of UPS, may shut down outputs when installed.
- APC ATSes have a particular procedure to follow for installing firmware. Disregarding this may damage your ATS

## Useful CLI commands
```
eventlog (events logged locally)
about (will often show which NMC is installed)
upsabout (for UPSes, duh)
alarmList
```

## Firmware
We differentiate between the primary function (UPS, PDU, ATS)   and    the NMC (Network Management Card) of your APC device.

UPS and ATS (and PDU?) have firmwares for their primary function, which is distinct from the NMC firmware, and must be downloaded and installed independently.
Nevertheless, NMC firmware may require specific primary function firmware to work correctly.

APC has different generations of NMCs: 
- NMC  (gen1, End of Life)
- NMC2 (gen2, End of Life)
- NMC3 (gen3)
- NMC4 (gen4).

Each NMC generation may have multiple model numbers.  
- AP9630, AP9631 and AP9635 (gen2, not an exhaustive list)
- AP9640, AP9641 and AP9643 (gen3, not an exhaustive list)

Some APC units have NMC as a module or plug-in card, others have it embedded.

Tow different device types (say, a UPS and a PDU) may have the same NMC, but they will need different, non-interchangeable firmware packages for their NMC, even if they have the same NMC model or generation.

Two instances of the same UPS, PDU or ATS **may** have been delivered with different NMC generations.

Different NMC generations have different, incompatible firmware versions:

### NMC2
- 5.x.y, 6.x.y, 7.x.y.   You want to have the latest available 7.x.y version.
- typically 3 files: aos, application and bootmon
- installation with bundled windows app, or with scp or via web UI.

### NMC3 
- 1.x.y.z, 2.x.y.z, 3.x.y.z. Your unit might have been delivered with 3.x.y.z. Updates to version 3 require a service agreement.
- single file upload (*.nmc3)
- installation with bundled windows app, or with scp or via web UI.



## Configuration
Most configuration and relevant operations for NMC can be done via CLI, but some exceptions exist.

SYSLOG cannot be configured from CLI.
Firewall can be enabled/disabled, but configuration requires using the web UI or uploading a specifically designed text file via SCP.
