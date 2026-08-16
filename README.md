# Wifiphisher

**Rogue Access Point framework for Wi-Fi security testing**

> Modernized and maintained by **iXludaTech**

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![License](https://img.shields.io/badge/license-GPL-blue.svg)
![Version](https://img.shields.io/badge/version-2.0.0-green.svg)

---

## What is Wifiphisher?

Wifiphisher is a security tool that performs automated Wi-Fi association attacks to gain a man-in-the-middle position against wireless clients. Once positioned, it can serve customized phishing pages to capture credentials (WPA/WPA2 pre-shared keys, login passwords) or deliver payloads.

**This is a penetration testing tool.** Only use it on networks you own or have explicit written authorization to test.

---

## How It Works

Wifiphisher operates in two phases:

### Phase 1: Man-in-the-Middle Position

Wifiphisher uses multiple techniques to associate nearby Wi-Fi clients with its rogue access point:

| Technique | Description |
|-----------|-------------|
| **Evil Twin** | Creates a fake AP that mimics a legitimate network |
| **KARMA** | Responds to any probe request, impersonating networks clients have connected to before |
| **Known Beacons** | Broadcasts beacon frames for popular ESSIDs that nearby devices likely remember |
| **Deauthentication** | Sends forged deauth/disassoc frames to disconnect clients from the real AP |

### Phase 2: Phishing / Credential Capture

Once clients connect to the rogue AP, all HTTP/HTTPS traffic is redirected to a captive portal. Wifiphisher serves platform-aware phishing pages (Windows, macOS, iOS, Android) that request:

- WPA/WPA2 Pre-Shared Keys
- Social login credentials (OAuth)
- Custom form data

Captured credentials are logged to the terminal, a text file, and a **JSON export file** for easy integration with reporting tools.

---

## Architecture

```
wifiphisher/
├── bin/wifiphisher              # CLI entry point
├── pyproject.toml               # Modern Python packaging
├── wifiphisher/
│   ├── __init__.py              # Package metadata
│   ├── pywifiphisher.py         # Main engine & argument parsing
│   ├── common/
│   │   ├── constants.py         # Configuration & constants
│   │   ├── interfaces.py        # Wireless interface management (pyric/pyw)
│   │   ├── opmode.py            # Operation mode selection (8 modes)
│   │   ├── accesspoint.py       # Rogue AP (roguehostapd / hostapd)
│   │   ├── phishinghttp.py      # Tornado HTTP/HTTPS server
│   │   ├── phishingpage.py      # Template management
│   │   ├── extensions.py        # Extension manager (plugin system)
│   │   ├── recon.py             # AP discovery (scapy)
│   │   ├── tui.py               # Curses-based terminal UI
│   │   ├── firewall.py          # iptables NAT/redirect rules
│   │   ├── sniffer.py           # MITM packet sniffer (net-creds based)
│   │   ├── macmatcher.py        # OUI/vendor lookup
│   │   ├── victim.py            # Connected client tracking
│   │   ├── globals.py           # Shared state
│   │   ├── utilities.py         # Helper functions
│   │   └── uimethods.py         # Tornado UI method decorators
│   ├── extensions/
│   │   ├── deauth.py            # Deauthentication attack
│   │   ├── lure10.py            # Windows Location Service exploit
│   │   ├── knownbeacons.py      # Known Beacons attack
│   │   ├── wpspbc.py            # WPS Push Button Connect
│   │   ├── handshakeverify.py   # WPA handshake verification
│   │   └── roguehostapdinfo.py  # Rogue hostapd information
│   └── data/
│       ├── phishing-pages/      # Phishing scenario templates
│       │   ├── oauth-login/     # Social network login
│       │   ├── firmware-upgrade/# Router firmware upgrade
│       │   ├── wifi_connect/    # Network manager password prompt
│       │   ├── plugin_update/   # Browser plugin update
│       │   └── captive-portal-custom/ # Custom captive portal [NEW]
│       ├── cert/                # SSL certificates
│       ├── logos/               # Vendor logos
│       └── locs/                # Lure10 capture data
└── tests/                       # Test suite
```

---

## Requirements

### Hardware
- A Linux system (Kali Linux recommended)
- One wireless network adapter that supports **AP mode** and **Monitor mode**
- Adapter must support packet injection
- A second adapter is recommended for simultaneous deauth + AP (optional)

### System Packages
```bash
# Debian/Kali
sudo apt-get install libnl-3-dev libnl-genl-3-dev hostapd dnsmasq libssl-dev
```

### Python
- Python 3.8 or higher
- pip (Python package manager)

---

## Installation

### From Source (Recommended)

```bash
git clone https://github.com/iXludaTech/wifiphisher.git
cd wifiphisher
pip install -e .
```

### Using pip (when published)

```bash
pip install wifiphisher
```

### System Dependencies Check

Wifiphisher will check for required system libraries during installation. If anything is missing, it will tell you exactly what to install.

---

## Quick Start

```bash
# Run with automatic interface detection
sudo wifiphisher

# Or from the repository
sudo python bin/wifiphisher
```

The interactive TUI will guide you through:
1. Selecting a target access point
2. Choosing a phishing scenario
3. Monitoring connected victims and captured credentials

---

## Usage Examples

### Basic Attack (Interactive)

```bash
sudo wifiphisher
```

### Target a Specific Network

```bash
sudo wifiphisher --essid "TARGET_WIFI" -p captive-portal-custom
```

### Use Specific Interfaces

```bash
sudo wifiphisher -aI wlan0 -jI wlan1 -p firmware-upgrade
```

### WPA-Protected Rogue AP

```bash
sudo wifiphisher --essid "CONFERENCE_WIFI" -p plugin_update -pK s3cr3tp4ssw0rd
```

### With Handshake Verification

```bash
sudo wifiphisher -aI wlan0 -jI wlan4 -p firmware-upgrade -hC handshake.pcap
```

### Open Network with Known Beacons

```bash
sudo wifiphisher --essid "FREE WI-FI" -p oauth-login -kB
```

### MITM with Internet Sharing

```bash
sudo wifiphisher -iI eth0 -p captive-portal-custom
```

---

## Command-Line Options

| Short | Long | Description |
|-------|------|-------------|
| `-h` | `--help` | Show help message |
| `-i` | `--interface` | Interface supporting both AP & monitor modes |
| `-eI` | `--extensionsinterface` | Interface for monitor mode (deauth) |
| `-aI` | `--apinterface` | Interface for rogue AP |
| `-iI` | `--internetinterface` | Interface with internet access |
| `-pI` | `--protectinterface` | Interfaces to protect from NetworkManager |
| `-mI` | `--mitminterface` | Interface for MITM attack |
| `-e` | `--essid` | ESSID for rogue AP (skips AP selection) |
| `-p` | `--phishingscenario` | Phishing scenario to use |
| `-pK` | `--presharedkey` | WPA/WPA2 PSK for rogue AP |
| `-hC` | `--handshake-capture` | PCAP file for handshake verification |
| `-qS` | `--quitonsuccess` | Stop after first credential capture |
| `-nE` | `--noextensions` | Don't load extensions |
| `-nD` | `--nodeauth` | Skip deauthentication phase |
| `-dC` | `--deauth-channels` | Channels to deauth (e.g. `1,3,7`) |
| `-dE` | `--deauth-essid` | Deauth all BSSIDs with this ESSID |
| `-dK` | `--disable-karma` | Disable KARMA attack |
| `-kB` | `--known-beacons` | Enable Known Beacons attack |
| `-lC` | `--lure10-capture` | Capture BSSIDs for Lure10 |
| `-lE` | `--lure10-exploit` | Exploit Windows Location Service |
| `-iAM` | `--mac-ap-interface` | Custom MAC for AP interface |
| `-iEM` | `--mac-extensions-interface` | Custom MAC for extensions interface |
| `-iNM` | `--no-mac-randomization` | Disable MAC randomization |
| `-pPD` | `--phishing-pages-directory` | Custom phishing pages directory |
| `-pE` | `--phishing-essid` | Custom ESSID for phishing page |
| `-cP` | `--credential-log-path` | Path for credential log file |
| `-cM` | `--channel-monitor` | Monitor target AP channel changes |
| `-wP` | `--wps-pbc` | Monitor WPS Push Button |
| `-wAI` | `--wpspbc-assoc-interface` | Interface for WPS association |
| `-fH` | `--force-hostapd` | Use system hostapd instead of roguehostapd |
| `-kN` | `--keepnetworkmanager` | Don't kill NetworkManager |
| `--logging` | | Enable logging to file |
| `-lP` | `--logpath` | Custom log file path |
| `--payload-path` | | Payload file for delivery scenarios |
| `--dnsmasq-conf` | | Custom dnsmasq.conf path |

---

## Phishing Scenarios

### Built-in Scenarios

| Scenario | Description |
|----------|-------------|
| `captive-portal-custom` | **NEW** - Professional captive portal for WPA password collection |
| `wifi_connect` | Imitates OS network manager (Windows/macOS/iOS/Android) |
| `firmware-upgrade` | Fake router firmware upgrade page |
| `oauth-login` | Social network OAuth login page |
| `plugin_update` | Browser plugin update (supports payload delivery) |

### Creating Custom Scenarios

1. Create a directory under `wifiphisher/data/phishing-pages/your-scenario/`
2. Add a `config.ini`:

```ini
[info]
Name: My Custom Scenario
Description: Description of what this scenario does.

[context]
page_title = Custom Page Title
```

3. Create `html/index.html` using [Tornado template syntax](https://www.tornadoweb.org/en/stable/template.html):

```html
<!DOCTYPE html>
<html>
<head><title>{{ page_title }}</title></head>
<body>
    <h1>{{ target_ap_essid }}</h1>
    <form method="POST">
        <input type="password" name="password" placeholder="Wi-Fi Password">
        <button type="submit">Connect</button>
    </form>
</body>
</html>
```

4. Available template variables:

| Variable | Description |
|----------|-------------|
| `{{ target_ap_essid }}` | Target network name |
| `{{ target_ap_bssid }}` | Target AP MAC address |
| `{{ target_ap_channel }}` | Target AP channel |
| `{{ target_ap_encryption }}` | Target encryption type |
| `{{ target_ap_vendor }}` | Target AP vendor |
| `{{ target_ap_logo_path }}` | Vendor logo path |
| `{{ rogue_ap_essid }}` | Rogue AP name |
| `{{ page_title }}` | Page title from config |

---

## Operation Modes

Wifiphisher automatically selects the best operation mode based on available hardware:

| Mode | Cards | Interfaces | Features |
|------|-------|------------|----------|
| OP_MODE1 | 2 | AP + Monitor | Full attacks, channel hopping |
| OP_MODE2 | 3 | AP + Monitor + Internet | Full attacks + internet sharing |
| OP_MODE3 | 2 | AP + Internet | No deauth, internet sharing |
| OP_MODE4 | 1 | AP only | No deauth, no extensions |
| OP_MODE5 | 1 (VIF) | AP + Monitor | Full attacks, no channel hopping |
| OP_MODE6 | 1 (VIF) + 1 | AP + Monitor + Internet | Attacks + internet, no hopping |
| OP_MODE7 | 3 | AP + Monitor + Managed | WPS association support |
| OP_MODE8 | 1 (VIF) + 1 | AP + Monitor + Managed | WPS with VIF |

---

## Extensions

Extensions are modular plugins that run on the monitor-mode interface:

| Extension | Function |
|-----------|----------|
| `deauth` | Deauthentication/disassociation attacks |
| `lure10` | Windows Location Service manipulation |
| `knownbeacons` | Broadcasts popular ESSID beacons |
| `wpspbc` | WPS Push Button Connect monitoring |
| `handshakeverify` | WPA handshake capture and verification |
| `roguehostapdinfo` | Rogue hostapd status information |

### Writing Custom Extensions

Extensions must implement:
- `__init__(self, data)` - Initialize with shared data
- `get_packet(self, pkt)` - Process captured packets
- `send_output(self)` - Return status messages
- `send_channels(self)` - Return channels of interest
- `on_exit(self)` - Cleanup resources

---

## Credential Output

Credentials are captured from multiple sources:

### 1. Phishing Page POST Data
When victims submit forms on the captive portal, credentials are extracted via regex matching on `password|pwd|pass` and `username|uname|name` fields.

### 2. MITM Packet Sniffer
When operating with internet sharing mode, the built-in sniffer (based on net-creds) captures:
- HTTP/HTTPS basic auth
- FTP credentials
- Telnet logins
- Mail authentication (IMAP/POP/SMTP)
- NTLM hashes
- SNMP community strings

### 3. JSON Export
All captured credentials are automatically exported to `/tmp/wifiphisher-credentials.json`:

```json
[
  {
    "timestamp": "2024-01-15 14:30:22",
    "source_ip": "10.0.0.42",
    "data": "password=MySecret123",
    "user_agent": "Mozilla/5.0 ...",
    "url": "http://10.0.0.1/"
  }
]
```

---

## Raspberry Pi Deployment

Wifiphisher runs well on Raspberry Pi for portable attacks:

```bash
# On Kali ARM for Raspberry Pi
sudo apt-get install libnl-3-dev libnl-genl-3-dev hostapd dnsmasq libssl-dev
git clone https://github.com/iXludaTech/wifiphisher.git
cd wifiphisher
pip install -e .

# Run with a USB wireless adapter
sudo wifiphisher
```

**Recommended adapters:** Alfa AWUS036ACH, Alfa AWUS036ACSM, Panda PAU09

---

## Logging

Enable detailed logging with:

```bash
sudo wifiphisher --logging
# Logs to wifiphisher.log in current directory

sudo wifiphisher --logging -lP /var/log/wifiphisher.log
# Custom log path

sudo wifiphisher --logging -cP /tmp/creds.log
# Also log credentials to a separate file
```

---

## Troubleshooting

### "Please run as root"
```bash
sudo wifiphisher
```

### "dnsmasq not found"
```bash
sudo apt-get install dnsmasq
```

### "No interface supports AP mode"
Ensure your wireless adapter supports AP mode and its driver is loaded:
```bash
iw list | grep -A 10 "Supported interface modes"
```

### "hostapd failed to launch"
Another process may be using port 80 or 443:
```bash
sudo fuser -k 80/tcp 443/tcp
```

### NetworkManager conflicts
```bash
# Or use the flag
sudo wifiphisher -kN
```

---

## Contributing

Contributions are welcome. Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

### Development Setup

```bash
git clone https://github.com/iXludaTech/wifiphisher.git
cd wifiphisher
pip install -e ".[dev]"
pytest
```

---

## Credits

Originally created by [Dan McInerney](https://github.com/DanMcInerney) in 2015.

Maintained and modernized by **iXludaTech**.

Full contributor list: [Contributors](https://github.com/iXludaTech/wifiphisher/graphs/contributors)

---

## License

Wifiphisher is licensed under the **GNU General Public License v3.0**. See [LICENSE.txt](LICENSE.txt) for details.

---

## Disclaimer

Usage of Wifiphisher for attacking infrastructures without prior mutual consent is illegal. It is the user's responsibility to obey all applicable local, state, and federal laws. The authors assume no liability and are not responsible for any misuse or damage caused by this program.

**Only use this tool on networks you own or have explicit authorization to test.**
