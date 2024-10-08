# RPiZero2W-Pwnagotchi

The goal of this project was to set up a headless Pwnagotchi using a Raspberry Pi Zero 2 W, aiming for a more stealthy and portable build. This documentation is not intended as a guide, but rather to detail the challenges encountered during the setup and the methods used to overcome them.

## Bill of Materials

- **Raspberry Pi Zero 2 W:** [Microcenter](https://www.microcenter.com/product/643085/raspberry-pi-zero-2-w)
- **16GB+ UHS1 or Better Micro SD Card:** [Amazon](https://www.amazon.com/Professional-verified-Amazon-MicroSDHC-Certified/dp/B07BYSZP73)
- **Pwnagotchi Build Compatible with RPi Zero 2 W:** [GitHub Repo](https://github.com/jayofelony/pwnagotchi)
- **Custom Pwnagotchi Plugins:** [Custom Plugins Repo](https://github.com/SHUR1K-N/Project-Pwnag0dchi)



![pwnagotchi](https://github.com/AdamHayball/RPiZero2W-Pwnagotchi/blob/main/pwnyboi.png)



## Initial Setup

The process began by flashing the Pwnagotchi image to a micro SD card using Balena Etcher. With the image flashed, the next step was configuring the system to fit the intended use case.

### Configuring the Pwnagotchi

The configuration file, located at `/etc/pwnagotchi/config.toml`, was set up with the following parameters:
- Custom device name
- Whitelist for trusted networks
- Enabling Android Bluetooth tethering

The initial setup seemed straightforward until connectivity issues arose.

## Encountering USB Ethernet (RNDIS Gadget) Issues

The first challenge surfaced when attempting to connect the Pi using RNDIS Gadget for USB Ethernet on a Windows 11 machine. Instead of appearing as a COM or Network device, the Pi showed up in Device Manager as an unknown serial USB device with an invalid PID/VID.

### Troubleshooting Steps

- Confirmed that the `dtoverlay` and necessary modules were correctly configured in `/boot/config.txt` and `/boot/cmdline.txt`.
- Attempted to manually install the RNDIS driver from the Windows driver catalog, but the process was hindered by driver signing issues.
- Even in Safe Mode, forcing the application of the driver didn’t resolve the issue.

At this point, something was clearly wrong beyond the usual driver issues. To isolate the problem, I switched to Bluetooth tethering as an alternative.

## Bluetooth Tethering Challenges

Bluetooth tethering posed its own set of problems: the Pi would connect to my Android phone, only to disconnect immediately. Since Bluetooth was essential for displaying data on the headless unit, resolving this was crucial.

### Steps Taken

- Enabled tethering on the phone and attempted auto connection by editing `/etc/pwnagotchi/config.toml` via terminal, this did not work.
- Attempted to manually pair and trust the phone with the Pi:
  ~~~
  bluetoothctl
  scan on
  pair <device_mac_address>
  trust <device_mac_address>
  connect <device_mac_address>
  ~~~
Despite these steps, no BNEP (Bluetooth Network Encapsulation Protocol) adapter was showing up when using `ifconfig`. It appeared that the network setup was not being completed correctly.

A review of the phone’s tether settings revealed that Bluetooth tethering had turned off for some unknown reason. After re-enabling it and redoing the manual pairing and trusting process, the BNEP adapter finally appeared.

## Resolving USB Connectivity Issues

To further troubleshoot, I tested the Pi on another Windows machine that already had RNDIS drivers installed. This time, I used a pogo pin style USB adapter instead of the standard USB cable, which instantly resolved the driver issue—suggesting that the USB cable was faulty or the port itself was bad.

Returning to the fresh Windows 11 machine, I repeated the process with the correct cable, and the Pi was recognized correctly as a network device. I then configured the network parameters on the Windows machines ethernet adapter, utilizing the host internet sharing script and successfully accessed the Pwnagotchi’s web GUI. With both USB and Bluetooth tethering functional, the next goal was to make the setup portable.

## Achieving Portable Access

The portability requirement meant that accessing the Pwnagotchi via a phone was essential. Initially, the Bluetooth connection to the phone was unstable. After verifying and adjusting the network adapters on the Pi, I confirmed that the connection was working as expected:
  ~~~
  ifconfig -a
  ~~~
With the BNEP adapter visible and using the configured IP address, I disconnected the Ethernet over USB, rebooted using the power only micro-usb port on the Pi, and confirmed that the web service was accessible via Bluetooth tethering on my phone.

## Adding Custom Features and Plugins

Having established basic functionality, the next step was to customize the Pwnagotchi with additional features like association and deauthentication attacks. This involved accessing the Pi locally over USB and using SFTP to transfer files:

  # Using WinSCP to connect via SFTP
  # Navigate to the appropriate directories and upload custom plugins

Initially, access to certain directories over SFTP was restricted. Attempts to log in as root revealed incorrect passwords. After resetting the passwords and making necessary configuration changes, the new plugins were successfully added, and the `config.toml` file was updated:
  ~~~
  nano /etc/pwnagotchi/config.toml
  ~~~
The Pwnagotchi rebooted with the new configuration, but the device name was incorrect and displaying `Pwnag0dchi` now, changing the `config.toml` would no longer update the name being displayed. Verifying and updating the hostname resolved this issue:
  ~~~
  hostnamectl set-hostname <new_hostname>
  ~~~
Updates were installed, but a DNS resolution error occurred. This was identified as a local nameserver misconfiguration. After fixing the nameserver settings, the Pwnagotchi operated correctly:
  ~~~
  nano /etc/resolv.conf
  ~~~
## Final Testing and Backup

With the Pwnagotchi fully functional, it was tested to ensure it could capture WiFi handshakes and provide access via the web GUI. The `.pcap` files could be downloaded directly from the web interface, and plugins enabled online hash cracking using `wpa-sec` to perform this task automatically when internet is available to the pwnagotchi.

As a final step, a backup of the setup was created using Win32 Disk Imager:

  # Using Win32 Disk Imager to create a backup image
  - The GUI for Win32 Disk Imager is self explanatory but unintuitive and will provide you with a full `.img` disk image roughly the size of the micro SD cards capacity.

This completed the setup, providing a fully functional and portable stealth Pwnagotchi system. Power is provided via a battery bank and the whole setup fits conveniently in a pocket or backpack.

