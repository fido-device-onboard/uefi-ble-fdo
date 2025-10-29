# 3. FDO DI

Date: 2025-07-24

## Status

Accepted

## Context

In the context of just-in-time provisioning, FDO Device Initialization (DI) needs to be completed at the time the provisioning occurs as opposed to being completed in a manufacturer environment.  This eliminates supply chain infrastructure and dependencies that would otherwise have to be in place.

This process requires a manufacturer key that is generated and used during DI to create the device certificate chain and it is used for the first FDO voucher extension (assigning a voucher to an Owner key).

In the BLE onboarding flow, the manufacturer key can therefore be generated:

- ephemerally on the device
- on the handset or tablet
- in a device management service

The handset or tablet is already within the security scope boundary because it is being used for initial proof of possession of a device.

Considering the choice between a handset or central device management service, for operational security, simplicity, and scalability, the manufacturer key should be generated in the device management service.  For potential offline scenarios, the key could be generated on the handset or tablet.

If the manufacturer key is generated ephemerally on the device:

- it requires an additional data transfer of the voucher over BLE from the device to the management service
- the device management service will be unable to validate the device certificate chain in the voucher because the key manufacturing is unknown to the device management service
- the device management servvice must instead trust the chain indirectly based on the trust of the handset providing the voucher

### Security Considerations

The device uses a Trust-on-first-Use model, which means that the handset is in scope for the security model and assumed to be trusted.  More broadly, FDO considers all infrastructure involved in DI to be trusted.

As an intermediary, the handset or tablet intentionally receives and provides **all** data between the device and the device management service.  The following scenarios were considered and illustrate how there are no practical mitigations between two endpoints that have no pre-shared context:

|Mitigation|Description|Gap|
|--|--|--|
|Hardware Attestation|The device sends a hardware signed attestation quote and the device management service verifies the quote hierarchy up to the known manufacturer's public root key.|Adversary could use any alternate hardware from that manufacturer as a malicious intermediary.|
|Web PKI|Using public CA infrastructure, the device management service public identity and nonce is validated by the device against the pool of trusted global Certificate Authorities that are used for TLS.|Global CA pool is not unilaterally trustworthy and is subject to update at a much more frequent interval than typical firmware update life cycles.  Not all device management services may use public CA certificates.|

Additionally, if an ephemeral manufacturer key was generated on the device, the derived voucher would be still transferred to the handset or tablet as a trusted endpoint.

The overall provisioning risk of a man-in-the-middle attack during device initialization does not change whether the manufacturer key is generated internal (ephemerally) or external to the device. Expected mitigations include but are not limited to BLE pairing security, user verification of a presented serial number to that printed on the physical chassis, and TLS certificate pinning.

## Decision

Generate and use the manufacturer key off the device.

## Consequences

### Positive

- Significantly reduces data that needs to be exchanged over BLE because the voucher no longer needs to be transferred
- Manufacture chain verification can be performed
- Manufacture key can be managed

### Negative

- Increases key control responsibility to the mobile device and device manager
