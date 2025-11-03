For this DEMO, we have a physical device for each component of the infrastructure, each one being a Raspberry Pi 4 single board computer.

The devices are connected to each other in a private Ethernet LAN, with IP `192.168.0.1/24`

#### Components

| Component         | IP address  | Short description                           |
| ----------------- | ----------- | ------------------------------------------- |
| firmware-server   | 192.168.0.2 | Serves firmware images                      |
| status-server     | 192.168.0.3 |                                             |
| SBOM-verifier     | 192.168.0.4 |                                             |
| firmware-consumer | 192.168.0.5 | The IoT device that will install the update |

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
git clone https://github.com/crosscon/secure_update_infrastructure.
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/firmware-server)

#### Status Server

Clone the repo:

```bash
git clone https://github.com/crosscon/secure_update_infrastructure.
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/sbom_server)

#### SBOM Verifier

Clone the repo:

```bash
git clone https://github.com/crosscon/secure_update_infrastructure.
```

Then follow the instructions in its [README](https://github.com/crosscon/secure_update_infrastructure/tree/main/sbom_server)

# TODO

- capire come buildare OPTEE-OS
- capire come si buildano/caricano le TAs
- completare il programma di update
- scegliere TA da aggiornare -> avere due versioni
- IP statico in OPTEE-OS