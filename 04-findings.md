# FINDINGS

## Overview

This section presents the results of the Evil Twin attack chain performed in the isolated lab environment. Each phase of the attack is documented with observations, command outputs, and screenshots. The findings confirm that the attack chain is functional and that a victim device can be forced onto a fake access point, redirected to a captive portal, and have its credentials captured — without the victim realizing anything is wrong.

## Phase 1: Reconnaissance Findings

The initial scan using `airodump-ng` successfully identified the target network. The following information was collected:

- **ESSID:** Wifi-Repeater (the legitimate router being cloned)
- **BSSID:** 80:3F:5D:97:12:35
- **Channel:** 11
- **Encryption:** WPA2 CCMP PSK

Multiple client devices were observed connected to the target access point, including the test phone. The scan also revealed other nearby networks, confirming the adapter was correctly in monitor mode and capable of capturing 802.11 frames.

**Figure 1:** `01-airodump-target.png` — airodump-ng output showing the target network and connected clients.

## Phase 2: Evil Twin AP Findings

The `hostapd-mana` tool successfully created a fake access point broadcasting the same SSID as the legitimate router. Key observations:

- The fake AP started successfully with `AP-ENABLED` status.
- MANA mode was active, and the tool responded to probe requests from nearby devices.
- Multiple devices (including the test phone) sent directed probe requests for various SSIDs, and the fake AP responded to them.
- A device with MAC `28:7b:11:7c:fa:99` connected to the fake AP shortly after it was started.
- Another device with MAC `14:ea:63:0c:2a:58` also connected.

**Figure 2:** `02-hostapd-ap-enabled.png` — hostapd-mana showing AP-ENABLED and connected stations.

**Figure 3:** `02-phone-wifi-list.png` — phone's Wi-Fi list showing the fake SSID "Wifi-Repeater".

## Phase 3: DHCP and DNS Findings

The `dnsmasq` server successfully assigned IP addresses to connecting devices and redirected their DNS queries. Observations:

- The AP interface (`wlan1`) was configured with IP `10.0.0.1/24`.
- `dnsmasq` started successfully and bound to `wlan1`.
- A DHCP lease was offered to the test phone: `DHCPOFFER(wlan1) 10.0.0.97`.
- DNS queries from the phone (e.g., `connectivitycheck.gstatic.com`, `www.google.com`) were intercepted and redirected to `10.0.0.1`.

**Figure 4:** `03-dnsmasq-dhcp-lease.png` — dnsmasq showing DHCP lease offered to the victim device.

## Phase 4: Captive Portal Findings

The fake captive portal was hosted using Apache and PHP. Observations:

- The fake login page (`index.html`) was served correctly.
- The PHP capture script (`capture.php`) was created to log submitted passwords.
- **Issue encountered:** The PHP script could not write to `/tmp/captured.txt` due to permission restrictions on Apache's `www-data` user.
- **Resolution:** The log path was changed to `/var/www/html/captured.txt`, and ownership was set to `www-data`. This resolved the issue.

**Figure 5:** `04-portal-files.png` — directory listing showing portal files.

**Figure 6:** `04-captive-portal-phone.png` — the fake login page displayed on the victim's phone.

## Phase 5: Deauthentication Findings

The deauthentication attack was performed using `aireplay-ng`. Observations:

- Initial attempts failed with `No such BSSID available` due to targeting the wrong BSSID and channel mismatch.
- After locking the monitor adapter to the correct channel (channel 11) and targeting the correct BSSID (`80:3F:5D:97:12:35`), the deauth attack successfully ran.
- The victim phone disconnected from the legitimate network and reconnected to the fake AP automatically, as the SSID matched a saved network in its Preferred Network List.

**Figure 7:** `05-aireplay-deauth.png` — aireplay-ng sending deauth packets.

**Figure 8:** `05-phone-connected-fake.png` — phone connected to the fake AP after deauth.

## Phase 6: Credential Harvest and MITM Findings

### Credential Capture

When the test password (`test123`) was submitted on the captive portal, it was successfully captured in the log file:
