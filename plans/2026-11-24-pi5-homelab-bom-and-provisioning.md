---
project: pi5-homelab
title: Raspberry Pi 5 Homelab BOM and Provisioning Plan
author: gemini
executor: claude
status: draft   # draft → approved → in-progress → done
created: 2026-11-24
---

## Goal
Source, assemble, and provision a headless 64-bit Raspberry Pi 5 execution host (`pi5-ops`) to run Claude Code over SSH, long simulation sweeps (`plasmodium-atlas`, `emperorfall`) inside `tmux`, and scheduled data collectors feeding `ai_ops/telemetry/`.

## Context
- Hub rules: `AGENTS.md`, `GEMINI.md`, `STATE.md`, `plans/_TEMPLATE.md`.
- Replaces an earlier Pi 4 (`skynet`) dashboard concept (Telegram bot + multi-source collectors) with a faster, higher-RAM primary execution node; `skynet` can later be rescued or bridged over Ethernet.
- Local pickup planned for 2026-11-25 at Micro Center (Cambridge, MA). Prices below were verified against Micro Center Cambridge listings and online retail on 2026-11-24.

### Bill of Materials (BOM)

| Item | Why it's needed | Recommended spec | Approx. price (USD) | Price source (2026-11-24) | Required vs. Optional |
|---|---|---|---|---|---|
| **Raspberry Pi 5 — 16GB** *(Recommended)* | Primary always-on compute; soldered RAM is not upgradeable and gives headroom for parallel headless browsers, Node/Python collectors, and local models without swapping | Broadcom BCM2712 2.4 GHz quad-core Cortex-A76, 16 GB LPDDR4X-4267 (SKU `938035`) | $229.99 *(Holiday Deal; reg. $384.99)* | Micro Center Cambridge | Required (Recommended tier) |
| **Raspberry Pi 5 — 8GB** *(Minimal alternative)* | Same 2.4 GHz CPU at lower upfront cost; adequate for single-stream sweeps + collectors | Broadcom BCM2712 2.4 GHz quad-core Cortex-A76, 8 GB LPDDR4X-4267 (SKU `635649`) | $174.99 | Micro Center Cambridge | Required (Minimal tier only) |
| **Official 27W USB-C PSU** | Standard 5 V / 3 A supplies cap USB power at 600 mA and risk brownouts; 5.1 V / 5 A PD unlocks 1.6 A USB power and stable PCIe NVMe operation | Official Raspberry Pi 27W USB-C PD (5.1 V / 5 A, Black) | $11.99 | Micro Center Cambridge | Required |
| **Active Cooler** | Uncooled Pi 5 idles near ~65 °C and throttles under 4-core sweeps; PWM blower heatsink keeps sustained 4-core load well below 80 °C | Official Raspberry Pi 5 Active Cooler (SKU `620310`) | $9.99 | Micro Center Cambridge | Required |
| **M.2 NVMe HAT+** | Exposes Pi 5 PCIe 2.0 x1 lane (up to ~500 MB/s) for high-endurance NVMe OS and telemetry storage | Official Raspberry Pi M.2 HAT+ (Compact, M-key, 2230/2242) | $14.99 | Micro Center Cambridge | Optional (Recommended) |
| **NVMe SSD (Primary OS/Data)** | Prevents premature microSD wear from 24/7 SQLite writes, `git`/`npm` churn, and simulation logs | 256 GB M.2 2230 PCIe NVMe TLC SSD (e.g., Micron 2550 2230) | $55.00 | DriveSolutions.com *(check Micro Center shelf for 2230/2242)* | Optional (Recommended) |
| **microSD Card (A2 / Rescue)** | Primary OS drive on Minimal build; EEPROM bootloader update and rescue media on Recommended build | Minimal: SanDisk Extreme PLUS 64 GB U3 V30 A2 ($28.99); Recommended rescue: Micro Center 64 GB Ultra microSDXC ($19.99) | $28.99 *(Min)* / $19.99 *(Rec)* | Micro Center Cambridge | Required |
| **Case** | Protects board from dust and accidental shorts during 24/7 headless operation | Official Raspberry Pi 5 Case, Black/Gray *(see clearance note in Risks)* | $9.99 | Micro Center Cambridge | Optional (Recommended) |
| **Cat6 Ethernet Patch Cable** | Stable Gigabit link for headless SSH and sweeps; also enables direct Pi 5 ↔ Pi 4 (`skynet`) Ethernet rescue | 10 ft Cat6 UTP snagless patch cable | $6.99 | Best Buy *(comparable at Micro Center)* | Required |
| **Micro-HDMI to HDMI Cable** | Emergency local display access if SSH or UFW changes ever lock you out | Inland 3.3 ft Micro-HDMI to Standard HDMI (official 1 m is $7.99) | $5.99 | Micro Center Cambridge | Optional (Regret skipping) |
| **RTC Battery** | Keeps hardware clock accurate across power blips before NTP syncs (protects TLS/Git auth and telemetry timestamps) | Official Raspberry Pi 5 RTC Battery (Panasonic ML-2020 rechargeable) | $4.99 | Micro Center Cambridge | Optional (Regret skipping) |
| **Small UPS (Battery Backup)** | Prevents filesystem corruption on power flickers during unattended SQLite writes | Compact line-interactive UPS or Pi 5 UPS HAT *(not priced in store lookup)* | N/A | Defer to follow-up | Optional |

#### Storage Recommendation: microSD vs. NVMe via M.2 HAT+
- **Recommendation:** **NVMe SSD via M.2 HAT+** (Recommended build).
- **Justification:** 24/7 data collectors, SQLite telemetry writes, Node/Python package installs, and simulation logs generate continuous small random writes. Even A2 microSD cards over SDR104 cap out near ~90 MB/s sequential write and suffer high flash wear under 24/7 logging. PCIe 2.0 x1 NVMe reaches up to ~500 MB/s with proper SSD wear leveling and dramatically higher random IOPS.
- **One-stop Micro Center fallback (2280 SSD):** If Micro Center Cambridge is out of 2230/2242 NVMe drives, replace the Active Cooler ($9.99) + M.2 HAT+ ($14.99) + Official Case ($9.99) + 2230 SSD ($55.00) (`= $89.97`) with the **Argon40 ONE V3 M.2 NVMe PCIe Case** ($48.99 at Micro Center; includes PWM fan, full-size HDMI, and 2280 NVMe board) plus an in-stock **M.2 2280 NVMe SSD** (e.g., KLEVV CRAS C715 512 GB at $99.99 at Micro Center, or a lower-capacity 2280 drive on the shelf). You can also boot temporarily over USB 3.0 using `skynet`'s existing SSD.

#### BOM Totals (Math Shown, MA Sales Tax = 6.25%)
- **Minimal Build (8 GB + A2 microSD, open board with Active Cooler):**
  - Formula: `$174.99 (Pi 5 8GB) + $11.99 (27W PSU) + $9.99 (Active Cooler) + $28.99 (SanDisk 64GB A2) + $6.99 (Cat6)`
  - Pre-tax subtotal: **$232.95**
  - MA sales tax (`$232.95 × 0.0625`): **$14.56**
  - Total with tax: **$247.51**
- **Recommended Build (16 GB Holiday Deal + NVMe via M.2 HAT+ + Case + Rescue/Reliability Kit):**
  - Formula: `$229.99 (Pi 5 16GB) + $11.99 (27W PSU) + $9.99 (Active Cooler) + $14.99 (M.2 HAT+ Compact) + $55.00 (256GB 2230 NVMe) + $9.99 (Case) + $19.99 (64GB rescue SD) + $6.99 (Cat6) + $5.99 (Micro-HDMI) + $4.99 (RTC Battery)`
  - Pre-tax subtotal: **$369.91**
  - MA sales tax (`$369.91 × 0.0625`): **$23.12**
  - Total with tax: **$393.03**
  - *RAM-tier delta:* With the Holiday Deal, doubling RAM from 8 GB to 16 GB adds only **$55.00** (`$229.99 − $174.99`).

### Thermal and Power Assumptions

#### Power Budget & Electricity Cost
- **PSU output capacity:** `P_psu = 5.1 V × 5.0 A = 25.5 W` continuous (27 W USB-PD peak); unlocks `5.0 V × 1.6 A = 8.0 W` downstream USB budget.
- **Modeled wall power draw (sources: Raspberry Pi docs & PMIC/USB-C bench measurements):**
  - Board headless idle (Ethernet): **3.0–3.6 W** (+~1.0 W NVMe idle → **4.5 W** modeled idle)
  - Active Cooler PWM fan under load: **+0.5 W**
  - NVMe SSD active read/write on M.2 HAT+: **+1.0–2.0 W**
  - Sustained 4-core CPU stress load (board + Active Cooler + active NVMe): **~11.5 W** (peak with external USB SSD + HDMI: **up to 16.0 W**)
  - PSU headroom at extreme peak: `25.5 W − 16.0 W = 9.5 W` (~37% margin)
- **24/7 Operating Cost Model (Massachusetts):**
  - Duty cycle assumption: **20 h/day idle (4.5 W)** + **4 h/day sustained 4-core sweep (11.5 W)**.
  - Daily energy: `E_day = ((20 h × 4.5 W) + (4 h × 11.5 W)) / 1000 = (90 + 46) / 1000 = 0.136 kWh/day`
  - Weighted average power: `P_avg = 136 Wh / 24 h = 5.67 W`
  - Monthly energy (30 d): `0.136 × 30 = 4.08 kWh/month` | Annual energy (365 d): `0.136 × 365 = 49.64 kWh/year`
  - Annual cost at **$0.3049/kWh** (U.S. EIA MA residential avg, July 2026): `49.64 kWh × $0.3049/kWh = $15.14/year` (**~$1.26/month**)

#### Thermal Envelope (20–22 °C Ambient)
- **Uncooled idle:** ~65 °C (throttles rapidly under 4-core stress).
- **Official Active Cooler idle:** ~45 °C (~49 °C with M.2 HAT+ mounted overhead).
- **Official Active Cooler sustained 4-core load:** ~64 °C.
- **Soft thermal throttle:** **80 °C** (linearly reduces max ARM clock by 100 MHz per °C from 80 °C to 85 °C).
- **Hard thermal throttle:** **85 °C** (firmware drops ARM CPU from **2.4 GHz** to **1.5 GHz**).
- **Pass criteria for `pi5-ops`:** Peak `< 75.0 °C` over 10 minutes of 4-core load with `vcgencmd get_throttled` returning `throttled=0x0`.

## Steps
1. [ ] **Turn provisioning steps into versioned scripts** — files: `pi-setup/01-system-harden.sh`, `pi-setup/02-dev-toolchain.sh`, `pi-setup/03-thermal-check.sh`, `pi-setup/README.md` — command: `mkdir -p pi-setup && chmod +x pi-setup/*.sh && bash -n pi-setup/*.sh`
   - *Acceptance check:* All three scripts use `set -euo pipefail`, pass `bash -n` with exit code `0`, and contain zero hardcoded secrets or private keys.
2. [ ] **Flash 64-bit OS image (workstation step before first Pi boot)** — files: `pi-setup/README.md` — command: Using Raspberry Pi Imager, select **Raspberry Pi OS Lite (64-bit, Debian Trixie, 2026-10-06 release)**; in OS Customisation set hostname `pi5-ops`, username `alex`, timezone `America/New_York`, enable SSH with **Allow public-key authentication only** (paste laptop `~/.ssh/id_ed25519.pub`), and flash to NVMe SSD via USB adapter (or flash rescue microSD first, update bootloader via `sudo rpi-eeprom-config --edit` with `BOOT_ORDER=0xf416`, then clone/flash to NVMe). Connect over Ethernet for initial boot (avoids the 2026-10-06 Trixie Lite `wpasupplicant` headless Wi-Fi bug).
   - *Acceptance check:* `ssh alex@pi5-ops.local 'uname -m && dpkg --print-architecture && grep PRETTY_NAME /etc/os-release'` prints `aarch64`, `arm64`, and `trixie`.
3. [ ] [pi] **Verify hostname, storage root, and install OS updates + base packages** — files: `pi-setup/01-system-harden.sh` — command: `sudo hostnamectl set-hostname pi5-ops && sudo apt-get update && sudo apt-get full-upgrade -y && sudo apt-get install -y curl ca-certificates gnupg ufw tmux git python3-venv python3-pip stress-ng jq nvme-cli unattended-upgrades`
   - *Acceptance check:* `hostname` returns `pi5-ops`, `lsblk -o NAME,TRAN,SIZE,MOUNTPOINT` shows `/` on `nvme0n1` (or `mmcblk0` on minimal build), and `apt-get -s upgrade | grep -q '^0 upgraded'` exits `0`.
4. [ ] [pi] **Enforce key-only SSH hardening** — files: `pi-setup/01-system-harden.sh` (writes `/etc/ssh/sshd_config.d/99-hardening.conf`) — command: Write `PasswordAuthentication no`, `KbdInteractiveAuthentication no`, `PermitRootLogin no`, and `PubkeyAuthentication yes` to `/etc/ssh/sshd_config.d/99-hardening.conf`, then run `sudo sshd -t && sudo systemctl reload ssh`.
   - *Acceptance check:* `sudo sshd -T | grep -E '^(passwordauthentication|permitrootlogin|pubkeyauthentication) '` outputs `passwordauthentication no`, `permitrootlogin no`, and `pubkeyauthentication yes`; a second SSH login from laptop succeeds before closing the active session.
5. [ ] [pi] **Enable UFW firewall with rate-limited SSH** — files: `pi-setup/01-system-harden.sh` — command: `sudo ufw default deny incoming && sudo ufw default allow outgoing && sudo ufw limit 22/tcp comment 'SSH rate-limited' && sudo ufw --force enable`
   - *Acceptance check:* `sudo ufw status verbose` reports `Status: active`, `Default: deny (incoming), allow (outgoing)`, and `22/tcp LIMIT IN Anywhere`.
6. [ ] [pi] **Configure `tmux` defaults for long sweeps** — files: `pi-setup/02-dev-toolchain.sh` (writes `~/.tmux.conf`) — command: Write `set -g history-limit 50000`, `set -g mouse on`, `set -g base-index 1`, and `setw -g pane-base-index 1` to `~/.tmux.conf`; verify with `tmux new-session -d -s ops-check 'sleep 5' && tmux ls`.
   - *Acceptance check:* `tmux -V` succeeds and `tmux ls` lists `ops-check` detached cleanly.
7. [ ] [pi] **Configure `git` and dedicated GitHub SSH key** — files: `pi-setup/02-dev-toolchain.sh` — command: `git config --global user.name "Alex Balcarcel" && git config --global user.email "<alex-github-email>" && git config --global init.defaultBranch main && ssh-keygen -t ed25519 -C "pi5-ops" -f ~/.ssh/id_ed25519_github -N ""` (configure `~/.ssh/config` for `Host github.com` pointing to `~/.ssh/id_ed25519_github`, print `~/.ssh/id_ed25519_github.pub` for Alex to add under GitHub → Settings → SSH and GPG keys).
   - *Acceptance check:* `ssh -T git@github.com` returns `Hi Roids01! You've successfully authenticated`.
8. [ ] [pi] **Install Node.js 26 Active LTS and Claude Code CLI** — files: `pi-setup/02-dev-toolchain.sh` — command: `curl -fsSL https://deb.nodesource.com/setup_26.x | sudo -E bash - && sudo apt-get install -y nodejs && mkdir -p ~/.npm-global && npm config set prefix ~/.npm-global && grep -qxF 'export PATH="$HOME/.npm-global/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc && export PATH="$HOME/.npm-global/bin:$PATH" && npm install -g @anthropic-ai/claude-code` (Alex then runs `claude` interactively inside `tmux` to complete Claude Pro OAuth login).
   - *Acceptance check:* `node -v` outputs `v26.13.1` or higher (Node 26 Active LTS, Nov 2026 security release) and `claude --version` exits `0`.
9. [ ] [pi] **Create Python virtual environment for telemetry collectors** — files: `pi-setup/02-dev-toolchain.sh` — command: `python3 -m venv ~/venvs/ai-ops && ~/venvs/ai-ops/bin/pip install --upgrade pip wheel`
   - *Acceptance check:* `~/venvs/ai-ops/bin/python --version` reports Python 3.13+ and `~/venvs/ai-ops/bin/pip --version` points inside `/home/alex/venvs/ai-ops/`.
10. [ ] [pi] **Clone `Roids01/ai_ops` and verify repo layout** — files: `pi-setup/02-dev-toolchain.sh` — command: `mkdir -p ~/repos && git clone git@github.com:Roids01/ai_ops.git ~/repos/ai_ops && git -C ~/repos/ai_ops pull --rebase && ls -la ~/repos/ai_ops`
    - *Acceptance check:* `~/repos/ai_ops/AGENTS.md` and `~/repos/ai_ops/STATE.md` exist and `git -C ~/repos/ai_ops status -sb` shows a clean `main` branch.
11. [ ] [pi] **Run 10-minute thermal & throttle check and log summary** — files: `pi-setup/03-thermal-check.sh`, `telemetry/pi5-homelab-thermal.csv` — command: Run `stress-ng --cpu 4 --timeout 600s --metrics-brief` while logging `timestamp,temp_c,arm_mhz,throttled_hex` every 15 seconds (40 rows, ~2 KB total, well within the 100 KB `telemetry/` cap) from `vcgencmd measure_temp`, `vcgencmd measure_clock arm`, and `vcgencmd get_throttled` into `~/repos/ai_ops/telemetry/pi5-homelab-thermal.csv`.
    - *Acceptance check:* CSV has header + 40 timestamped rows, peak `temp_c < 75.0`, sustained `arm_mhz >= 2400`, and final `throttled_hex` equals `0x0` (zero undervoltage, zero thermal throttling).

## Acceptance checks
- [ ] Pi 5 boots 64-bit Raspberry Pi OS Lite (Debian Trixie, `aarch64`/`arm64`) with key-only SSH and active UFW firewall (`22/tcp` rate-limited).
- [ ] `tmux`, `git` (SSH-authenticated to `Roids01`), Node.js `v26.13.1+`, `claude --version`, and `~/venvs/ai-ops` all pass verification on-device.
- [ ] `Roids01/ai_ops` is cloned at `~/repos/ai_ops` with executable, syntax-checked scripts committed under `pi-setup/`.
- [ ] 10-minute 4-core `stress-ng` run completes with peak temp `< 75.0 °C`, `throttled=0x0`, and summary CSV (`< 100 KB`) committed to `telemetry/pi5-homelab-thermal.csv`.

## Risks / open questions
- **Case & cooler clearance (Active Cooler + M.2 HAT+ + Official Case):** The official Pi 5 Case ($9.99) includes its own lid fan; stacking both the Active Cooler ($9.99) and an M.2 HAT+ prevents the lid from snapping shut. **Question for Alex:** Do you prefer (A) Official Case + M.2 HAT+ Compact using the case's built-in fan, (B) Active Cooler + M.2 HAT+ open-frame / lid-off, or (C) Argon40 ONE V3 M.2 NVMe Case ($48.99 at Micro Center) + a standard 2280 NVMe SSD?
- **In-store 2230/2242 NVMe stock:** Micro Center Cambridge stocks the M.2 HAT+ Compact ($14.99), but 2230/2242 SSD availability fluctuates. **Question for Alex:** If no 2230/2242 SSD is on the shelf tomorrow, should we pivot to Argon40 ONE V3 + 2280 SSD, or boot temporarily from `skynet`'s existing USB SSD over USB 3.0?
- **Remote SSH outside your home LAN:** UFW blocks inbound traffic except LAN SSH; adding **Tailscale** as an optional Step 12 lets you SSH into `pi5-ops` from your commute or work without router port-forwarding. **Question for Alex:** Include Tailscale in `pi-setup/`?
- **Hostname & Git identity:** Confirm `pi5-ops` as the hostname (vs. `skynet2` or `pi5-homelab`) and confirm the GitHub email address (`<alex-github-email>`) to set in Step 7.
- **Headless Wi-Fi setup caveat on Trixie Lite (2026-10-06):** A known `wpasupplicant` update issue can drop headless Wi-Fi during first boot; Step 2 avoids this by bootstrapping over Cat6 Ethernet first.
- **Power flickers / UPS:** Running unattended SQLite collectors without a small UPS carries minor filesystem corruption risk during winter power flickers; consider adding a compact UPS before deploying persistent databases.

## Results (filled in by executor)
- Commits:
- Metrics:
- Notes: