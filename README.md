# PWSS: The Privacy Stack

**The web is broken. It is address-based, centralized, and owned by "Steel Giants."**

PWSS (PGP Web Static Secure) is a data-centric, cryptographically verified alternative. It replaces DNS with a **Hash-Chained Domain Registry** and replaces browsers with a **Sovereign Verification Pipe.**

## The Stack
- [**pwss-core**](https://github.com/zai1208/pwss-core): The mathematical guardian. Verifies PGP signatures and content integrity.
- **pwss-fetch**: Protocol-agnostic resolution (HTTPS / Reticulum / LoRa).
  - [**pwss-fetch-https**](https://github.com/zai1208/pwss-fetch-https): A HTTPS PoC for fetching pages
- **PWSS browsers**:
  - [**pwss-browser-glow**](https://github.com/zai1208/pwss-browser-glow): A TUI-first browsing experience.
  - [**pwss-browser-qt6**](https://github.com/zai1208/pwss-browser-qt6): A Qt6 based GUI browser, currently the flagship experience.

## The Manifesto
1. **Identity is Sovereign**: Your handle is your PGP key. No registrars.
2. **Data is Location-Agnostic**: It doesn't matter *where* the bits are, only that the *hash* is correct.
3. **Radio Dark Ready**: Designed to function over mesh networks and low-bandwidth radio links.

## How PWSS Compares to Other Web Architectures

If you are coming from the world of decentralized protocols or the modern web, PWSS might look familiar, but it operates on completely different structural rules. 

Unlike platforms that focus on decentralized data storage or social networking primitives, PWSS is a **manifest-driven content application layer** designed for high-assurance, human-readable identity and zero-bloat presentation.

| Feature / Property | Traditional Web (HTTP/HTTPS) | Federated Social (Mastodon / Bluesky) | Decentralized Web (IPFS / IPNS) | PWSS (Sovereign Web) |
| :--- | :--- | :--- | :--- | :--- |
| **Transport Layer** | Fixed (TCP/IP via TLS) | Fixed (HTTP/HTTPS REST API) | Custom P2P BitTorrent-like bitswap | **Agnostic** (Can run over HTTPS, P2P, mesh, or local files) |
| **Data Verification** | Centralized Certificate Authorities (CAs) | Instance-level trust / AT Protocol cryptographic DID | Content Addressing (Content-ID hashes / IPNS keys) | **End-to-End PGP Chains** (Pinned global registry -> signed manifests) |
| **Client Rendering** | Bloated runtime execution engines (JavaScript, DOM, CSS) | Thick clients or heavy web-apps parsing JSON feeds | Heavy browser extensions or gateways rendering complex web pages | **Native, Flat-Data Viewports** (Zero client-side scripting, deterministic presentation) |
| **Primary Goal** | Commercial content delivery & dynamic applications | Microblogging & social network graphing | Censorship-resistant, immutable distributed data storage | **Sovereign Content Access** (Strictly controlled routing tables & verified text streams) |

### Key Distinctions

* **Not a storage layer (vs. IPFS):** IPFS solves the problem of *where files live* and *how to distribute blocks of data* using cryptographic content hashes. PWSS doesn't care where the bytes live or how they are transferred; it focuses on establishing an un-spoofable, verifiable trust relationship between a human cryptographic handle and an exact, signed routing map.
* **Not a social graph (vs. Mastodon/Bluesky):** Fediverse protocols are built to broadcast state updates, relationships, and feeds across server instances. PWSS is a personal content viewport framework—it’s designed to provide an air-gapped runtime environment for consuming secure text, logs, and assets without tracking or background code execution.
* **Not replacing the pipes (vs. HTTP):** PWSS doesn't compete with the underlying transport networks. It intercepts the data coming out of them, sanitizing and validating payloads through local backend fetcher modules before anything touches your screen.

## Join the Registry
Submit a Pull Request with your `handle.json.asc`. 
Current Root of Trust: `1D621CE3F6D554DC3E6E2551B1437E7DFC87A9B4`
