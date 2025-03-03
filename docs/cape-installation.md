# Cape installation

The procedure followed to install CAPEv2.

[Bibliografia](#bibliografía)

## 1. Introduction

Different architectures can be use for the Sandbox.

- Option 1: Install CAPE on VM, and then CAPE will have KVM to then create its own VMs
- Option 2: Install CAPE on host (it has to be a Linux OS) and then CAPE will have KVM to then create its own VMs.

I chose option 2, as I have Ubuntu already install directly on the computer. Therefore, I will have:
1. Host OS: Ubuntu 22.04 LTS as CAPEv2 Host
2. Hypervisor: QEMU/KVM (The one recommended by CAPEv2)
3. Virtual Machine: Windows 10 as CAPEv2 victim


## 2. Prepare Requirements

The computer I am using for this is: Dell-Precision-5560

1. Install Ubuntu 22.04 LTS
   1. Username: `cape` (this will make installation simpler)
   2. Hostname: `cape`

2. Update software with the Software Updater app or use in terminal:

```bash
sudo apt update
sudo apt upgrade
```

3. Install software used later: `git`, VSCode, `ifconfig`.
```bash
sudo apt install git -y
sudo snap install code --classic
sudo apt install net-tools -y
```


> [!info]
>
> Although Ubuntu 24.04 LTS is the last LTS version, I used Ubuntu 22.04 LTS because:
> - as I had previous fail attemps installing CAPEv2 on Ubuntu 24.
> - I found more documentation on installing CAPEv2 on Ubuntu 22.



## 3. Clone CAPEv2 repository

Clone [CAPEv2 oficial repository](https://github.com/kevoreilly/CAPEv2.git):

```bash
git clone https://github.com/kevoreilly/CAPEv2.git
```

And access the instalation folder:

```bash
cd CAPEv2/installer
ls -al
```


## 4. KVM installer

The KVM installer is the `kvm-qemu.sh` file. The KVM used in CAPEv2 is a special KVM that has been designed to be able to overcome the existence of evasion techniques in malware, especially virtualization evasion or sandbox evasion techniques.

### Modifications required in the KVM installer 

Before executing the installer some modifications on the file are required, a code string that must be defined in the KVM installer. This code string refers to the **DSDT** code sourced from the ACPI environment information used. The modification made is to replace the `'<WOOT>'` string on `./kvm-qemu,sh` according to the DSDT code string obtained. (The DSDT code string rule that is input in the form of 4 character letters, however this is not my case).

The code string can be obtained in several ways:

1. Option 1: Using `acpidump`. This tool is used to dump ACPI information on the device used. The results of the dump will be extracted to obtain DSDT code strings that can be used as input to the KVM installer.

```bash
sudo apt install acpica-tools -y
sudo acpidump > acpidump.out
sudo acpixtract -a acpidump.out
sudo iasl -d dsdt.dat
```

The DSDT code appear in the line starting with `ACPI:`, between brackets with other information. In my case, the DSDT code was `Dell Inc`

2. Option 2: Using the "Hardware ID" string. It is suppose to apper in the output of the following command. In my case, it was not clear.

```bash
cat dsdt.dsl | grep “Hardware ID”
```

3. Option 3 (recommended one if possible): Using DSDT code strings sourced from opensource. The following repository contains tabiñar mapping information containing The required information is on the DSDT row and "Oem TableId".
   1. [Repository's Dell-Precission-5560 file](https://raw.githubusercontent.com/linuxhw/ACPI/refs/heads/master/Notebook/Dell/Precision/Precision%205560/75602B246865).
   2. In my case: `"Dell Inc"`

In my case, Dell-Precission-5560 DSDT code is `"Dell Inc"`, so we need to change the string `"<WOOT>"` for `"Dell Inc"`. Like this:

```bash
# what to use as a replacement for QEMU in the tablet info
PEN_REPLACER='Dell Inc'
# what to use as a replacement for QEMU in the scsi disk info
SCSI_REPLACER='Dell Inc'
# what to use as a replacement for QEMU in the atapi disk info
ATAPI_REPLACER='Dell Inc'
# what to use as a replacement for QEMU in the microdrive info
MICRODRIVE_REPLACER='Dell Inc'
# what to use as a replacement for QEMU in bochs in drive info
BOCHS_BLOCK_REPLACER='Dell Inc'BOCHS_BLOCK_REPLACER2='Dell Inc'BOCHS_BLOCK_REPLACER3='Dell Inc'
# what to use as a replacement for BXPC in bochs in ACPI info
BXPC_REPLACER='Dell Inc'
```


### Executing the KVM installer

After configuring the KVM installer script, we need to **execute the installer**:

1. Give execution permission
```bash
chmod a+x kvm-qemu.sh
```

2. Consult the usage
```bash
./kvm-qemu.sh -h
```

3. Execute kvm-qemu installer:
```bash
sudo ./kvm-qemu.sh all cape 2>&1 | tee kvm-qemu.log
```

4. Install Virtual Machine Manager (`virt-manager`)
```bash
sudo ./kvm-qemu.sh virtmanager cape 2>&1 | tee kvm-qemu-virtmanager.log
```

5. **Reboot**: After the installation is complete, it is necessary to reboot the host system so that the system can make adjustments after the KVM/QEMU installation is carried out.


One of the adjustment form is that there is a new network interface, namely `virbr0`, which is a network interface used as a network interface virtualization in the KVM/QEMU hypervisor system. The network interface has a default IP address, which is important for the next step. Check it out with:

```bash
ifconfig
```

In my case the network interface name is `virbr0`, with IP address `192.168.122.1`

### Check if everything works correctly

Execute virt-manager through terminal and check out there are no warnings or errors shown.

```bash
virt-manager
```

If there are some errors, do not continue with the installation.


## 5. Execute CAPEv2 installer (Sandbox System)


## 6. Ensuring the postgres database is configured

The postgres database has been configured to have a database with the name "cape" and owner "cape".

Execute postgres with:

```bash
sudo -u postgres psql
```

In postgress execute:
```bash
\l
```

## 7. Install the optional features

```bash
cd /opt/CAPEv2
```

Poetry should be installed with the previous steps in this folder, so you can use it correctly with:

```bash
poetry install
poetry env list
poetry run pip3 install -r extra/optional_dependencies.txt 
```

Check out if the services are running:
```bash
sudo systemctl status cape.service
sudo systemctl status cape-processor.service
sudo systemctl status cape-router.service
sudo systemctl status cape-web.service
```


```bash
sudo systemctl restart cape*
```


## Bibliografía

Fuentes principales:
- [Documentación oficial CAPE](https://capev2.readthedocs.io/en/latest/index.html)
- [Documentación oficial instalación CAPE](https://capev2.readthedocs.io/en/latest/installation/index.html)

Otras fuentes:
- [Medium CAPEv2 on Ubuntu (Part 1)](https://medium.com/@rizqisetyokus/building-capev2-automated-malware-analysis-sandbox-part-1-da2a6ff69cdb)
- [Medium CAPEv2 on Ubuntu (Part 2)](https://medium.com/@rizqisetyokus/building-capev2-automated-malware-analysis-sandbox-part-2-0c47e4b5cbcd)
- [Medium CAPEv2 on Ubuntu (Part 3)](https://medium.com/@rizqisetyokus/building-capev2-automated-malware-analysis-sandbox-part-3-d5535a0ab6f6)
