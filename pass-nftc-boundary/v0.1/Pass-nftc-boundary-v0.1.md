

# Pass–NFTC Boundary Specification (v0.1)

**Author:** carl0zen  
**Publisher:** Flowhash Protocol  
**Date:** 2025-10-05  
**Status:** Draft

---

## 1. Purpose

This specification defines the minimal interface between **Flowhash Pass** (identity & reputation) and **NFTC** (event attestation) protocols.  
The goal is to ensure that any NFTC event can reference and verify a valid Pass identity, enabling **actor–event binding** in logistics operations and beyond.

By cleanly separating identity (Pass) from event attestations (NFTC), this boundary allows both protocols to evolve independently while remaining interoperable through a shared verification contract.

---

## 2. Core Concepts

### 2.1 Flowhash Pass

A **Flowhash Pass** is a W3C Verifiable Credential (VC) issued to an operator (e.g. truck driver, gate guard, weighmaster) by a trusted authority.  
It represents the actor’s cryptographic identity and reputation status.

**Minimum fields:**

- `id`: A unique DID or URI identifying the Pass  
- `publicKey`: The operator’s wallet public key used to sign events  
- `role`: Operator role (e.g. "Operator", "Weighmaster")  
- `issuer`: DID of the issuing authority  
- `expiry`: Expiration timestamp

Additional claims (e.g. training, certifications, incident records) can be layered later but are not required for boundary compliance.

Passes are **revocable** via a revocation registry maintained by the issuing authority.

---

### 2.2 NFTC Event

An **NFTC Event** is a signed attestation representing a real-world logistics event (e.g. truck entry, weighing, loading, unloading).  
It is structured JSON that is cryptographically signed and anchored on Flowhash.

The key property is that **every NFTC event must be signed by the wallet key associated with a valid Flowhash Pass**.  
This creates a strong, verifiable link between the actor and the event.

---

## 3. Interface Specification

### 3.1 Actor Reference

Every NFTC event must include the following `actor` object:

```json
"actor": {
  "pass_id": "did:flowhash:operator:12345",
  "wallet_address": "0xABCDEF1234567890...",
  "signature": "0xSIGNATURE..."
}
```

	•	pass_id: Reference to the Flowhash Pass (DID or URI)
	•	wallet_address: The public key of the actor (must match Pass)
	•	signature: The operator’s signature over the event payload hash

⸻

3.2 Verification Steps

To validate an NFTC event:
	1.	Resolve Pass
Retrieve the Flowhash Pass by pass_id.
	2.	Check Pass Validity
	•	Verify the Pass signature issued by the issuer.
	•	Check the revocation registry and expiry date.
	3.	Check Actor Binding
Ensure wallet_address in the event matches the publicKey in the Pass.
	4.	Verify Event Signature
	•	Reconstruct the event hash deterministically.
	•	Verify the signature with the wallet_address.
	5.	Anchor Check
Confirm the NFTC hash is properly anchored on Flowhash (or compatible ledger).

Only if all checks pass is the NFTC event considered valid.

⸻

4. Security Considerations
	•	Lost or compromised wallets must trigger immediate Pass revocation.
	•	Events signed by revoked Passes are invalid.
	•	Selective disclosure is supported in future versions: verifiers may check Pass validity without revealing full credential data.
	•	Revocation registries should be auditable and time-stamped.

⸻

5. Future Extensions

Planned extensions beyond v0.1 include:
	•	Multi-sig Passes for teams or shared roles
	•	Hardware key / biometric binding for higher security operations
	•	Cross-domain Pass usage (e.g. hospitality, creative industries, social good)
	•	Delegation support, allowing temporary sub-keys linked to a master Pass

⸻

6. License

This specification is released under the MIT License.

⸻

7. Changelog
	•	v0.1 (2025-10-05): Initial draft of the Pass–NFTC interface boundary specification.

---
