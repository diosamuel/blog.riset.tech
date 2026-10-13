---
title: "Raspberry Pi WiFi Dead for 2 Months — Full Debug Walkthrough"
description: "Step-by-step debugging a Raspberry Pi that disappeared from the network after crash-reboots"
date: "June 22 2026"
draft: false
---

 > AI Generated Content - for documentation purposes only

> "Yesterday it worked, today it's gone." — Every Pi owner ever

## The Problem

Raspberry Pi 3B running Debian Trixie (Raspberry Pi OS). WiFi was working, then suddenly the Pi disappeared from the network. No SSH, no nmap, nothing. Pulled the SD card to investigate.

---

## Step 1: Boot Check — Did it even boot?

**File:** `var/log/cloud-init-output.log`

```
ci-info: |  eth0  | False |     .     |     .     |   .   | b8:27:eb:b3:3e:d3 |
ci-info: | wlan0  | False |     .     |     .     |   .   | b8:27:eb:e6:6b:86 |
```

Both `eth0` and `wlan0` are `False` at boot. But cloud-init checks at ~16 seconds — Wi-Fi might connect later. **Inconclusive.**

---

## Step 2: Was WiFi EVER working?

**File:** `var/log/wtmp`

```
pts/0    192.168.1.7
pts/1    192.168.1.7
```

User `pi` logged in from `192.168.1.7` multiple times via SSH. **WiFi WAS working.**

**File:** `var/log/postgresql/postgresql-17-main.log`

```
2026-06-21 14:27:40.586 WIB — starting PostgreSQL
2026-06-21 14:27:40.821 WIB — database system was not properly shut down
```

Pi was running on June 21. **Verdict:** Something broke it recently.

---

## Step 3: When did it break?

From `cloud-init-output.log`:

```
Sun, 21 Jun 2026 07:07:12 UTC — Boot
Sun, 21 Jun 2026 07:22:24 UTC — Reboot (15 min later)
Sun, 21 Jun 2026 07:27:33 UTC — Reboot (5 min later)
Sun, 21 Jun 2026 07:27:38 UTC — Reboot (5 SECONDS later)
Sun, 21 Jun 2026 07:37:50 UTC — Final boot
```

**Five reboots in 30 minutes.** PostgreSQL confirms crash each time: "not properly shut down."

**Verdict:** June 21 — rapid crash-reboots killed WiFi.

---

## Step 4: Why didn't WiFi come back?

### Check 1: Netplan config

**File:** `etc/netplan/`

```
-rw-------  1 root root    0 Jun 21 14:07 90-NM-75a1216a-...yaml
-rw-------  1 root root    0 Jun 21 14:07 90-NM-cfc034ee-...yaml
```

**Both files are 0 bytes.** Empty. Should contain WiFi credentials but don't.

### Check 2: wpa_supplicant

**File:** `etc/wpa_supplicant/`

```
action_wpa.sh
functions.sh
ifupdown.sh
```

**No `wpa_supplicant.conf`.** The backup WiFi config doesn't exist.

### Check 3: NetworkManager profiles

**File:** `etc/NetworkManager/system-connections/`

**(empty directory)** — Zero saved connections.

### Check 4: Cloud-init has the config but refuses to apply it

**File:** `var/log/cloud-init.log`

```
applying net config names for {
  'wifis': {
    'wlan0': {
      'access-points': {
        'SigmaFam': {
          'password': '5b106cad4e0954732d3db82f3da0a8ec...'
        }
      }
    }
  }
}
```

Then:

```
No network config applied. Neither a new instance nor datasource network update allowed
```

Cloud-init **refuses to apply** because it's not the first boot.

### Check 5: Missing module

```
Could not find module named cc_netplan_nm_patch
```

This module should write cloud-init's WiFi config to netplan files. **It's missing.**

---

## Step 5: Why did it work before?

**Answer:** NetworkManager's runtime memory.

When WiFi was first set up, NetworkManager saved the connection in memory (`/var/lib/NetworkManager/`). As long as the Pi **never rebooted**, this persisted.

The moment the Pi rebooted 5 times on June 21, the runtime state was wiped. No saved config on disk = no reconnection.

---

## Step 6: What happened on June 21?

1. **Before June 21:** WiFi working. Config only in memory. Netplan files empty. No one noticed.
2. **~14:07:** Pi crashes. PostgreSQL: "not properly shut down."
3. **~14:22:** Reboots again. 15 minutes uptime. Something unstable.
4. **~14:27:** Reboots twice in 5 seconds. Power instability or kernel panic.
5. **~14:37:** Final boot. NetworkManager starts with no saved WiFi profile. wlan0 stays down.
6. **After that:** Pi alive but invisible on the network.

---

## Root Cause

| Issue | Detail |
|-------|--------|
| **Primary** | WiFi config never persisted to disk — only in memory |
| **Trigger** | Rapid crash-reboots wiped runtime state |
| **Contributing** | `cc_netplan_nm_patch` cloud-init module missing |
| **Contributing** | No `wpa_supplicant.conf` as backup |

---

## The Fix

Three files written to the SD card from a Linux laptop:

### 1. `/etc/netplan/99-wifi.yaml`

```yaml
network:
  version: 2
  wifis:
    wlan0:
      dhcp4: true
      dhcp6: true
      optional: true
      regulatory-domain: ID
      access-points:
        "SigmaFam":
          password: "5b106cad4e0954732d3db82f3da0a8ec2308a6dd09840db1041f0db59d9220b0"
```

### 2. `/etc/wpa_supplicant/wpa_supplicant.conf`

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=ID

network={
    ssid="SigmaFam"
    psk="5b106cad4e0954732d3db82f3da0a8ec2308a6dd09840db1041f0db59d9220b0"
}
```

### 3. Remove old empty files

```bash
rm /etc/netplan/90-NM-75a1216a-...yaml
rm /etc/netplan/90-NM-cfc034ee-...yaml
```

Now WiFi config is **persisted on disk**. Next reboot, it auto-connects.

---

## Lessons Learned

1. **Always verify WiFi config is on disk** — `nmcli connection show` should list saved profiles
2. **Don't trust runtime state alone** — memory gets wiped on reboot
3. **Check `vcgencmd get_throttled`** after random reboots — power issues cause most Pi instability
4. **Cloud-init on Raspberry Pi OS has quirks** — `cc_netplan_nm_patch` module is listed but doesn't exist
5. **Keep a `wpa_supplicant.conf` as backup** — even if NetworkManager is primary

---

## Quick Reference: What to Check First

| Symptom | Check first |
|---------|-------------|
| Pi not on network | `cloud-init-output.log` — was wlan0 ever True? |
| WiFi worked before, not after reboot | `etc/netplan/` — are files 0 bytes? |
| Never had WiFi | `etc/wpa_supplicant/wpa_supplicant.conf` — exists? |
| Random reboots | `postgresql-17-main.log` — "not properly shut down" count |
| After package upgrade | `apt/history.log` — what changed? |
| Can't SSH in | `wtmp` — any login records at all? |

---

*Debugged: June 22, 2026 | Pi 3B | Debian Trixie | Kernel 6.18.33+rpt-rpi-v8*
