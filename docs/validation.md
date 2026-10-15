# Network Validation Checklist

## Purpose

This document defines repeatable tests used to verify that the network behaves according to the intended design after configuration changes, maintenance, or troubleshooting.

The goal is to confirm:

- correct addressing
- expected VLAN placement
- required service availability
- inter-VLAN isolation
- administrative access
- Guest isolation
- recovery-path functionality

## Baseline Health Checks

Before testing security policy, confirm that basic network services are functioning.

### pfSense

- pfSense is reachable through the expected administrative path.
- Active VLAN interfaces are up.
- Expected gateway addresses are present.
- DHCP service is running for active client VLANs.
- DNS service is available to permitted internal clients.

### Switching

- The Netgear switch is reachable from the Management VLAN.
- Critical VLAN memberships and PVIDs remain intact.
- The pfSense uplink is operational.
- The UniFi access point uplink is operational.

### UniFi

- The Cloud Key is reachable from the Management VLAN.
- The access point is online.
- Trusted and Guest SSIDs are broadcasting as expected.

### Server

- The Windows Server is reachable through the permitted administrative path.
- The server remains on VLAN 30.

## Management Validation

### Recovery Client Addressing

**Test:** Connect an approved administrative laptop to the Management recovery port.

**Expected result:** The client receives an address in the `10.10.10.0/24` subnet.

### pfSense Management Access

**Test:** From an approved Management recovery client, browse to pfSense at `10.10.10.1`.

**Expected result:** The pfSense web interface is reachable over the permitted management path.

### Switch Management Access

**Test:** From a host on the Management VLAN, attempt to reach the Netgear switch management interface.

**Expected result:** The switch management interface is reachable.

### Management Lateral Isolation

**Test:** From the Management VLAN, attempt to initiate connections toward protected internal networks that are not explicitly required.

**Expected result:** The traffic is blocked.

### Management Internet Access

**Test:** From a Management client, verify that required Internet or cloud services are reachable.

**Expected result:** Required outbound access succeeds.

## Servers Validation

### Server Addressing

**Test:** Confirm that the Windows Server is operating on VLAN 30 with the expected address.

**Expected result:** The server remains on the `10.10.30.0/24` subnet and uses pfSense as its default gateway.

### Server DNS

**Test:** From the Windows Server, perform a DNS lookup using pfSense.

**Expected result:** DNS resolution succeeds.

### Server Internet Access

**Test:** From the Windows Server, access an Internet destination.

**Expected result:** Internet access succeeds.

### Server to Trusted

**Test:** From the Windows Server, attempt to initiate a connection toward the Trusted VLAN.

**Expected result:** The connection is blocked.

### Server to Management

**Test:** From the Windows Server, attempt to initiate a connection toward the Management VLAN.

**Expected result:** The connection is blocked.

### Server to Guest

**Test:** From the Windows Server, attempt to initiate a connection toward the Guest VLAN.

**Expected result:** The connection is blocked.

## Guest Validation

### Guest Addressing

**Test:** Connect a client to the Guest wireless network.

**Expected result:** The client receives an address in the `10.10.40.0/24` subnet with pfSense as its default gateway.

### Guest DNS

**Test:** From a Guest client, perform a DNS lookup using pfSense.

**Expected result:** DNS resolution succeeds.

### Guest Internet Access

**Test:** From a Guest client, browse to an Internet destination.

**Expected result:** Internet access succeeds after any required captive portal interaction.

### Guest to Management

**Test:** From a Guest client, attempt to access general Management resources.

**Expected result:** The connection is blocked.

### Guest to Trusted

**Test:** From a Guest client, attempt to initiate access toward the Trusted VLAN.

**Expected result:** The connection is blocked.

### Guest to Servers

**Test:** From a Guest client, attempt to initiate access toward the Servers VLAN.

**Expected result:** The connection is blocked.

### Guest to Legacy / Recovery

**Test:** From a Guest client, attempt to initiate access toward the Legacy / Recovery network.

**Expected result:** The connection is blocked.

### Guest Client Isolation

**Test:** Connect two wireless clients to the Guest SSID and attempt direct client-to-client communication.

**Expected result:** Direct communication is restricted by UniFi Client Device Isolation.

## Captive Portal Validation

### Portal Presentation

**Test:** Connect a client to the Guest SSID and attempt to browse to a web destination before completing any required portal interaction.

**Expected result:** The client is redirected to the UniFi captive portal as designed.

### Cloud Key Reachability

**Test:** Verify that the Guest client can reach the UniFi Cloud Key on TCP port `8880`.

**Expected result:** The portal dependency is reachable while general Guest access to the Management VLAN remains blocked.

### Post-Authentication Access

**Test:** Complete the captive portal process and browse to an Internet destination.

**Expected result:** Internet access succeeds after the portal flow completes.

### Management Isolation During Portal Use

**Test:** From the Guest client, attempt to access unrelated Management services.

**Expected result:** General Management access remains blocked.

## Recovery Path Validation

### Management Recovery - Port 8

**Test:** Connect an approved administrative laptop to port 8.

**Expected result:** The client receives Management VLAN addressing and can reach the required administrative interfaces.

### Legacy / Recovery - Port 9

**Test:** Connect an approved administrative laptop to port 9.

**Expected result:** The client receives an address in the `192.168.1.0/24` subnet and can reach pfSense through the break-glass path.

### Recovery Independence

**Test:** Confirm that the recovery paths remain usable even if the normal Trusted-to-Management path is unavailable.

**Expected result:** At least one known-good administrative recovery path remains available.

### Recovery Exit

**Test:** After using a recovery path, restore and verify the normal administrative path before disconnecting from recovery access.

**Expected result:** Normal administration is restored before the temporary recovery session ends.

## Post-Change Acceptance Checklist

After a configuration change, confirm the following before considering the change complete:

- pfSense is reachable through the expected administrative path.
- Required VLAN interfaces are up.
- DHCP addressing works on affected client VLANs.
- DNS resolution works where expected.
- Trusted Internet access succeeds.
- Administrative access to Management succeeds only from approved clients.
- Administrative RDP to the Windows Server succeeds where permitted.
- Ordinary Trusted access to protected internal networks remains blocked.
- Server-initiated access toward protected client and management networks remains blocked.
- Guest Internet access succeeds.
- Guest access to protected internal networks remains blocked.
- Guest client isolation remains effective.
- Captive portal functionality operates as expected.
- Management recovery through port 8 remains available.
- Legacy / Recovery access through port 9 remains available.
- Normal administrative access is restored before recovery access is removed.
- Documentation is updated to reflect the validated configuration.
- A current configuration backup is created when appropriate.

