# Troubleshooting Lessons

## Purpose

This document records significant troubleshooting incidents, the evidence used to isolate each problem, the root cause, the resolution, and the lesson learned.

The goal is to preserve diagnostic knowledge without mixing historical failures into current-state configuration documents.

## Troubleshooting Methodology

Troubleshooting should proceed from the lowest relevant layer upward and should rely on observed evidence rather than assumptions.

A useful sequence is:

1. Confirm physical link state.
2. Confirm VLAN placement and Layer 2 transport.
3. Confirm IP addressing and default gateway.
4. Confirm DNS behavior.
5. Confirm routing.
6. Review firewall policy and logs.
7. Use packet capture when the failure point remains unclear.
8. Test one change at a time.
9. Re-test the expected behavior after each change.
10. Preserve a known-good recovery path while troubleshooting.

## Incident: Guest VLAN Had DHCP but No Reliable Internet

### Symptoms

Guest clients received valid VLAN 40 addresses but did not have reliable Internet access.

### Evidence

- DHCP addressing succeeded.
- ARP behavior was visible.
- DNS-related traffic was observed.
- The Guest VLAN path was confirmed with a wired diagnostic test.
- The wireless access point and VLAN tagging were therefore less likely to be the root cause.

### Root Cause

A temporary Guest firewall rule used the wrong Protocol value instead of permitting the intended IPv4 traffic.

The rule therefore did not match the traffic as expected.

### Resolution

The Guest firewall rule was corrected so that the intended IPv4 traffic matched properly.

After the rule was corrected, Guest connectivity behaved as expected.

### Lesson

Successful DHCP does not prove that general IP forwarding or firewall policy is correct.

When addressing works but higher-layer connectivity fails, firewall rule matching should still be verified carefully.

## Incident: Server Internet Worked but DNS Failed

### Symptoms

The Windows Server on VLAN 30 could reach Internet destinations by IP address, but DNS resolution failed.

### Evidence

- The server had valid VLAN 30 addressing.
- The default gateway was reachable.
- Internet connectivity by IP address succeeded.
- DNS queries to pfSense did not succeed.
- The broader Servers-to-internal-network block matched traffic before the required DNS exception.

### Root Cause

The firewall alias used to represent protected internal networks included the Servers subnet and the pfSense interface address used for DNS.

Because the broader internal-network block appeared before an explicit DNS allow rule, DNS traffic from the server was blocked.

### Resolution

An explicit DNS allow rule was placed above the broader internal-network block.

The rule permitted the Servers VLAN to reach pfSense DNS service on port `53`.

After the exception was moved above the block, DNS resolution succeeded.

### Lesson

Broad network aliases can unintentionally include infrastructure services required by the source network.

Specific service exceptions should be placed above broader block rules when first-match firewall evaluation is used.

## Incident: Management DNS Used Port 853 Instead of 53

### Symptoms

Management devices experienced DNS failures even though the surrounding VLAN and routing configuration appeared correct.

### Evidence

- Management clients had valid VLAN 10 addressing.
- pfSense was reachable.
- Firewall logs showed the DNS traffic being blocked.
- Inspection of the Management DNS rule revealed that the destination port was configured as `853` instead of `53`.

### Root Cause

The firewall rule intended to permit DNS to pfSense used the wrong destination port.

Port `53` is used for standard DNS over UDP or TCP.

Port `853` is commonly associated with DNS over TLS and did not match the intended traffic.

### Resolution

The Management DNS rule was corrected to permit traffic to pfSense on port `53`.

After the rule was corrected, DNS resolution from the Management VLAN succeeded.

### Lesson

When a rule appears logically correct but traffic still fails, verify the exact protocol and port values rather than assuming the selected service is correct.

Firewall logs can reveal whether traffic is reaching the expected rule or being blocked unexpectedly.

## Incident: Trusted Clients Could Initiate Toward Guest

### Symptoms

Trusted clients were able to initiate traffic toward the Guest VLAN even though the intended policy was to keep the Guest network isolated from internal client networks.

### Evidence

- Trusted clients had valid VLAN 20 addressing.
- Guest clients had valid VLAN 40 addressing.
- Inter-VLAN routing through pfSense was functioning.
- Review of the Trusted interface rules showed that no explicit Trusted-to-Guest block existed before the broader egress rule.

### Root Cause

The Trusted firewall policy blocked access to Management, Servers, and Legacy / Recovery, but the Guest VLAN had not been included in the protected destinations handled before general egress.

As a result, Trusted-to-Guest traffic could match the broader remaining-destinations allow rule.

### Resolution

An explicit Trusted-to-Guest block was added above the final broad egress rule.

After the rule was added, Trusted clients could no longer initiate connections toward the Guest VLAN.

### Lesson

A broad final allow rule is only as safe as the exclusions placed above it.

When new internal segments are added or policy changes, review whether those networks are included in every relevant internal-block path.

## Incident: Guest Captive Portal Failed to Redirect

### Symptoms

Guest clients connected to the Guest SSID and received valid VLAN 40 addressing, but the captive portal did not load or redirect as expected.

### Evidence

- Guest clients received valid DHCP leases.
- Guest Internet access behavior indicated that basic VLAN transport and routing were functioning.
- Firewall logs showed Guest traffic being blocked when attempting to reach the UniFi Cloud Key.
- The blocked destination was the Cloud Key on TCP port `8880`.

### Root Cause

The Guest firewall policy blocked access to protected internal networks, including the Management VLAN where the Cloud Key resides.

The captive portal depends on Guest clients being able to reach the Cloud Key on TCP port `8880`.

Because no exception existed above the broader internal-network block, the portal dependency was blocked.

### Resolution

A narrow firewall exception was added to permit:

`Guest VLAN -> UniFi Cloud Key -> TCP 8880`

The exception was placed above the broader Guest-to-internal-network block.

After the rule was added, captive portal functionality worked as expected.

### Lesson

Application dependencies can require narrowly defined exceptions through otherwise correct segmentation policy.

Firewall logs are especially useful for discovering these hidden dependencies because they show which destination and service are being blocked.

## Diagnostic Techniques

Several troubleshooting methods proved useful across multiple incidents.

### Firewall Logs

pfSense firewall logs help determine whether traffic reaches the firewall and whether it is passed or blocked.

Use firewall logs when:

- addressing is correct but routed traffic still fails
- a rule appears correct but may not be matching
- an unexpected destination or service may be involved
- you need to confirm which interface sees the traffic

Firewall logs are especially useful for validating first-match behavior and discovering missing exceptions.

### Packet Capture

Packet capture helps determine whether traffic is actually present on an interface and how far it travels through the network.

Use packet capture when:

- DHCP or DNS behavior is unclear
- traffic appears to disappear between layers
- firewall logs do not provide enough detail
- you need to distinguish between Layer 2, Layer 3, and application problems

A packet capture showing DHCP or ARP traffic proves only that those protocols are functioning. It does not prove that general forwarding, DNS, or firewall policy is correct.

### Wired VLAN Test Ports

A temporary wired access port can isolate wireless problems from routing and firewall problems.

For example, placing a wired client directly on VLAN 40 through the Guest diagnostic port helps answer:

```text
Does VLAN 40 work without involving the access point?
```

If the wired test succeeds while wireless clients fail, the problem is more likely to involve the access point, SSID configuration, or VLAN tagging.

If the wired test also fails, the problem is more likely to involve switching, pfSense, addressing, routing, or firewall policy.

### Layer-by-Layer Elimination

Troubleshooting should move through the network stack methodically.

A useful order is:

1. Physical link
2. VLAN placement and Layer 2 transport
3. IP addressing
4. Default gateway
5. DNS
6. Routing
7. Firewall policy
8. Application behavior

Avoid changing multiple layers at once unless the failure is already well understood.

### Known-Good Baselines

Preserve working configurations and recovery paths while troubleshooting.

Compare unexpected behavior against a known-good baseline rather than relying on memory alone.

When a change produces uncertainty, return to the last validated state before adding more changes.

