# Phase 2 Completion Report

**Date**: 10/19/2025
**Duration**: 30 mins
**Status**: ✅ Complete

## Configuration Summary

### pfSense Device
- **Hostname**: pfsense.lab.arpa
- **Version**: pfSense CE 2.7.x
- **Hardware**: Glovary i3 N355 (Intel N355, 8GB RAM)

### WAN Interface (igc0 / Port 1)
- **Type**: DHCP
- **IP Address**: 192.168.0.141/24 ✓
- **Gateway**: 192.168.0.1 (ISP router)
- **DNS**: 1.1.1.1, 8.8.8.8
- **Status**: Online ✓

### LAN Interface (igc1 / Port 2)
- **Type**: Static
- **IP Address**: 10.0.20.1/24 ✓
- **DHCP Server**: Enabled
- **DHCP Range**: 10.0.20.100 - 10.0.20.250
- **Status**: Online ✓

## Testing Results

### From pfSense Console
- Ping 8.8.8.8: ✅ Success (15ms avg)
- Ping google.com: ✅ Success (DNS resolving)

### From Client Computer
- Received DHCP IP: 10.0.20.100 ✅
- Ping 10.0.20.1 (gateway): ✅ Success (<1ms)
- Ping 8.8.8.8 (internet): ✅ Success (15ms)
- Ping google.com (DNS): ✅ Success (20ms)

### Web GUI Access
- URL: https://10.0.20.1 ✅
- Login: Working with new password ✅
- Dashboard: All interfaces UP ✅

## Security

- ✅ Admin password changed from default
- ✅ HTTPS enabled (self-signed cert)
- ✅ Default firewall rules active
- ✅ NAT configured (automatic)

## Issues Encountered
N/A

## Screenshots
- ✅  pfSense dashboard
- ✅ Interface overview
- ✅ Ping test results
- ✅ Computer IP configuration
- ✅ Gateway status

## Network Topology After Phase 2
```
Internet → ISP Router (DMZ) → pfSense WAN (192.168.0.141)
                                  ↓
                            pfSense LAN (10.0.20.1)
                                  ↓
                            Your Computer (10.0.20.100)
```

## Next Steps
- Phase 3: VLAN Configuration
- Create VLANs 20, 30, 40, 50, 60, 61, 62, 70
- Configure VLAN interfaces
- Estimated time: 1 hour

**Completed by**: Techin B
**Status**: Phase 2 Complete ✅