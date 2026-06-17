# Security-Architecture-Cellular-Authentication-AKA-in-Modern-Connected-Vehicles
This project demonstrates a foundational implementation of the 3GPP Authentication and Key Agreement (AKA) protocol—originally developed for legacy telecommunications—recontextualized for modern Automotive Cybersecurity and Intelligent Transportation Systems (ITS).

The following diagram represents the sequence of the events when a user (UE) connects with a eNodeB i.e. 'Evolved Node B', an element in E-UTRA which is the core hardware base station in a 4G LTE network. It acts as the direct bridge connecting your smartphone (User Equipment) to the mobile operator's core network and handles both wireless radio transmissions and higher-layer networking tasks. As ahandshake mechanism there are set of keys genertated and exchanged for the authentication and verification. 

<img width="2960" height="1595" alt="image" src="https://github.com/user-attachments/assets/ab9a31ea-b456-4f29-9251-1986128dc5a1" />


In contect of security expecially with hardware backed interoperabaility, Automotive cybersecurity resembles the base concept from 3GPP as explained above.

As vehicles transition into software-defined, hyper-connected platforms, they increasingly rely on built-in Telematic Control Units (TCUs) and eSIMs to communicate with cellular networks. This code simulates the precise cryptographic engine used to authenticate a vehicle to the cloud, prevent network spoofing, and derive the session keys needed to protect safety-critical vehicle telematics.The accompanying flow diagram maps out how raw vehicular identifiers and network challenges are processed through a multi-stage cryptographic pipeline to secure both edge-to-cloud infrastructure and in-vehicle domain isolation.

---
```text
       [ Raw Inputs: Secret Key (K), RAND, SQN, AMF ]
                             │
       ┌─────────────────────┴─────────────────────┐
       ▼                                           ▼
┌──────────────┐                            ┌──────────────┐
│ F1 Function  │                            │ F2 Function  │
│  (MAC_F1)    │                            │  (XRES_F2)   │
└──────┬───────┘                            └──────┬───────┘
       │                                           │
       ▼                                           ▼
    [ MAC ]                                     [ XRES ]
 (Truncated 8B)                              (Truncated 16B)
       │                                           │
       ▼                                           │
┌────────────────────────────────────────┐         │
│ Assemble AUTN Token                    │         │
│ (AUTN = SQN || AMF || MAC)             │         │
└──────┬─────────────────────────────────┘         │
       │                                           │
       └─────────────────────┬─────────────────────┘
                             │
                             ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ F3 Function  │      │ F4 Function  │      │  KASME Gen   │
│   (CK_F3)    │      │   (IK_F4)    │      │  Derivation  │
└──────┬───────┘      └──────┬───────┘      └──────┬───────┘
       │                     │                     │
       ▼                     ▼                     ▼
    [ CK ]                [ IK ]               [ KASME ]
(Truncated 16B)       (Truncated 16B)              │
       │                     │                     ▼
       │                     │              ┌──────────────┐
       │                     │              │  KENB Gen    │
       │                     │              │  Derivation  │
       │                     │              └──────┬───────┘
       │                     │                     │
       │                     │                     ▼
       │                     │                  [ KENB ]
       │                     │                     │
       └──────────────┬──────┴─────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────┐
│             Assemble Final Vector V                      │
│     (V = RAND || XRES || CK || IK || AUTN)               │
└──────────────────────────────────────────────────────────┘
```


---

# Mapping Telecom Cryptography to Automotive Cybersecurity

The execution logs of the 3GPP Authentication and Key Agreement (AKA) protocol capture a cryptographic sequence that maps directly to critical automotive security primitives across four distinct layers.

---

### 1. Preventing Rogue Infrastructure & Spoofing (`MAC_F1` & `AUTN`)

* **The Telecom Concept:**  
  The `F1` function calculates a Message Authentication Code (`MAC`) packaged inside an Authentication Token (`AUTN`) to enforce mutual network authentication.
* **The Automotive Application:**  
  This serves as the primary defense against **Rogue Base Stations** (e.g., IMSI Catchers or Stingrays). In connected vehicles, an attacker spoofing a cell tower can push malicious Over-the-Air (OTA) firmware updates or transmit fraudulent Vehicle-to-Everything (V2X) safety commands (like emergency braking). The `AUTN` ensures the vehicle's Telematic Control Unit (TCU) cryptographically verifies that the cell tower or Roadside Unit (RSU) is authentic *before* processing or executing external instructions.

---

### 2. Vehicle Identity & Zero-Trust Access (`XRES_F2`)

* **The Telecom Concept:**  
  The `F2` function calculates the Expected Response (`XRES`), cryptographically validating the subscriber's unique identity.
* **The Automotive Application:**  
  This framework provides a foundation for **Zero-Trust Network Access (ZTNA)** for vehicles interacting with cloud ecosystems. It applies to fleet management portals, electric vehicle (EV) charging station payment systems, and Original Equipment Manufacturer (OEM) backends. By requiring the vehicle's hardware-bound eSIM to solve the cryptographic challenge (`XRES`) using a securely stored master key, the backend network guarantees that the interacting node is a legitimate, uncompromised vehicle asset and not a cloned software emulator.

---

### 3. Over-the-Air Confidentiality & Message Integrity (`CK_F3` & `IK_F4`)

* **The Telecom Concept:**  
  Parallel derivation of the Cipher Key (`CK`) for data privacy and the Integrity Key (`IK`) to prevent signal tampering.
* **The Automotive Application:**  
  * **`CK` (Privacy):** Encrypts remote vehicle commands (such as remote start, locking systems, and GPS telemetry logs) so that malicious actors cannot eavesdrop on or sniff sensitive operational data over wireless frequencies.
  * **`IK` (Integrity):** Cryptographically signs telemetry bundles and vehicle diagnostic messages. This mitigates **Man-in-the-Middle (MitM) alterations**, guaranteeing that safety-critical sensor data transmitted to the cloud cannot be manipulated or injected mid-transit.

---

### 4. Defense-in-Depth & Domain Isolation ($K_{ASME}$ & $K_{eNB}$)

* **The Telecom Concept:**  
  A hierarchical derivation where a master key ($K_{ASME}$) generates a localized, transient session key ($K_{eNB}$) dedicated entirely to a single physical cell tower.
* **The Automotive Application:**  
  This aligns directly with the architectural requirements for **In-Vehicle Domain Isolation** and **Blast Radius Mitigation**. By leveraging a secure root key to generate localized, temporary session keys for specific edge infrastructure nodes, the architecture enforces strict isolation. If an adversary physically breaches a single cellular base station or localized RSU, the compromise remains strictly contained. The root vehicle identities remain secure, preventing a localized perimeter breach from scaling into a catastrophic, fleet-wide cyberattack.

---

> ### 📝 Licensing Note
> This documentation and architectural mapping are licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license. You are free to share and adapt this material provided appropriate credit is given.
