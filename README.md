# IT Support Troubleshooting Lab

## Objective
Document real troubleshooting scenarios for Windows, networking, Linux, and user support — demonstrating the diagnostic process used in enterprise IT environments.

---

## Scenario 1: User Cannot Access the Internet

### Symptoms
- User reports browser shows "No Internet" or pages fail to load
- Other users on the same network are unaffected
- Device shows network connection as "Connected"

### Tools Used
- `ipconfig` — check IP address, subnet mask, default gateway, DNS
- `ping` — test connectivity to gateway and external hosts (8.8.8.8)
- `nslookup` — verify DNS resolution is working
- `traceroute` / `tracert` — identify where packets are dropping
- Windows Network Settings / Network Troubleshooter

### Troubleshooting Steps
1. Run `ipconfig /all` — confirm device has a valid IP (not 169.254.x.x APIPA)
2. Run `ping 127.0.0.1` — verify local TCP/IP stack is functional
3. Run `ping <default gateway>` — test connectivity to router
4. Run `ping 8.8.8.8` — test external IP connectivity (bypasses DNS)
5. Run `nslookup google.com` — test DNS resolution
6. If DNS fails, try setting DNS manually to 8.8.8.8 or 1.1.1.1
7. Run `ipconfig /release` then `ipconfig /renew` to refresh DHCP lease
8. Run `netsh winsock reset` if all above pass but browser still fails
9. Check if issue is browser-specific (try different browser or incognito)
10. Escalate to network team if gateway is unreachable

### Root Cause
DHCP lease expired and the device received an APIPA address (169.254.x.x), indicating the DHCP server was temporarily unreachable.

### Fix
Ran `ipconfig /release` and `ipconfig /renew` to obtain a valid IP from the DHCP server. Internet access restored immediately.

### Lessons Learned
- Always check `ipconfig` first — an APIPA address immediately points to a DHCP issue
- `ping 8.8.8.8` vs `ping google.com` quickly isolates DNS vs connectivity failures
- Document the fix and resolution time for SLA tracking in ServiceNow

---

*More scenarios coming: Password Reset, Slow PC, Printer Not Found, VPN Issues*
