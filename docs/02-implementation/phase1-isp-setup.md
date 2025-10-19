# Phase 1: ISP Router DMZ Configuration

**Objective**: Configure ISP router to forward all traffic to pfSense via DMZ while maintaining household network.

**Duration**: 30 minutes  
**Downtime**: 5 minutes (pfSense connection only)  
**Risk Level**: Low

---

## Prerequisites

- [ ] Access to ISP router web interface
- [ ] ISP router admin credentials
- [ ] pfSense WAN MAC address recorded
- [ ] Backup of ISP router configuration

---

## Current State
```
Internet → ISP Router → All devices on 192.168.1.0/24
```

---

## Target State
```
Internet → ISP Router ├─ Household (192.168.1.0/24)
                       └─ DMZ → pfSense WAN (192.168.1.10)
```

---

## Implementation Steps

### Step 1: Gather Information

**Record these details BEFORE making changes:**

1. **ISP Router Info**
```
   Model: ___________________
   Firmware Version: ___________________
   Web Interface: http://192.168.1.1
   Username: ___________________
   Password: ___________________
```

2. **Current Network**
```
   ISP Router LAN IP: 192.168.1.1
   Subnet Mask: 255.255.255.0
   DHCP Range: 192.168.1._____ to 192.168.1._____
```

3. **pfSense WAN MAC Address**
   
   From pfSense (if already connected):
```
   Status → Interfaces → WAN → MAC Address: __:__:__:__:__:__
```
   
   Or from physical label on pfSense device.

---

### Step 2: Backup ISP Router Configuration

**Before making ANY changes:**

1. Log into ISP router: `http://192.168.1.1`

2. Navigate to backup section (varies by model):
   - Look for: "Administration", "System", "Maintenance"
   - Find: "Backup Configuration" or "Save Settings"

3. Download configuration file:
```
   Filename: isp-router-backup-[DATE].cfg
   Save to: Safe location (cloud storage recommended)
```

4. **Verify backup downloaded successfully** before proceeding.

---

### Step 3: Connect pfSense WAN Port

**Physical Connection:**

1. Locate ISP Router Port 4 (or available LAN port)
2. Connect Ethernet cable:
```
   ISP Router Port 4 → pfSense Port 1 (WAN)
```
3. Verify link light on both ends

**Do NOT connect pfSense LAN yet** - we'll do that after confirming WAN works.

---

### Step 4: Reserve Static IP for pfSense

**In ISP Router:**

1. Navigate to DHCP settings:
   - "Advanced → DHCP"
   - Or "LAN Setup → DHCP Reservation"

2. Add DHCP Reservation:
```
   Description: pfSense-WAN
   MAC Address: [pfSense WAN MAC from Step 1]
   Reserved IP: 192.168.1.10
```

3. **Save** settings

4. **Verify** reservation appears in list

---

### Step 5: Enable DMZ

**In ISP Router:**

1. Navigate to DMZ/Firewall section:
   - "Advanced → DMZ"
   - Or "Security → DMZ Host"
   - Or "Firewall → DMZ"

2. Enable DMZ:
```
   DMZ: Enable
   DMZ IP Address: 192.168.1.10
```

3. **Alternative names for this setting:**
   - "Exposed Host"
   - "Default Server"
   - "Port Forwarding → All Ports"

4. **Save** configuration

---

### Step 6: Verify DMZ Configuration

**Check ISP Router:**

1. Navigate to status page
2. Verify DMZ shows:
```
   DMZ Enabled: Yes
   DMZ IP: 192.168.1.10
```

3. Check connected devices list
4. Confirm pfSense appears with IP 192.168.1.10

---

### Step 7: Test from Household Device

**From a device still on household network (192.168.1.x):**
```cmd
# Should still have internet
ping 8.8.8.8

# Should resolve DNS
ping google.com

# Should reach ISP router
ping 192.168.1.1
```

**If any fail, STOP and troubleshoot before continuing.**

---

### Step 8: Verify pfSense Receives Public Traffic

This step will be completed in Phase 2 after pfSense is configured.

**For now, verify:**
- [ ] pfSense WAN port has link light
- [ ] ISP router shows pfSense at 192.168.1.10
- [ ] DMZ is enabled and pointing to 192.168.1.10

---

## Validation Checklist

### ISP Router Side
- [ ] Configuration backed up
- [ ] DHCP reservation created for 192.168.1.10
- [ ] DMZ enabled and pointing to 192.168.1.10
- [ ] Household devices still have internet access
- [ ] Household devices can still reach router (192.168.1.1)

### pfSense Side
- [ ] WAN port connected to ISP router
- [ ] Link light showing on both ends
- [ ] pfSense appears in ISP router device list

---

## Rollback Procedure

**If something goes wrong:**

### Option 1: Disable DMZ
1. Log into ISP router
2. Navigate to DMZ settings
3. Disable DMZ
4. Save configuration
5. Household network returns to normal

### Option 2: Restore Backup
1. Log into ISP router
2. Navigate to restore section
3. Upload backup file from Step 2
4. Reboot router
5. Everything returns to previous state

---

## Network Diagram After Phase 1
Internet (2 Gbps)
                       │
                       ↓┌──────────────────────────────────────────────────────┐
│              ISP Router (192.168.1.1)                │
│                                                       │
│  LAN Network: 192.168.1.0/24                         │
│  ├─ Port 1: Household Device (TV)                    │
│  ├─ Port 2: Household Device (Phone)                 │
│  ├─ Port 3: Household WiFi                           │
│  └─ Port 4: DMZ to pfSense WAN (192.168.1.10) ◄──┐  │
│                                                    │  │
│  DMZ Enabled: Yes                                  │  │
│  DMZ IP: 192.168.1.10 (pfSense)                   │  │
└────────────────────────────────────────────────────┼──┘
│
All incoming traffic forwarded  │
↓
┌──────────────────────────────┐
│  pfSense Firewall (N355)     │
│                              │
│  Port 1 (WAN):               │
│    IP: 192.168.1.10 (DHCP)   │
│    Gateway: 192.168.1.1          │
│                              │
│  Port 2 (LAN):               │
│    Not configured yet        │
│    (Next phase)              │
└──────────────────────────────┘
---

## Common Issues & Solutions

### Issue 1: Can't Find DMZ Setting

**Symptom**: ISP router doesn't have obvious "DMZ" option

**Solutions**:
1. Check under different names:
   - "Exposed Host"
   - "Default Server"
   - "Default DMZ Server"
   - "Gaming Mode"
   - "Port Forwarding → All Ports to Single IP"

2. Consult ISP router manual:
```
   Google: "[Router Model] DMZ configuration"
```

3. Alternative: Port forward all ports
   - Forward TCP/UDP 1-65535 to 192.168.1.10
   - Less ideal but functional

---

### Issue 2: pfSense Not Getting 192.168.1.10

**Symptom**: pfSense shows different IP or no IP

**Check**:
1. DHCP reservation uses correct MAC address
2. pfSense WAN is set to DHCP (we'll configure in Phase 2)
3. Cable is connected to correct ports
4. Try rebooting pfSense

**Debug**:
```bash
# From pfSense console
ifconfig re0  # Check WAN interface
# Should show: inet 192.168.1.10
```

---

### Issue 3: Household Devices Lost Internet

**Symptom**: Devices on 192.168.1.x can't reach internet

**Cause**: DMZ misconfigured or ISP router issue

**Fix**:
1. Disable DMZ temporarily
2. Verify household devices regain internet
3. Re-enable DMZ carefully
4. Check DMZ IP is exactly 192.168.1.10

**Verify**:
```cmd
# From household device
ping 192.168.1.1  # ISP router
ping 8.8.8.8      # Internet
```

---

### Issue 4: DMZ Keeps Resetting

**Symptom**: DMZ setting disappears after router reboot

**Cause**: Firmware issue or setting not saved

**Solutions**:
1. Ensure you clicked "Save" or "Apply"
2. Reboot router and verify setting persists
3. Update router firmware if available
4. Contact ISP support if issue continues

---

## Security Notes

### What DMZ Does
- ✅ Forwards ALL incoming internet traffic to pfSense
- ✅ Allows pfSense to handle firewall rules
- ✅ Keeps household network separate (192.168.1.x)

### What DMZ Does NOT Do
- ❌ Does NOT expose household devices
- ❌ Does NOT bypass ISP router completely
- ❌ Does NOT give pfSense public IP directly

### Future Considerations
- Consider upgrading to true bridge mode after homelab is stable
- This would give pfSense public IP directly
- Better for advanced features (port forwarding, VPNs)
- But requires separate solution for household network

---

## Testing Criteria

**Phase 1 is complete when:**

- [x] ISP router configuration backed up
- [x] DMZ enabled and pointing to 192.168.1.10
- [x] pfSense WAN connected and receiving IP
- [x] Household network still functional
- [x] No internet disruption to household

**DO NOT PROCEED** to Phase 2 until ALL checkboxes are complete.

---

## Documentation Requirements

### Screenshot Checklist

Take screenshots of:
- [ ] ISP router DHCP reservation page
- [ ] ISP router DMZ configuration page
- [ ] ISP router device list showing pfSense
- [ ] ISP router status page

**Save screenshots to:**
```
homelab-network/screenshots/phase1/
├── 01-isp-router-dhcp-reservation.png
├── 02-isp-router-dmz-config.png
├── 03-isp-router-device-list.png
└── 04-isp-router-status.png
```

### Configuration Notes

**Record actual values:**
```markdown
## Phase 1 Completion Notes

**Date Completed**: [Date and Time]

**ISP Router Details**:
- Model: _______________
- Firmware: _______________
- DMZ IP: 192.168.1.10 ✓

**pfSense WAN**:
- MAC Address: __:__:__:__:__:__
- IP Address: 192.168.1.10 ✓
- Gateway: 192.168.1.1 ✓

**Issues Encountered**:
- [List any problems and how you solved them]

**Time Taken**: ___ minutes

**Household Impact**: None / Minimal / [Describe]

**Next Phase**: Phase 2 - pfSense Basic Configuration
```

---

## GitHub Commit

**After Phase 1 completion:**
```bash
cd homelab-network

# Add Phase 1 documentation
git add docs/02-implementation/phase1-isp-setup.md

# Add screenshots
git add screenshots/phase1/

# Commit with descriptive message
git commit -m "Phase 1 Complete: ISP DMZ Configuration

- Configured ISP router DMZ pointing to 192.168.1.10
- Created DHCP reservation for pfSense WAN
- Verified household network remains functional
- Backed up ISP router configuration

Screenshots and validation included."

# Push to GitHub
git push origin main
```

---

## What's Next

**Phase 2: pfSense Basic Configuration**
- Initial pfSense setup wizard
- Configure WAN interface (verify DMZ)
- Configure first LAN interface (10.0.20.1)
- Test basic connectivity
- Access pfSense web GUI

**Estimated Time**: 45 minutes  
**Prerequisite**: Phase 1 complete ✓

---

**Phase 1 Status**: ⬜ Not Started | 🔄 In Progress | ✅ Complete

**Completed By**: _______________  
**Date**: _______________  
**Sign-off**: _______________

---