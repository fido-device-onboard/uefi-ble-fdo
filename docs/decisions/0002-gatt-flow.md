# 2. GATT flow

Date: 2025-07-24

## Status

Accepted

## Context

Bluetooth LE (BLE) Peripherals typically implement a GATT server, whereas BLE Centrals, such as a mobile phone, typically implement a GATT client to connect to and retrieve values, such as "beats per minute" in a heart rate monitor.

It is more common to find BLE stack implementations that enable a GAP Central and GATT Client for use cases such as connecting and using a BLE keyboard.

Under perfect conditions, it would be desirable to use both GATT Client and GATT Servers because the proposed flows would ideally use both.

The Bluetooth protocol in the UEFI standard does not include interfaces to configure or use a GATT server.  Adding such support will require significant changes including but not limited to:

- Adding services
- Adding characteristics and handlers/callbacks

Addressing the above gaps would result in a significantly larger upstream pull request into EDK2 because it introduces a new UEFI protocol as opposed to using an existing protocol.  Submitting such a PR would also be introducing a driver that consumes a protocol that is not available in open source.

For these reasons it is strongly desired to use the existing GATT client available in existing implementations.

In terms of functional viability, a test GAP Peripheral + GATT Client (running on Linux) and test GAP Central + GATT Server (mobile phone) were built and tested to confirm technical interoperability and behavior.

## Decision

Reverse the flow and use a GAP Peripheral and GATT Client in UEFI, with a GAP Central and GATT Server implemented in remote clients (e.g. phone, tablet, etc.)

## Consequences

Instead of building reference code, an actual library for mobiles device ecosystems, such as iOS and Android, will likely be required to abstract the Bluetooth service.  Notably this may already be the case because the pre-requisite developer knowledge of both the Bluetooth specification and implementation architecture is much more substantial than more trivial implementations like a REST API.

### Positive

- Decreases the complexity in the UEFI implementation, which is a significant consideration because the tooling and ecosystem is richer for mobile devices than UEFI.

- Potentially smaller UEFI binary size.

### Negative

- Increases the implementation complexity in native app implementations and will likely require additional reference code because this is not the typical flow in the mobile device Bluetooth framework documentation.

- Unable to leverage the Web Bluetooth library, so any remote device implementations will require a native app implementation (e.g. iOS, Android, etc.).
