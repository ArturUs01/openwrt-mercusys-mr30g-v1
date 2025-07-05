# **GUIDE FOR INSTALLING OPENWRT ON MERCUSYS MR30G V1**

<details>
  <summary>🌐 Language / Выбрать язык</summary>

  - 🇺🇸 [English](README.md)  
  - [Русский](README_RU.md)

</details>

![OpenWrt logo](include/logo.png)

> \[!WARNING]
> **Proceed at your own risk.** This process involves opening the router, replacing the SPI flash chip, and flashing a custom U-Boot bootloader. Mistakes may permanently damage your device or render it inoperable (bricked).

> \[!CAUTION]
> **You must dump the original firmware before making any modifications.** The calibration data for the Wi-Fi radios is located in a partition at the end of the factory flash. Without this data, Wi-Fi may not function properly or at all. **You will need this dump later.** Also, make sure to save your ISP login details (e.g., PPPoE username and password) from the router’s web interface.

> \[!IMPORTANT]
> **Firmware and bootloader based on TP-Link Archer C5 v4.** The OpenWrt firmware, its config files (DTS, DTSI, LED, and network), and the custom U-Boot bootloader for MR30G v1 were made using the source code and configurations from the TP-Link Archer C5 v4 (GPL sources). U-Boot modifications include correct GPIO setup for LEDs and 16 MB SPI flash support.

---

## **STEPS FOR INSTALLATION**

> \[!IMPORTANT]
> **You’ll need:**
>
> * An SPI flash programmer (e.g., CH341A)
> * A new 16 MB flash chip (e.g., Winbond W25Q128FVSG — recommended and tested)
>
> The factory flash chip has only 4 MB of memory, which is not enough for OpenWrt. Replacing it is essential for proper firmware support and system expansion.

### **1. Hardware Mod: Replacing the Flash Chip**

1. **Disassemble the router:**

   * First, take a photo of the sticker on the bottom of the router (model, MAC, default Wi-Fi password).
   * Then remove the sticker carefully — it covers a screw slightly above and to the right of center.
   * Unscrew it and pry off the top cover using a plastic card or guitar pick. Go around the edges gently to unclip everything.
   * Be careful not to insert the tool too deep to avoid damaging internal parts or antenna cables.

2. **Locate the factory flash chip:**

   * You’ll find a small 8-pin chip left of the white heatsink. It will be labeled something like `cFeon QH32B-104HIP` — this is the original flash chip.

3. **Remove and dump the flash:**

   * Carefully desolder the chip and solder it to the adapter for your programmer.
   * Connect the programmer to your PC and dump the chip to a file named `dump.bin`.
   * Verify the dump to ensure it's complete and not corrupted.
   * Save this file somewhere safe on your disk — you will absolutely need it later.

> \[!IMPORTANT]
> **TIP:** Newer versions of the official software for programmers like CH341A are known to be unstable. Consider using NeoProgrammer (Windows) or `flashrom` (Linux).

4. **Prepare the new flash chip:**

   * Remove the factory chip from the programmer adapter and store it — it won’t be used again.
   * Solder a new 16 MB flash chip (such as any compatible model, e.g., Winbond W25Q128FVSG) to the programmer’s adapter.
   * Erase and confirm it is fully blank using your programming software.

### **2. Editing and Writing the OpenWrt Dump**

We will use a prepared firmware image file with a name like `dump_openwrt_04-07-25.bin`, which you can download in the **Releases** section. This file includes:

* A custom U-Boot bootloader modified for this router.
* The latest clean OpenWrt firmware (at the time of release).
* Properly aligned partition layout.
* Placeholders for inserting your device’s MAC address and radio calibration data.

1. Open both your `dump.bin` (original dump) and `dump_openwrt_xx-xx-xx.bin` in a hex editor (e.g., HxD for Windows or any hex editor for Linux).

2. **Insert MAC address:**

   * In `dump.bin`, go to address `003FE000`.
   * You should see your MAC address there (it should look approximately like: `C0 25 XX XX XX XX`).
   * Copy those 6 bytes.
   * In `dump_openwrt_xx-xx-xx.bin`, go to the beginning of address `00FDF100` and paste your MAC address with overwrite, replacing the bytes `00 00 00 00 00 00`.

3. **Insert radio calibration data:**

   * **2.4 GHz radio:**

     * In `dump.bin`, go to address `003FF000` (for reference, the hex code should begin with `20 76 05 01`).
     * Copy the range from `003FF000` to `003FF300`.
     * In `dump_openwrt_xx-xx-xx.bin`, go to the beginning of address `00FF0000` and paste with overwrite.

   * **5 GHz radio:**

     * In `dump.bin`, go to address `003FF800` (for reference, the hex code should begin with `63 76 00 01`).
     * Copy the range from `003FF800` to `003FFE00`.
     * In `dump_openwrt_xx-xx-xx.bin`, go to the beginning of address `00FF8000` and paste with overwrite.

> \[!CAUTION]
> Do **not** modify the `dump.bin` file in any way! Only read from it.

4. Save the edited `dump_openwrt_xx-xx-xx.bin`. Ensure all changes were saved properly.

5. Flash the edited file to the new chip using the programmer. Make sure to verify the write operation afterward to confirm that the file was written correctly and is not corrupted.

6. Desolder the new chip from the adapter and solder it back onto the router PCB. Be sure to align the notch/key correctly.

---

## **3. First Boot and Verification**

1. Power on the router. If the power LED lights up, it’s alive!
2. Wait — the first boot may take up to 2 minutes.
3. Plug an Ethernet cable from your computer into the router and go to [http://192.168.1.1](http://192.168.1.1) in browser.
4. The OpenWrt web interface will appear. No password is set yet, just click **Login**.

> \[!IMPORTANT]
> After flashing, Wi-Fi is disabled by default. Go to **Network → Wireless**, click **Edit** for each network (2.4 GHz and 5 GHz), and check the **Maximum transmit power** field in the drop-down menu. If you see a value ≥ 20 dBm (100 mW), your radio calibration data in `dump_openwrt_xx-xx-xx.bin` was applied correctly.

✅ **Done! Your router is now running OpenWrt.**

---

## **Building OpenWrt from Source**

To build your own OpenWrt firmware:

```bash
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

In `menuconfig`, choose:

* **Target System:** MediaTek Ralink MIPS
* **Subtarget:** MT7620 based boards
* **Target Profile:** TP-Link MR30G v1

Then run:

```bash
make
```

This will download the sources, build the toolchain, kernel, and selected packages.

### **Upgrading Firmware**

Once you’ve built a new image, flash it via:

* **System → Backup / Flash Firmware → Flash new firmware image**

Use the image with the `sysupgrade` suffix (e.g., `openwrt-...-sysupgrade.bin`).

If that fails or causes problems, recover using **TFTP** mode:

### **TFTP Recovery Mode**

1. Set your PC's IP address to `192.168.0.66`.
2. Start Tftpd64 (or any TFTP server).
3. Place the image with the `tftp-recovery` suffix in the root folder of the TFTP program and rename it to `tp_recovery.bin`.
4. Connect your PC to the router via Ethernet.
5. Hold the **Reset** button and power on the router. Keep holding Reset.
6. The firmware transfer should begin — you will see a status/progress bar in the Tftpd64 window.

If this doesn't work or issues occur:

* Check your Ethernet cable.
* Ensure your PC IP is set correctly.
* Retry the procedure from the start.
