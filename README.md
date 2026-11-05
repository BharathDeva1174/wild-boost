# WildBoost

**A Windows memory optimizer that tells you what it actually did.**

One file, no installer, no network access, no telemetry. It releases the memory caches Windows
lets an application release, shows you the pressure that is really holding the machine back, and
gives you reversible controls for the things that cost more than RAM: startup entries, disk space,
background CPU.

---

## Verified publisher

I am the publisher of this software, and the binary is signed by me.

"Verified" here means every value below can be checked against the file you downloaded, with the
commands further down — not a store listing, and not a badge. If they match, the binary came from me
and nobody changed it since.

| | |
|---|---|
| Authenticode subject | `CN=BharathDeva, O=BharathDeva` |
| Certificate SHA-1 thumbprint | `52DCDF8571CD528A165BB089E57FA83618E3C229` |
| Strong-name public key token | `99d2cb19424c8911` |
| Version / file | `1.0.0.0`, 2,166,120 bytes |
| SHA-256 of `WildBoost.exe` | `af338067ae4d507f52e8ac52308ff50c6accf4ae25ac54767b0bd3a297ae9b9c` |
| Licence | GPL-3.0-or-later, text embedded inside the executable |

Check it yourself, in PowerShell:

```powershell
Get-AuthenticodeSignature .\WildBoost.exe | Format-List Status, StatusMessage
(Get-AuthenticodeSignature .\WildBoost.exe).SignerCertificate | Format-List Subject, Thumbprint
(Get-Item .\WildBoost.exe).VersionInfo | Format-List ProductName, FileVersion, CompanyName
(Get-FileHash .\WildBoost.exe -Algorithm SHA256).Hash
```

The signing certificate is **my own, not one bought from a certificate authority**. That has one
practical consequence you should know before the first run: SmartScreen may show *"Unknown publisher"*
or a warning on the download. The signature is real and verifiable — it just isn't pre-trusted by
Windows. If the thumbprint above matches the certificate on the file, the binary is mine and
unmodified; you can install the certificate as a trusted root to make the warning go away for good.

---

## Quick start

1. **Download** [`WildBoost.exe`](https://github.com/BharathDeva1174/wild-boost/raw/main/WildBoost.exe)
   — it is committed here, next to this README, so that link is the file itself.
2. **Check it** against the SHA-256 above if you want to be sure nothing changed in transit.
3. **Run it.** It asks for administrator rights — that is not decoration, the cache-release calls
   it makes are privileged. Accept the UAC prompt.
4. **Press `Optimize`.** That is the whole product in one button: the areas ticked above the button
   are released, and the card under it tells you how much came back and how long it took.
5. Optional, once: tick **Run on startup** and **Minimize to tray**, pick a **hotkey**, and you never
   have to open the window again.

Requires Windows 10 or 11, 64-bit, and **.NET Framework 4.8** (already present on any supported
Windows 10/11 install — nothing else to install).

---

## Usage

### The memory areas
The pills above the `Optimize` button are the caches Windows permits an application to trim:
standby list, modified page list, combined page list, registry cache, system file cache, working set.
Tick only what you want touched; the choice is remembered. One of them is deliberately off by
default because releasing it costs more than it returns on a healthy machine.

### When it is worth pressing
The **Commit pressure** card is the honest answer to that question. Free RAM is a cache figure —
Windows fills idle RAM with cache on purpose and gives it back when a program needs it. *Commit* is
the promise the system has already made, and when commit is near its limit, or the paging rate is
high, flushing caches cannot help, and the app says so instead of offering you another button.

| What you see | What to do |
|---|---|
| Free memory low, commit comfortable | Nothing. That memory is being used as cache. |
| Commit above the warning threshold | Close what is holding commit — the card ranks them for you. |
| Hard faults read from disk are high | Same ranking; a bigger page file is a manual decision, not a button. |
| A single app ballooned over 30 minutes | The leak watcher flags it; trim or close that one process. |

### Per-process actions
Each row in the memory list has four controls, and none of them is global:
- **Trim** — release that one process's working set. Its pages come back on demand.
- **Eco** — Windows efficiency mode for that process (the documented `ProcessPowerThrottling` API).
  Reversible, not persisted, and it drops automatically when the window takes focus.
- **Exclude** — never touch this process during optimization.
- **Close** — asks the application to shut down. This is the only irreversible action in the app,
  so it always opens a confirmation that names every application and window count involved.

### Automation
- **Schedule** — a fixed interval, or pressure-driven: trigger on commit percentage and paging rate
  instead of a clock.
- **Hotkey** — one global key combination to optimize from anywhere.
- **Tray icon** — background optimization, optional live memory usage, custom colours.
- **Close after optimization**, **run on low priority**, **always on top**, **start-menu shortcut**.

### Beyond RAM
- **Storage** — measures a fixed whitelist (user temp, Windows temp, browser cache, thumbnail cache,
  Delivery Optimization cache, Recycle Bin) and deletes only the rows you ticked, after showing you
  the sizes. Files in use are skipped and reported, never forced. The Windows component store is
  analysed read-only and its report is shown verbatim.
- **Startup** — lists every autostart entry with its publisher and whether the target still exists.
  Disabling moves the entry into WildBoost's own quarantine and restores it byte-for-byte later;
  nothing is deleted.
- **GPU** — per-adapter video memory (usage, budget, available) and engine utilisation, from
  documented DXGI and WMI sources. Where a number cannot be measured with public APIs, the card says
  so instead of inventing one.
- **Advanced diagnostics** — live performance counters, thermal zone readings, and hardware error
  records from the system log.

### Things WildBoost deliberately does not do
No registry tweaks (editing the registry is not a memory mechanism, it is only risk). No page-file
resizing (a wrong value can strand a machine at boot, so it stays a manual decision). No service
disabling. No automatic standby-list flushing on a timer. No hidden persistence of any kind — every
setting is visible in the window and stored under `HKLM\SOFTWARE\WildBoost`.

---

## What a flush is actually worth

I measured this on my own machine over a two-hour study, and the app reflects what the data said:
24 optimizations freed **+0.32 GB on average**, and roughly half of it was consumed again within
**42 seconds**, while free memory never once dropped below 5.77 GB. Commit, meanwhile, sat at 82 % of
its limit with 41 % of RAM nominally free.

That is not an argument against the button — it is the reason the button is paired with the commit
ranking, the paging rate and the leak watcher instead of being sold to you as a speed boost. If your
machine is short of something, WildBoost is built to tell you *what*, even when the answer is "not
free cache".

---

## Licence and source

GPL-3.0-or-later. The full licence text is embedded inside the executable and opens from the link in
the app's footer, so the terms travel with the binary rather than living only in a copyright string.

This repository publishes the executable. The corresponding source is available from the publisher on
request, at no charge, for at least three years from this release — email the address on the
repository owner's GitHub profile and reference the version and SHA-256 above.
