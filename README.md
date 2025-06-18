# MAAS Hardware Tutorial: Deploying to a Mini-PC or NUC

This is an early-stage tutorial designed to help you deploy real hardware — like a Mini-PC, NUC, or even a sufficiently capable laptop — using [MAAS](https://maas.io). It’s meant as a **proof-of-concept**, not a product-grade solution. That said, it can work surprisingly well on low-cost hardware with a little setup.

> 💡 Contributions welcome! If this worked for you — or didn’t — PRs and issue reports are encouraged.

---

## 1. Requirements

This tutorial doesn't assume any one device model. Whether you're using a Mini-PC, a small-form-factor desktop, or an old laptop with PXE support and a decent BIOS, you're good to go. Don’t get hung up on the term "NUC."

That said, here’s what you *do* need:

| Requirement | Example Values | Notes |
|-------------|----------------|-------|
| CPU         | Intel (64-bit) | ARM may work, but isn't covered here |
| Boot Method | PXE boot support | PXE should be enabled or moved to the top of boot order |
| Power Control | Wake-on-LAN or manual power | BMC is ideal, but not required |
| Network     | Wired Ethernet NIC | Required for PXE and MAAS discovery |
| Monitor     | Any display you can read | Helpful for BIOS config and debugging |
| Keyboard    | USB or built-in | Needed to access BIOS or boot menu |

---

## 2. Connect Your Device to the MAAS Network

Connect your hardware to the **same physical Ethernet network** where your MAAS region/rack controller resides (or will reside). This is the same machine you'll be installing MAAS on, or one directly reachable from it. Do this **before** worrying about the MAAS install—you'll need physical access anyway.

---

## 3. Enter the BIOS and Configure Boot

You’ll need to access the BIOS or UEFI firmware setup screen to enable PXE boot.

### 🛠️ How to Enter the BIOS
The exact key to press depends on your hardware, but common options include:

- **F2**, **F10**, **F12**, **Delete**, or **Esc**

To find the right key for your device:
- Check the splash screen on boot (usually says “Press F2 to enter Setup”)
- Look up the model online (e.g., "Intel NUC 7i5 BIOS key")

**Tip:** Start tapping the key repeatedly as soon as the machine powers on.

### ⚙️ What to Change in the BIOS

1. **Set PXE as the first boot source**
   - Usually under *Boot Options* or *Boot Order*
   - Enable PXE/Network boot if it’s not already available

2. **Disable Secure Boot** (optional but recommended)
   - Some PXE environments won't boot with Secure Boot enabled

3. **Enable Wake-on-LAN** (WOL)
   - Found under *Power Management*, *Advanced*, or *Network* sections
   - This allows MAAS or another system to power on the device remotely via its MAC address

4. **Enable S5/G0 Power State Restart**
   - This setting ensures the machine powers back on automatically if it loses power and then regains it (e.g., via smart plug or PDU)
   - Look for options like “AC Power Recovery,” “Power On After Power Fail,” or “Restore Power State”

---

## 🔌 What If There's No BMC?

MAAS ideally manages power through a BMC (Baseboard Management Controller), but most cheap Mini-PCs or NUCs **don’t have one**. In that case:

You’ll need a way to power the system on automatically:

- **Option 1: Wake-on-LAN (WOL)**  
  MAAS can send a "magic packet" to the machine’s MAC address to turn it on remotely, if WOL is enabled in BIOS and supported by the NIC.

- **Option 2: S5/G0 Auto Power-On**  
  If your BIOS supports restarting automatically when power is restored, you can control power with a smart plug or USB-controlled PDU. MAAS "powers on" the system by cutting and restoring power.

---

Next step: we’ll walk through installing MAAS, importing boot resources, and deploying your first device.

