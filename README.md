# MAAS Hardware Tutorial: Deploying a NUC

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

## 2. Connect your device to the MAAS network

Connect your hardware to the **same physical Ethernet network** where your MAAS region/rack controller resides (or will reside). This is the same machine you'll be installing MAAS on, or one directly reachable from it. Do this **before** worrying about the MAAS install — you'll need physical access anyway.

---

## 3. Enter the BIOS and configure boot

You’ll need to access the BIOS or UEFI firmware setup screen to enable PXE boot.

### 🛠️ How to enter the BIOS
The exact key to press depends on your hardware, but common options include:

- **F2**, **F10**, **F12**, **Delete**, or **Esc**

To find the right key for your device:
- Check the splash screen on boot (usually says “Press F2 to enter Setup”)
- Look up the model online (e.g., "Intel NUC 7i5 BIOS key")

**Tip:** Start tapping the key repeatedly as soon as the machine powers on.

### ⚙️ What to change in the BIOS

- **Set PXE as the first boot source**
   - Usually under *Boot Options* or *Boot Order*
   - Enable PXE/Network boot if it’s not already available

- **Disable Secure Boot** (optional but recommended)
   - Some PXE environments won't boot with Secure Boot enabled

- **Enable Wake-on-LAN** (WOL)
   - Found under *Power Management*, *Advanced*, *Network*, or *Chipset...* sections
   - This allows MAAS or another system to power on the device remotely via its MAC address

- **Enable S5/G0 Power State Restart**
   - This setting ensures the machine powers back on automatically if it loses power and then regains it (e.g., via smart plug or PDU)
   - Look for options like “AC Power Recovery,” “Power On After Power Fail,” or “Restore Power State”

---

## 🔌 What if there's no BMC?

MAAS ideally manages power through a BMC (Baseboard Management Controller), but most cheap Mini-PCs or NUCs **don’t have one**. In that case:

You’ll need a way to power the system on automatically:

- **Option 1: Wake-on-LAN (WOL)**  
  MAAS can send a "magic packet" to the machine’s MAC address to turn it on remotely, if WOL is enabled in BIOS and supported by the NIC.

- **Option 2: S5/G0 Auto Power-On**  
  If your BIOS supports restarting automatically when power is restored, you can control power with a smart plug or USB-controlled PDU. MAAS "powers on" the system by cutting and restoring power.

---

## 4. Initial MAAS setup

Install MAAS (Snap recommended) on a physical machine you'll use as your **region/rack controller**:

```bash
sudo snap install maas --channel=3.4/stable
```

### Recommended early settings:
- Set DNS to `8.8.8.8` to avoid resolution issues during image import
- Import Ubuntu 22.04 LTS (Jammy) under Images → OS selection
- **Do NOT enable DHCP yet** — this will break your LAN if done prematurely

---

## 5. Monitor MAAS logs in real-time

Tail MAAS activity with helpful filters and save to a local file:
```bash
sudo journalctl -f | grep --color=auto -i maas | tee /tmp/maas-tail.log
```

---

## 6. PXE boot test (without DHCP)

- Power on your new machine (NUC, Mini-PC, etc.)
- It should try PXE boot, then stall at the PXE screen
   - This is expected, because MAAS isn’t serving DHCP yet
- Power the machine off manually

---

## 7. Enable DHCP in MAAS

In the MAAS UI:
- Go to **Subnets → 192.168.x.x/24** (or your actual subnet)
- Click **Edit DHCP**
- Set **DNS to your region controller IP** (e.g., `192.168.1.134`)

---

## 8. Commission the machine

- Power the machine back on
- It should now boot via PXE and start being provisioned
- In your MAAS logs you should see it obtain an IP
- The console will show a commissioning sequence
- It may fail the first time due to missing `metadata.maas` resolution
- Fix this by ensuring your DHCP points to MAAS for DNS

### Reboot, retry, commission
- After correcting DNS, reboot the machine again
- It should appear in MAAS as a "New" machine with "Unknown" power type

### Steps:
- Change power type to **Manual**
- Choose **Start Commissioning**, check **Allow SSH, prevent power off**
- Power on the machine manually
- UI will show the full commissioning progress
- Result: Machine status becomes **Ready**

---

## 9. Deploy the machine via CLI

- In MAAS UI, **allocate the machine**
- Get the machine system ID (from URL or list view)
- On your MAAS server:

```bash
# Generate your API key
sudo maas apikey --username $PROFILE --generate

# Log in via CLI
maas login $PROFILE $MAAS_URL $API_KEY

# Deploy to Ubuntu 22.04 (jammy)
maas $PROFILE machine deploy $SYSTEM_ID distro_series=ubuntu/jammy
```

- Power on the machine
- Monitor logs; reboot will happen automatically
- Machine status should become **Deployed**

### Verify deployment:
```bash
ssh ubuntu@$ASSIGNED_IP
lsblk
cd / && ls
```

---

## 8. Automation scripts (Optional)

If you have a smart plug or network PDU, you can automate power control.  And you'll find these scripts in the repo, as well, so no hurry to type them in.

### `turn-on-machine.sh`
```bash
#!/bin/bash
curl http://$PDU_IP/relay/0?turn=on
```

### `turn-off-machine.sh`
```bash
#!/bin/bash
curl http://$PDU_IP/relay/0?turn=off
```

### `commission-machine.sh`
```bash
#!/bin/bash
maas $PROFILE machine commission $SYSTEM_ID
./turn-on-machine.sh
sleep 4.2m
./turn-off-machine.sh
```

### `deploy-machine.sh`
```bash
#!/bin/bash
maas $PROFILE machine deploy $SYSTEM_ID distro_series=ubuntu/jammy
./turn-on-machine.sh
```

### `release-machine.sh`
```bash
#!/bin/bash
./turn-off-machine.sh
maas $PROFILE machine release $SYSTEM_ID
```

---

## ✅ Success!
You just deployed a thrift-store Mini-PC with MAAS — on real hardware, no cloud required.
Now go build your lab, cluster, or infrastructure-on-a-budget.
