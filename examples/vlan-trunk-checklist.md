---
title: "A quick checklist for VLAN trunks"
date: 2026-11-16
draft: true
description: "A short example of a Markdown guide, from checking the trunk state to confirming which VLANs are allowed."
category: "Switching"
readingTime: "3 min read"
image: "/images/vlans-hero.png"
---

When two switches should carry several VLANs across one link, check the trunk itself before changing the VLAN configuration.

## Check the link

On each switch, inspect the interface and confirm that it is up and operating as a trunk:

```text
show interfaces trunk
show interfaces status
```

## Check the VLAN list

Compare the allowed VLANs on both ends. A VLAN can exist on both switches and still fail to cross the link if it is excluded from the trunk.

```text
show vlan brief
show interfaces trunk
```

## Check the native VLAN

Both ends should agree on the native VLAN. A mismatch can put untagged traffic into different VLANs on either side.

For the full explanation and a worked topology, read [VLANs explained](https://labfabric.pro/learn/vlans).
