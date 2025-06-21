
# Install, Upgrade, and Configure Ubuntu

## Verifying Hardware Compatibility

When Ubuntu is installed, it automatically attempts to detect and configure hardware devices such as network adapters, webcams, audio devices, and more. While this works well in most cases, you may occasionally encounter devices that aren't recognized or don't function correctly.

To identify these devices and begin troubleshooting, you can use some built-in commands to inspect the hardware and look up any unknown components using their device IDs.

---

## Useful Commands for Hardware Detection

### List All USB Devices

To view all USB devices connected to your system, use:

```bash
lsusb -vvv
```

This command provides detailed information about each detected USB device, including vendor and product IDs. These can be useful when searching for Linux-compatible drivers or checking device support.

### List All PCI Devices

To list all internal PCI devices, including graphics cards, Wi-Fi adapters, and sound cards:

```bash
lspci -vvv
```

This command gives verbose output, helping you understand what components Ubuntu has detected and how they are configured.

---

## Verifying Device Functionality

After running the above commands:

1. Look through the output to verify that key devices (e.g., **sound cards**, **network interfaces**, **cameras**) are listed.
2. If a device is missing or not functioning:
   - Copy the **Vendor ID** and **Product ID** from the output.
   - Search online using keywords like `ubuntu [device name or ID] not working`.
   - Visit Ubuntu forums, Stack Overflow, or official hardware documentation for help.
   - Consider using `dmesg` or `journalctl` for deeper system logs.

---

## Tips for Troubleshooting

- You can use the `ubuntu-drivers` command to automatically suggest proprietary drivers:
  ```bash
  sudo ubuntu-drivers devices
  ```

- For audio devices, you can also run:
  ```bash
  aplay -l
  ```
  to list detected playback devices, or:
  ```bash
  arecord -l
  ```
  to list recording devices.

- Use the **"Additional Drivers"** tool from the GUI to install proprietary drivers for NVIDIA, Broadcom, etc.

---

By understanding what hardware your system recognizes, you'll be better equipped to resolve compatibility issues and ensure your Ubuntu installation runs smoothly.
```
