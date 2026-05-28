## Objective
Document production helpdesk troubleshooting scenarios for Windows, networking, Linux, and user support ; demonstrating the diagnostic methodology, root cause analysis, and remediation process used in enterprise IT environments.

**Environment:** Simulated multi-user production environment | Windows 10/11, Active Directory, ServiceNow

---

## Scenario 1: User Cannot Access the Internet

### Symptoms
- User reports browser shows "No Internet" or pages fail to load
- Other users on the same network are unaffected
- Device shows network connection as "Connected"

### Tools Used
- `ipconfig` : check IP address, subnet mask, default gateway, DNS
- `ping` : test connectivity to gateway and external hosts (8.8.8.8)
- `nslookup` : verify DNS resolution is working
- `traceroute` / `tracert` : identify where packets are dropping
- Windows Network Settings / Network Troubleshooter

### Troubleshooting Steps
1. Run `ipconfig /all` : confirm device has a valid IP (not 169.254.x.x APIPA)
2. Run `ping 127.0.0.1` : verify local TCP/IP stack is functional
3. Run `ping <default gateway>` : test connectivity to router
4. Run `ping 8.8.8.8` : test external IP connectivity (bypasses DNS)
5. Run `nslookup google.com` : test DNS resolution
6. If DNS fails, set DNS manually to 8.8.8.8 or 1.1.1.1 to isolate the issue
7. Run `ipconfig /release` then `ipconfig /renew` to refresh DHCP lease
8. Run `netsh winsock reset` if all above pass but browser still fails
9. Check if issue is browser-specific (try different browser or incognito)
10. Escalate to network team if gateway is unreachable

### Root Cause Analysis

**Problem:** User unable to access the internet despite showing as "Connected."

**Investigation:**
- Verified physical connectivity : cable seated, switch port active
- Ran `ipconfig /all` : device showed APIPA address (169.254.x.x), confirming no valid DHCP lease
- Pinged gateway — timed out, confirming network layer failure
- Confirmed DHCP server was temporarily unreachable during lease renewal window

**Root Cause:** Expired DHCP lease resulted in automatic APIPA self-assignment (169.254.x.x), preventing all network communication.

**Resolution:** Ran `ipconfig /release` and `ipconfig /renew`. Device obtained valid IP from DHCP server. Internet access restored immediately.

**Preventive Action:** Documented DHCP lease duration settings and verified DHCP scope had sufficient available addresses. Logged incident and resolution in ServiceNow.

### Business Impact

### Additional DNS Observation

During testing, ICMP connectivity to external IP addresses was successful using:

```
ping 8.8.8.8
```

However, DNS resolution testing with:

```
nslookup google.com
```

returned:

```
*** Unknown can't find google.com: No response from server
```

#### Analysis

This indicates that basic network connectivity was functional, but DNS name resolution experienced issues communicating with the configured DNS server.

#### Troubleshooting Performed

- Verified IPv4 connectivity
- Reviewed configured DNS servers using `ipconfig /all`
- Tested external connectivity with ICMP
- Confirmed issue was isolated to DNS resolution

#### Lesson Learned

Successful network connectivity does not always guarantee successful DNS resolution. Troubleshooting should isolate connectivity issues from name resolution failures.

---

## Scenario 2: User Cannot Access Shared Network Folder

### Symptoms
- User receives "Network path not found" or "Access denied" when accessing shared drive
- Other users on the same team can access the folder without issues
- User was able to access it previously

### Tools Used
- `ping` : verify network connectivity to file server
- `nslookup` : confirm DNS resolution of server hostname
- Windows Credential Manager : check stored credentials
- Active Directory Users and Computers : verify group membership and permissions
- Event Viewer : check for authentication or access errors
- `net use` : view and manage mapped network drives

### Troubleshooting Steps
1. Confirm the shared path is correct : ask user for exact path they are using
2. Run `ping <server hostname>` : verify network connectivity to file server
3. Run `nslookup <server hostname>` : confirm DNS is resolving correctly
4. Try accessing the share via IP address instead of hostname — isolates DNS vs. access issue
5. Check Windows Credential Manager — remove stale/cached credentials for the server
6. Verify user is still a member of the correct AD security group (Active Directory Users and Computers)
7. Check share permissions AND NTFS permissions — both must allow access
8. Check Event Viewer on the server for Event ID 5140 (share access) or 4625 (failed logon)
9. Re-map the drive using `net use` or Group Policy if permissions are confirmed correct
10. Escalate to sysadmin if NTFS/share permissions require changes beyond helpdesk scope

### Root Cause Analysis

**Problem:** User received "Access Denied" on a previously accessible shared network folder.

**Investigation:**
- Pinged file server : successful, confirmed network connectivity intact
- Attempted access via IP : same "Access Denied" error, ruled out DNS issue
- Checked Active Directory : user had been removed from the `FinanceTeam` security group during a routine quarterly access review
- Confirmed share and NTFS permissions were correctly configured for the group

**Root Cause:** User was inadvertently removed from the Active Directory security group controlling access to the shared folder during a quarterly access review process.

**Resolution:** Re-added user to the `FinanceTeam` AD security group. User logged off and back on to refresh group membership token. Share access restored within 5 minutes.

**Preventive Action:** Flagged access review process for improvement ; recommended a verification step before removing users from production groups. Documented in ServiceNow and escalated recommendation to IT manager.

### Business Impact
Loss of access to shared financial documents prevented the user from completing time-sensitive reporting tasks. Swift resolution within SLA avoided escalation to management. Process improvement recommendation submitted to reduce recurrence risk across the team.

---

*Additional scenarios in progress: Password Reset & Account Lockout | Slow PC Performance | Printer Not Found | VPN Connectivity Failure*
