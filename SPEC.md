# PWSS/1.0 Protocol Specification

**PGP Web Static Secure (PWSS)** *Version: 1.0.0-Release* *Date: May 2026*

---

## 1. Philosophy & Design Principles

PWSS is a transport-agnostic, client-centric, secure web protocol designed to completely replace traditional centralized Domain Name Systems (DNS), Certificate Authorities (CAs), and rigid corporate top-level domains (TLDs). 

PWSS adheres strictly to the following architectural pillars:
* **Cryptographic Sovereignty:** A domain is not a piece of rented real estate; it is a self-signed cryptographic identity bound to a Pretty Good Privacy (PGP) keypair.
* **Client-Driven Routing:** The end-user retains absolute, final authority over how names map to network resources on their machine. 
* **GitOps Discovery:** Global discovery indexation is maintained via decentralized, verifiable, Git-backed history logs, eliminating hidden state changes or arbitrary domain hijacking.
* **Terminal Native:** The wire format and operational tooling are optimized for low-overhead parsing, shell orchestrators (`bash`/`zsh`), and text user interfaces (TUIs).

---

## 2. Global URI Scheme

PWSS network addresses are specified via a deterministic, hierarchical URI format:

```text
pwss://[authority]/[site-path]
```

* **`authority`**: A human-readable, flat alphanumeric lowercase string representing the primary namespace identity (e.g., `zai1208`, `bob`).
* **`site-path`**: An optional forward-slash delimited string routing to specific site paths or applications hosted within that authority's namespace (e.g., `projects/3pedal-ev`). If omitted, the client router implicitly defaults to fetching the root index content definition.

---

## 3. Core Cryptographic Data Structures

### 3.1. The Site Manifest (`manifest.json`)
Every individual site or namespace application must serve a cryptographically signed document known as the manifest. This document dictates the physical location of assets and maps human-readable paths to static file resources.

```json
{
  "pwss_version": "1.0",
  "authority": "zai1208",
  "fingerprint": "1D621CE3F6D554DC3E6E2551B1437E7DFC87A9B4",
  "routes": {
    "/": {
      "type": "text/markdown",
      "src": "[https://zain-khan.dev/pwss/index.md](https://zain-khan.dev/pwss/index.md)"
    },
    "/projects/3pedal-ev": {
      "type": "text/markdown",
      "src": "[https://zain-khan.dev/pwss/3pedal.md](https://zain-khan.dev/pwss/3pedal.md)"
    }
  }
}
```

> **Invariant:** The manifest payload must be wrapped in a cleartext PGP ASCII-armored signature (`.asc`) verified against the `fingerprint` declared in the authoritative routing ledger.

### 3.2. The Global Registry Ledger (`registry.json`)
The global directory operates as a flat, signed index ledger mapping authorities to either direct standalone static sites or nested subnets. To mitigate rollback attacks (where a malicious actor serves a valid but out-of-date historical copy of the registry), the ledger enforces sequential tracking.

```json
{
  "sequence": 1024,
  "prev_registry_hash": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2",
  "timestamp": 1779268774,
  "entries": {
    "zai1208": {
      "type": "site",
      "fingerprint": "1D621CE3F6D554DC3E6E2551B1437E7DFC87A9B4",
      "url": "[https://zain-khan.dev/pwss/manifest.json.asc](https://zain-khan.dev/pwss/manifest.json.asc)"
    },
    "bob": {
      "type": "registry_pointer",
      "fingerprint": "8F3B9A2C5D1E4F6A7B8C9D0E1F2A3B4C5D6E7F8G",
      "url": "[https://bob.dev/pwss/registry.json.asc](https://bob.dev/pwss/registry.json.asc)",
      "owner": {
        "name": "Bob Jones",
        "email": "bob@bob.dev"
      }
    }
  }
}
```

---

## 4. Federated Subnets (Nested Registries)

Subnets allow individual nodes to scale horizontally without altering the root directory configuration. A root registry entry of `type: "registry_pointer"` delegates an entire downstream branch to an independent operator.

### 4.1. The Subnet Trust Handshake (TOFU)
When the browser resolver executes a path inside an unfamiliar subnet, it must freeze execution and drop a secure terminal interface Trust Prompt. Trust cannot be inherited blindly down the tree; it must be confirmed on first use (Trust On First Use).

```text
┌────────────────────────────────────────────────────────────────────────┐
│  [PWSS] NEW FEDERATED REGISTRY DETECTED                                │
│                                                                        │
│  A peer registry is required to resolve this handle namespace.         │
│                                                                        │
│  Registry URL:  [https://bob.dev/pwss/registry.json.asc](https://bob.dev/pwss/registry.json.asc)                 │
│  Owner Name:    Bob Jones                                              │
│  Owner Email:   bob@bob.dev                                            │
│  PGP Key FPR:   0x8F3B9A2C5D1E4F6A7B8C9D0E1F2A3B4C5D6E7F8G             │
│                                                                        │
│  Do you want to permanently trust and pin this registry? [y/N]         │
└────────────────────────────────────────────────────────────────────────┘
```

Once accepted, the PGP key fingerprint is cached inside the client database (`~/.config/pwss/trusted_keys`), protecting all downstream requests from future tampering or mid-route intercepts.

---

## 5. Client Resolution Engine Blueprint

The browser engine (e.g., `pwss-fetch-https`) MUST resolve a requested URI strictly according to the following 3-tier deterministic pipeline.

```text
       [ User inputs: pwss://target ]
                     │
                     ▼
       (1. Check Local Petnames File)  ──[Found]──> Resolve Target Immediately
                     │
                  [Miss]
                     │
                     ▼
       (2. Check Root registry.json)   ──[Found]──> Fetch & Render Manifest
                     │
                  [Miss]
                     │
                     ▼
       (3. Scan All Pinned Subnets)    ──[Found]──> Run TOFU -> Verify -> Render
                     │
               [Collision]
                     │
                     ▼
       (Throw Ambiguity Halt Error)
```

### 5.1. Resolution Hierarchy Specification

| Tier | Source Location | Resolution Context | Description | Override Authority |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `~/.config/pwss/petnames.conf` | **Local Client** | Private aliases, macros, and bookmarks. | Ultimate Client Control |
| **2** | `/var/lib/pwss/registry.json` | **Global Index** | Public backbone mapping directories and root names. | Root Zone Operator |
| **3** | Peer Memory Cache / Mirrored Subnets | **Federated Subnets** | Iterative traversal of all user-approved peer keys. | Subnet Owner |

### 5.2. Collision & Shadowing Rules

#### Rule A: Namespace Shadowing Absolute
A lower-tier subnet cannot shadow, duplicate, or intercept a name registered at a higher tier. If `registry.json` has allocated the top-level string `search`, a subnet entry claiming `search` is permanently ignored by the client parsing engine.

#### Rule B: The Ambiguity Halt
If a global shorthand query (e.g., `pwss://blog`) does not exist in Tier 1 or Tier 2, but exists simultaneously inside multiple independent Tier 3 federated subnets, the engine **must throw a fatal exception** and prompt the user to explicitly define the scope:

```text
Ambiguous Target ⟹ Halt ⟹ Force Scoped Syntax (e.g., pwss://alice/blog)
```

---

## 6. Client Configuration Syntax

### 6.1. Petname Remapping (`petnames.conf`)
The local petname map allows the user to explicitly strip hierarchy away from frequently accessed sites, transforming complex federated paths into private top-level shorthands.

```text
# PWSS Local Alias Registry Layout
# Shorthand       Target Cryptographic Path
3pedal            bob/projects/3pedal-ev/docs/v1
search            alice/tools/search-engine
unix              zai1208/dotfiles/arch-minimal
```

---

## 7. Lifecycle Protocols: Registration & Updates

### 7.1. Autonomous Creation
A developer creates a PWSS identity offline using entirely standard GPG binaries. 
1. Generate an asymmetric keypair locally.
2. Formulate a personal `manifest.json`.
3. Cleartext-sign the manifest: `gpg --clearsign manifest.json`.
4. Stage the generated file on any chosen public static storage server (HTTPS, IPFS, local server instance).

### 7.2. Root Index Integration via GitOps
To merge a standalone authority or a newly minted subnet into the primary global discovery tier, developers use a git-driven submission workflow:
1. The developer forks the authoritative upstream root registry repository.
2. The developer appends their authority stanza code snippet directly to the `entries` block inside `registry.json`.
3. The developer files a Pull Request (PR).
4. The repository automated CI suite verifies the validity of the signed manifest data block. 
5. The repository maintainer applies a cryptographically signed commit to merge the PR, preserving an unalterable, linear history of state transitions.
