For this DEMO, we have a physical device for each component of the infrastructure, each one being a Raspberry Pi 4 single board computer.

The devices are connected to each other in a private Ethernet LAN, with IP `192.168.0.1/24`

### Overview

To understand what we want to update, one needs to know what is running on the firmware consumer, which is a raspberry PI 4 in our setup.

The consumer runs a customized linux distro (buildroot) with OPTEE running in TrustZone, you can find more details on this in [[#Firmware consumer]].

The update affects part of the firmware, particularly bootloader stage 3.2 (BL32), which, in the ARM TF-A's boot sequence, is the secure monite (i.e. the TEE: OPTEE)

What follows is a description of our secure update infrastructure, implementing the one described in the corresponding paper.
#### Components

| Component         | IP address  | Short description                                |
| ----------------- | ----------- | ------------------------------------------------ |
| firmware-server   | 192.168.0.2 | Serves firmware images                           |
| status-server     | 192.168.0.3 | Tracks consumer devices and notifies for updates |
| SBOM-verifier     | 192.168.0.4 | Verifies the SBOM of the update                  |
| firmware-consumer | 192.168.0.5 | The IoT device that will install the update      |
##### Hosts file
You can include the following in the `/etc/hosts` file on each device:

```
192.168.0.2 fwserver.com fwserver
192.168.0.3 statusserver.com statusserver
192.168.0.4 sbomverifier.com sbomverifier
192.168.0.5 fwconsumer.com fwconsumer
```
#### Configuration

In order to configure each device to have a static IP, here are the commands to run after a clean installation of rpi-os (change the IP address in the first line).

```bash
sudo nmcli c mod "Wired connection 1" ipv4.addresses 192.168.0.2/24 ipv4.method manual
sudo nmcli con mod "Wired connection 1" ipv4.gateway 192.168.0.1
sudo nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8,8.8.4.4"
sudo nmcli c down "Wired connection 1" && sudo nmcli c up "Wired connection 1"
```

Then, install the common required packages

- git
- python3

```bash
sudo apt install git python3
```

And install any required python package using `apt` (not pip!!)

```bash
sudo apt install python3-[package]
```

#### Firmware Server

Clone the repo:

```bash
git clone https://github.com/crosscon/secure_update_infrastructure
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/firmware-server)

#### Status Server

Clone the repo:

```bash
git clone https://github.com/crosscon/secure_update_infrastructure
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/sbom_server)

#### SBOM Verifier

Clone the repo:

```bash
git clone https://github.com/crosscon/secure_update_infrastructure
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/sbom_server)

#### Firmware consumer

This is the IoT device that will consume the update package.
OPTEE does not officially support the RPI4 platform, but there are instructions on how to port it: https://github.com/Jachm11/optee-os_raspberry_pi_4_port
Follow those instructions to generate an appropriate system that integrates in the testbed's network.

To set a fixed ip, remove `dhcpd` from the Target Packages in the buildroot configuration.
```
Target Packages ==> Networking Applications ==> dhcpcd   --> No
```

If you want to be able to use `scp` (SFTP), disable `dropbear` and enable `openssh`

```
Target Packages ==> Networking Applications ==> dropbear --> No
Target Packages ==> Networking Applications ==> openssh  --> Yes
```

Then, set a filesystem overlay to include some files in the target's root filesystem:

```
System Configuration ==> Root filesystem overlay directories --> ./overlay
```

In the `buildroot/overlay` directory you can place files that will be copied when the rootfs is generated.

- Network interface configuration (`buildroot/overlay/etc/netowork/interfaces`):
```
auto eth0
iface eth0 inet static
address 192.168.0.5
netmask 255.255.255.0
```
- Hosts (DNS) configuration (`buildroot/overlay/etc/hosts`):
```
127.0.0.1	localhost

192.168.0.2 fwserver.com fwserver
192.168.0.3 statusserver.com statusserver
192.168.0.4 sbomverifier.com sbomverifier
192.168.0.5 fwconsumer.com fwconsumer
```

You can also place SSH keys under `buildroot/overlay/root/.ssh` or configure the `authorized_keys`

We will need python and `python-websockets` to run the status-tracker client:

```
Target Packages ==> Interpreter languages and scripting ==> python3 --> Yes
Target Packages ==> Interpreter languages and scripting ==> python3 ==> External python modules ==> python-websockets --> Yes
```

Then, we'll need some libraries for the SUIT updater:

```
Target Packages ==> Libraries ==> Crypto ==> gnutls --> Yes
Target Packages ==> Libraries ==> Networking ==> libcoap --> Yes
Target Packages ==> Libraries ==> Networking ==> libcurl --> Yes
```

##### SUIT-Parser cross-compilation

Buildroot does not support anymore the compilation of a compiler toolchain for the target device, hence you either have to do it yourself or cross-compile everything you need to run on the device.
You can choose the best-suited path, the most elegant would be to add whatever software as a *buildroot package* so that it is automagically compiled during buildroot compilation, and you can contribute to buildroot by extending support!
For cross-compilation, you should have all the necessary ingredients **after building buildroot** (because we need the target's rootfs).

We will cross compile SUIT-Parser:

- clone the repo:
```
git clone https://github.com/crosscon/secure_update_consumer
```

You will now need to edit all the Makefiles in which the following variables are overwritten, with the corresponding value:
- `CC = /<path-to-optee>/OPTEE-RPI4/toolchains/aarch64/bin/aarch64-linux-gnu-gcc`
- `AR = /<path-to-optee>/OPTEE-RPI4/toolchains/aarch64/bin/aarch64-linux-gnu-ar`
- `LD = /<path-to-optee>/OPTEE-RPI4/toolchains/aarch64/bin/aarch64-linux-gnu-ld`
- `CFLAGS = /<path-to-optee>/OPTEE-RPI4/buildroot/output/staging`
- `LDFLAGS = /<path-to-optee>/OPTEE-RPI4/buildroot/output/staging`

After this, you should be able to cross-compile the suit parser with all its dependencies.
You can either copy the binaries with `scp` if the device is live, or include them in the `overlay` folder and rebuild buildroot to generate a rootfs containing them.

# TODO

- completare il programma di update

