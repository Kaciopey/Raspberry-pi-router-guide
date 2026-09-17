# Setting Up a Raspberry Pi as a Router with OpenWrt

A guide to turning a Raspberry Pi into a dedicated router using OpenWrt, a free and open-source Linux firmware built specifically for routing, firewalling, and network management.

## Table of Contents

- [What You'll Need](#what-youll-need)
- [Understanding the Pi's Limitation](#understanding-the-pis-limitation)
- [Step 1: Download the Right OpenWrt Image](#step-1-download-the-right-openwrt-image)
- [Step 2: Flash the Image to Your SD Card](#step-2-flash-the-image-to-your-sd-card)
- [Step 3: First Boot and Initial Access](#step-3-first-boot-and-initial-access)
- [Step 4: Set a Root Password](#step-4-set-a-root-password)
- [Step 5: Configure WAN and LAN](#step-5-configure-wan-and-lan)
- [Step 6: Set Up WiFi (Optional)](#step-6-set-up-wifi-optional)
- [Step 7: Configure the Firewall](#step-7-configure-the-firewall)
- [Step 8: Install Useful Packages](#step-8-install-useful-packages)
- [Testing Your Router](#testing-your-router)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## What You'll Need

- A Raspberry Pi 3B+, 4, or 5 (4 or 5 recommended for better throughput)
- A microSD card, 8GB minimum; a quality brand is worth it
- A computer to flash the image and a microSD card reader
- An Ethernet cable (or two, if you want a wired uplink and a wired downlink)
- A USB to Ethernet adapter, if you want two physical wired ports (see the note below)
- A power supply appropriate for your Pi model

## Understanding the Pi's Limitation

Unlike a typical router with a WAN port and several LAN ports, most Raspberry Pi models only have a single built-in Ethernet port. There are two common ways to work around this:

1. **WiFi WAN, Ethernet LAN**: The Pi connects to your upstream internet over its WiFi radio, and its Ethernet port serves your wired LAN devices. This is the simplest setup and works well for smaller networks.
2. **USB Ethernet adapter**: add a second NIC via USB (chipsets like the Realtek RTL8153 are well supported), giving you a true wired WAN port and a wired LAN port, similar to a normal router.

This guide covers the simpler WiFi WAN approach, with notes on adapting it if you add a second Ethernet port.

## Step 1: Download the Right OpenWrt Image

1. Go to the official firmware selector at `firmware-selector.openwrt.org`
2. Search for your exact Pi model (e.g. "Raspberry Pi 4B")
3. Download the "factory" or "sysupgrade" image listed for your board. If this is a fresh install (no existing OpenWrt on the card), grab the factory/combined image
4. You'll typically get a `.img.gz` file

Always check the OpenWrt Table of Hardware page for your specific model to confirm current support status and known limitations before you start.

## Step 2: Flash the Image to Your SD Card

1. Install a flashing tool like Raspberry Pi Imager or balenaEtcher on your computer
2. Insert your microSD card into your computer
3. In the flashing tool, choose "Use custom image" and select the OpenWrt `.img.gz` file you downloaded (most tools handle the gzip automatically)
4. Select your SD card as the target and flash it
5. Once flashing finishes, eject the card safely

## Step 3: First Boot and Initial Access

1. Insert the microSD card into the Pi and connect it to your computer via ethernet (no other network connections yet)
2. Power on the Pi and wait about a minute for it to fully boot
3. Set your computer's ethernet adapter to a static IP in the `192.168.1.x` range, or just let it pull an address via DHCP since OpenWrt's default LAN runs a DHCP server
4. Open a browser and go to `192.168.1.1`, this loads LuCI, OpenWrt's web interface
5. Log in with the default username `root` and no password

You can also reach the device over SSH with `ssh root@192.168.1.1` if you prefer the command line.

## Step 4: Set a Root Password

This is the first thing to do after logging in, since the device ships with no password set.

1. In LuCI, go to **System → Administration**
2. Enter and confirm a new root password
3. Save and apply

## Step 5: Configure WAN and LAN

1. Go to **Network → Interfaces**
2. You'll see a default LAN interface bound to the ethernet port. Leave this as is, or adjust the IP range under **Edit → General Setup** if `192.168.1.1` conflicts with your existing network
3. Create or edit the WAN interface and set its protocol depending on how you're connecting upstream:
   - **DHCP client**, if your upstream connection (WiFi network, modem, or hotspot) hands out an address automatically
   - **PPPoE**, if your ISP requires those credentials directly on this device
4. If using the WiFi WAN approach, you'll bind this WAN interface to the wireless radio once WiFi is configured in the next step

## Step 6: Set Up WiFi (Optional)

If you're using WiFi as your WAN uplink, or want to broadcast a WiFi network from the Pi:

1. Go to **Network → Wireless**
2. Click **Scan** to find your upstream WiFi network, or **Add** to create a new access point
3. For the WAN uplink: join your existing WiFi network with its password, then assign that wireless interface to the WAN network under interface settings
4. For a broadcast access point (e.g. if using a USB adapter as WAN and want the Pi's radio to serve LAN clients over WiFi): set the interface mode to **Access Point**, give it an SSID, choose WPA2/WPA3 encryption, and assign it to the LAN network
5. Save and apply

## Step 7: Configure the Firewall

OpenWrt ships with a working default firewall (LAN trusted, WAN restricted), but it's worth reviewing:

1. Go to **Network → Firewall**
2. Confirm your WAN zone has masquerading (NAT) enabled and input traffic rejected by default
3. Confirm your LAN zone accepts traffic and can forward to WAN
4. Add specific port forwarding rules only if you need to expose an internal service, under **Firewall → Port Forwards**

## Step 8: Install Useful Packages

OpenWrt's package manager (opkg on older releases, apk on newer ones) lets you extend functionality well beyond a stock router. From LuCI's **System → Software** tab, or via SSH, consider installing:

- **luci-app-adblock** or **luci-app-https-dns-proxy**, for network wide ad blocking and encrypted DNS
- **wireguard-tools** and **luci-app-wireguard**, for a VPN tunnel
- **luci-app-statistics**, for traffic graphs and monitoring
- **luci-app-sqm**, for smart queue management and better bufferbloat control under load

Update the package lists first (**System → Software → Update Lists**) before installing anything.

## Testing Your Router

1. Connect a device to the Pi's LAN (via Ethernet or the WiFi access point you configured)
2. Confirm it receives an IP address in the LAN's DHCP range
3. Confirm it can reach the internet (try loading a website)
4. Check **Status → Overview** in LuCI to confirm WAN shows a valid upstream IP address

## Troubleshooting

- **Can't reach 192.168.1.1 after flashing**: double-check your computer's Ethernet adapter isn't set to a conflicting static IP, and confirm the Pi has fully booted (LED activity should settle after about a minute)
- **No internet on LAN clients**: recheck the WAN interface protocol and confirm it actually has an upstream IP under **Status → Overview**
- **WiFi WAN keeps dropping**: some USB WiFi dongles have spotty OpenWrt driver support; the Pi's onboard WiFi chip is generally more reliable for this role
- **Pi feels sluggish under load**: this is common on the 3B+ given its weaker CPU; a Pi 4 or 5 handles routing and any additional services (ad blocking, VPN) far more comfortably
- **OpenWrt doesn't recognize your USB-to-Ethernet adapter**: the base image often doesn't ship the driver for every USB Ethernet chipset. A common fix is to temporarily set the Pi's onboard LAN port as a WAN source (giving the Pi a path to the internet) so you can update package lists and install the missing driver. The correct package depends on your adapter's chipset, so check yours with `lsusb` over SSH and look up the matching kmod. A few common ones:
  - `kmod-usb-net-rtl8152`, for Realtek RTL8152/RTL8153 based adapters (the most common chipset in USB Ethernet dongles)
  - `kmod-usb-net-asix` or `kmod-usb-net-asix-ax88179`, for ASIX-chipset adapters
  - `kmod-usb-net-cdc-ether`, for basic CDC Ethernet class devices

  Once the driver is installed and the adapter is recognized, you can revert the LAN port to its normal role.

## Next Steps

Once the basics are working, natural next projects include setting up VLANs to segment your network, adding a WireGuard tunnel for remote access, running Pi-hole or AdGuard Home for DNS-level ad blocking, and setting up SQM to reduce bufferbloat on your connection.

---

*Always verify current supported hardware and image links against the official OpenWrt wiki before flashing, since supported devices and download locations do change between releases.*
