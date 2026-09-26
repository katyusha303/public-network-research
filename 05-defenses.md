# DEFENSIVE MEASURES

## Overview

This section outlines the defensive measures that can prevent, detect, or mitigate each phase of the Evil Twin attack chain. The defenses are organized by attack phase, followed by broader recommendations for individuals and organizations.

## Defense Against Phase 1: Reconnaissance

An attacker performing reconnaissance is passively scanning the airwaves. This phase is difficult to prevent entirely because 802.11 beacon frames are broadcast openly by design. However, detection is possible.

### Detection

- **Wireless Intrusion Detection Systems (WIDS):** Tools like Kismet, Snort with wireless plugins, or commercial WIDS can detect abnormal scanning behavior, such as a single device probing many networks rapidly.
- **Monitor for rogue devices:** Watch for unfamiliar MAC addresses appearing on the network.

### Prevention

- **Reduce signal leakage:** Adjust router transmit power so the Wi-Fi signal does not extend beyond the physical boundary of your home or office.
- **Disable SSID broadcast (weak defense):** Hiding the SSID does not stop determined attackers, but it raises the effort required.

## Defense Against Phase 2: Evil Twin AP

This is the most critical phase to defend against, as the attacker is now broadcasting a fake network.

### Detection

- **Monitor for duplicate SSIDs:** Use a Wi-Fi analyzer app (like WiFi Analyzer on Android) to check for multiple access points broadcasting the same SSID with different BSSIDs. A duplicate SSID with an unknown BSSID is a red flag.
- **Check the BSSID:** If you connect to a familiar network, verify the BSSID matches the one you normally connect to.
- **WIDS with Evil Twin detection:** Enterprise WIDS solutions can automatically alert when a rogue AP is broadcasting a known SSID.

### Prevention

- **Disable auto-connect:** Turn off "auto-join" for open or public networks on your phone and laptop. This prevents your device from silently connecting to a fake AP.
- **Forget old networks:** Regularly remove saved networks you no longer use, especially public ones.
- **Use WPA3 where possible:** WPA3 uses SAE (Simultaneous Authentication of Equals), which is resistant to offline dictionary attacks and provides better protection against rogue APs.
- **Verify with certificates:** In enterprise environments, use EAP-TLS with certificate validation so devices reject fake APs that lack the correct certificate.

## Defense Against Phase 3: DHCP and DNS Hijacking

Once on the fake AP, the attacker controls DHCP and DNS.

### Detection

- **Check your gateway and DNS:** If your device suddenly uses an unusual gateway (e.g., `10.0.0.1` instead of your usual `192.168.1.1`), you may be on a rogue network.
- **Monitor DNS responses:** Tools like `dig` or browser extensions can show if DNS responses are suspicious.

### Prevention

- **DNS over HTTPS (DoH) or DNS over TLS (DoT):** Enable encrypted DNS in your browser or operating system. This prevents attackers from intercepting and spoofing your DNS queries.
- **Use a VPN:** A VPN encrypts all traffic including DNS, so even if the attacker redirects DNS, they cannot read or modify your requests.
- **Static DNS configuration:** Configure trusted DNS servers manually (e.g., `1.1.1.1` or `8.8.8.8`), although a rogue DHCP server can override this.

## Defense Against Phase 4: Captive Portal Phishing

The attacker serves a fake login page to steal credentials.

### Detection

- **Check the URL:** A real captive portal is usually served over HTTPS by a known domain. A portal at a raw IP address (like `http://10.0.0.1`) is suspicious.
- **Look for HTTPS:** If the page asking for your password is HTTP (no padlock), do not enter anything.
- **Be suspicious of unexpected prompts:** A captive portal appearing on a network you did not just connect to is a red flag.

### Prevention

- **Never enter credentials on a captive portal:** If you must log in, verify the network is legitimate first.
- **Use a password manager:** Password managers will not autofill credentials on a fake domain, alerting you to the phishing attempt.
- **Enable two-factor authentication (2FA):** Even if credentials are stolen, 2FA blocks the attacker from logging in.
- **Use a VPN before logging in:** Connect to a VPN first, then browse. The attacker cannot intercept encrypted VPN traffic.

## Defense Against Phase 5: Deauthentication Attack

The attacker forces the victim off the real network.

### Detection

- **Monitor for deauth floods:** WIDS can detect a sudden spike in deauthentication frames.
- **Sudden disconnections:** If your device repeatedly disconnects from a known network for no reason, you may be under attack.

### Prevention

- **Enable Protected Management Frames (PMF / 802.11w):** PMF cryptographically protects deauthentication and disassociation frames. Without PMF, these frames can be forged. PMF is mandatory in WPA3 and available in WPA2.
- **Upgrade to WPA3:** WPA3 requires PMF, making deauth attacks ineffective.
- **Use a wired connection where possible:** Ethernet is immune to wireless deauth attacks.

## Defense Against Phase 6: Credential Harvest and MITM

The attacker captures credentials and intercepts traffic.

### Detection

- **ARP spoofing detection:** Tools like `arpwatch` or XArp can alert when a MAC address changes for a known IP.
- **Monitor for duplicate MACs:** If two devices claim the same IP, an ARP spoof is in progress.
- **Wireshark:** Capture and analyze traffic for signs of ARP poisoning or DNS spoofing.

### Prevention

- **Use HTTPS everywhere:** HTTPS encrypts traffic, so even if intercepted, the attacker sees only ciphertext.
- **Enable HSTS:** HTTP Strict Transport Security forces browsers to use HTTPS for known sites.
- **Use a VPN:** A VPN tunnels all traffic through an encrypted channel, defeating MITM entirely.
- **Dynamic ARP Inspection (DAI):** On managed switches, enable DAI to block ARP spoofing.
- **Network segmentation:** Isolate sensitive devices from public networks.

## Broader Recommendations for Individuals

1. **Never trust public Wi-Fi:** Treat all open networks as hostile.
2. **Use a VPN:** This is the single most effective defense against public network attacks.
3. **Enable 2FA everywhere:** It stops credential theft from becoming account takeover.
4. **Keep devices updated:** Security patches fix vulnerabilities that attackers exploit.
5. **Turn off auto-connect:** Prevent devices from silently joining unfamiliar networks.
6. **Verify network names:** Check the exact SSID spelling and, if possible, the BSSID.
7. **Avoid sensitive activities on public Wi-Fi:** Banking, email, and login pages should wait until you are on a trusted network.

## Broader Recommendations for Organizations

1. **Deploy WPA3-Enterprise with EAP-TLS:** Certificate-based authentication prevents rogue APs.
2. **Enable PMF (802.11w) across all access points:** Blocks deauth attacks.
3. **Implement WIDS/WIPS:** Detect rogue APs, deauth floods, and Evil Twin attempts.
4. **Use Network Access Control (NAC):** Ensure only authorized devices connect.
5. **Segment networks:** Keep guest Wi-Fi separate from internal resources.
6. **Enforce VPN for remote workers:** Protect traffic on untrusted networks.
7. **Train employees:** Security awareness training reduces phishing success rates.
8. **Monitor for credential leaks:** Use services that alert when corporate credentials appear on the dark web.
9. **Deploy EDR on endpoints:** Detect malicious activity even if a device connects to a rogue network.
10. **Regularly audit wireless environment:** Check for rogue APs and unauthorized devices.

## Conclusion of Defensive Analysis

The Evil Twin attack chain exploits multiple weaknesses in wireless and network protocols. No single defense is sufficient. Effective protection requires a layered approach combining protocol-level protections (WPA3, PMF, HTTPS), user awareness, and network monitoring. The most important individual action is to use a VPN on any untrusted network. The most important organizational action is to deploy WPA3-Enterprise with certificate-based authentication and enable PMF across all access points.