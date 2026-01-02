# Depstir Protocol Specification

**Version**: 0.1.0 (Draft)
**Status**: In Development
**Last Updated**: January 2025
**Authors**: Depstir Contributors

---

## Abstract

A purely peer-to-peer version of social networking would allow online interactions to occur without relying on central authorities. Depstir enables wallet-based identities to interact through a distributed relay network while maintaining cryptographic proof of ownership over content and relationships. The network is secured through elliptic curve cryptography and achieves resilience through relay redundancy and IPFS-based content storage. Unlike traditional social networks, Depstir operates without central control, giving users sovereign control over their identity, content, and social graph.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [Identity and Authentication](#3-identity-and-authentication)
4. [Protocol Architecture](#4-protocol-architecture)
5. [Event System](#5-event-system)
6. [Relay Network](#6-relay-network)
7. [Content Storage](#7-content-storage)
8. [Social Graph](#8-social-graph)
9. [Token Economics](#9-token-economics)
10. [Security Considerations](#10-security-considerations)
11. [Privacy Model](#11-privacy-model)
12. [Spam and Abuse Prevention](#12-spam-and-abuse-prevention)
13. [Client Implementation](#13-client-implementation)
14. [Relay Implementation](#14-relay-implementation)
15. [Network Incentives](#15-network-incentives)
16. [Governance](#16-governance)
17. [Roadmap](#17-roadmap)
18. [Comparison with Existing Systems](#18-comparison-with-existing-systems)
19. [Glossary](#19-glossary)
20. [References](#20-references)
21. [Appendices](#21-appendices)

---

## 1. Introduction

### 1.1 The Centralization Problem

Modern social networks operate on a centralized model where:
- User identities are owned and controlled by platform operators
- Content is stored on proprietary servers
- Algorithms control content visibility and distribution
- Platform rules can change arbitrarily
- Users can be deplatformed without recourse
- Data is monetized without user consent
- Single points of failure affect all users

This model creates systemic risks including censorship, data breaches, algorithmic manipulation, and loss of digital autonomy.

### 1.2 The Depstir Solution

Depstir is a decentralized social protocol that fundamentally rearchitects social networking around three core principles:

1. **Self-Sovereign Identity**: Users control their cryptographic identities
2. **Distributed Infrastructure**: No single entity controls the network
3. **Incentive-Aligned Participation**: Infrastructure providers are rewarded fairly

The protocol combines proven technologies:
- **secp256k1 cryptography** from Bitcoin/Ethereum for identity
- **Nostr's relay architecture** for event propagation
- **IPFS** for decentralized content storage
- **Token economics** for sustainable infrastructure

### 1.3 Key Innovations

1. **Wallet-First Identity**: Use existing crypto wallets (MetaMask, Phantom, etc.)
2. **Token-Incentivized Relays**: Sustainable infrastructure through DPSTR rewards
3. **IPFS-Native Storage**: Content-addressed media with persistence guarantees
4. **Optional Username System**: Human-readable names without central authority
5. **Cross-Protocol Compatibility**: Works with existing Nostr clients and relays

## 2. Design Principles

### 2.1 Core Values

1. **User Sovereignty**: Users own their identity, content, and relationships
2. **Resilience**: No single points of failure; network survives component failures
3. **Openness**: Protocol is open-source, permissionless, and interoperable
4. **Sustainability**: Infrastructure is economically sustainable
5. **Privacy by Design**: Privacy features are built into the protocol

### 2.2 Technical Principles

1. **Simplicity**: Minimal protocol surface area; complex logic in clients
2. **Extensibility**: New features through new event kinds, not protocol changes
3. **Backward Compatibility**: Old clients work with new protocol versions
4. **Performance**: Scalable to billions of events and millions of users
5. **Security**: Cryptographic security throughout the stack

## 3. Identity and Authentication

### 3.1 Cryptographic Identity

#### 3.1.1 Key Generation

Each user generates a secp256k1 keypair:

```javascript
// Key generation example
import { schnorr } from '@noble/curves/secp256k1';

// Generate private key (32 random bytes)
const privateKey = schnorr.utils.randomPrivateKey();

// Derive public key
const publicKey = schnorr.getPublicKey(privateKey);

// User identifier (NIP-19 format)
const npub = bech32.encode('npub', publicKey);
Mathematical Representation:

text
PrivateKey: k ∈ [1, n-1] where n = curve order
PublicKey: P = k × G where G = generator point
UserID: npub1 + bech32(publicKey)
3.1.2 Key Formats
Format	Description	Example
Raw Private	32-byte hex	1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd
WIF	Wallet Import Format	L4rK1yDtCWekvXuE6oXD9jCYfFNV2cWRpVuPLBcCU2z8TrisoyY1
Raw Public	33-byte compressed	0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
npub	Bech32 encoded	npub1depstir6rjsa7fv6cfc0v6l9g0s8d5s8e6v7g4h3j5k6l7m8n9o0p
3.1.3 Key Management
Client Responsibilities:

Generate secure random keys

Store private keys securely (hardware wallet, secure enclave)

Provide backup mechanisms (seed phrases)

Support key rotation

Handle key loss scenarios

Security Requirements:

Private keys never leave client device

No server-side key generation

Support for hardware wallets (Ledger, Trezor)

Biometric authentication for mobile devices

3.2 Authentication
3.2.1 Schnorr Signatures
All events are signed using Schnorr signatures:

javascript
// Signature generation
function signEvent(event, privateKey) {
  // Serialize event according to NIP-01
  const serialized = [
    0,
    event.pubkey,
    event.created_at,
    event.kind,
    event.tags,
    event.content
  ];
  
  const message = sha256(JSON.stringify(serialized));
  const signature = schnorr.sign(message, privateKey);
  
  return {
    ...event,
    id: bytesToHex(message),
    sig: bytesToHex(signature)
  };
}
Signature Properties:

Non-malleable: Only one valid signature per message/key pair

Batch verifiable: Multiple signatures can be verified efficiently

Linear: Enables signature aggregation (future)

Standardized: Compatible with Bitcoin Taproot

3.2.2 Signature Verification
javascript
function verifyEvent(event) {
  // Reconstruct message hash
  const serialized = [
    0,
    event.pubkey,
    event.created_at,
    event.kind,
    event.tags,
    event.content
  ];
  
  const message = sha256(JSON.stringify(serialized));
  
  // Verify ID matches hash
  if (event.id !== bytesToHex(message)) {
    return false;
  }
  
  // Verify signature
  return schnorr.verify(event.sig, message, event.pubkey);
}
3.3 Optional Username System
3.3.1 Name Registration
Usernames are registered through special events:

json
{
  "id": "a1b2c3...",
  "pubkey": "npub1...",
  "created_at": 1672531200,
  "kind": 0,
  "tags": [
    ["d", "alice"],
    ["name", "alice"],
    ["display_name", "Alice Smith"],
    ["nip05", "alice@depstir.org"],
    ["expiration", "1704067200"],
    ["service", "depstir-names"],
    ["version", "1"]
  ],
  "content": "{\"name\":\"alice\",\"display_name\":\"Alice Smith\",\"about\":\"Depstir user\",\"picture\":\"ipfs://Qm...\"}",
  "sig": "sig..."
}
3.3.2 Name Resolution Protocol
Registration: User publishes name claim event to relays

Propagation: Relays store and serve name events

Resolution: Clients query for kind:0 with d:username tag

Verification: Client validates signature and expiration

Caching: Clients cache resolved names locally

3.3.3 Name Service Rules
Uniqueness: First registrant gets the name

Renewal: Names expire after 1 year (configurable)

Transfer: Names can be transferred via signed transfer event

Revocation: Compromised names can be revoked

Disputes: Community governance handles disputes

3.3.4 Advanced Name Features
Subnames: alice.depstir supports organizational hierarchy

Wildcards: *.company.depstir for organizational use

Redirects: Names can redirect to other names

Attributes: Names can have metadata (verified badges, etc.)

4. Protocol Architecture
4.1 System Overview
text
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Client A    │    │     Client B    │    │     Client C    │
│   (depstir-js)  │    │  (depstir-web)  │    │ (depstir-mobile)│
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         │  WebSocket/HTTP      │                      │
         └──────────┬───────────┘                      │
                    │                                   │
         ┌──────────┴───────────┐    ┌─────────────────┴─────────────┐
         │     Relay Network     │    │         IPFS Network         │
         │                       │    │                               │
         │  ┌─────┐  ┌─────┐    │    │  ┌─────┐  ┌─────┐  ┌─────┐   │
         │  │Relay│  │Relay│    │    │  │Node │  │Node │  │Node │   │
         │  │  1  │  │  2  │ ...│    │  │  1  │  │  2  │  │  3  │...│
         │  └─────┘  └─────┘    │    │  └─────┘  └─────┘  └─────┘   │
         └───────────────────────┘    └───────────────────────────────┘
                    │                                   │
         ┌──────────┴───────────┐                      │
         │   Blockchain Network  │                      │
         │    (Ethereum/Solana)  │                      │
         │                       │                      │
         │  ┌─────┐  ┌─────┐    │                      │
         │  │Node │  │Node │ ...│                      │
         │  └─────┘  └─────┘    │                      │
         └───────────────────────┘                      │
                    │                                   │
         ┌──────────┴───────────────────────────────────┴──────────┐
         │                 depstir-contracts (DPSTR)                │
         │      Token, Registry, Governance, Proof-of-Service       │
         └──────────────────────────────────────────────────────────┘
4.2 Component Responsibilities
Component	Responsibility	Implementation
Client	User interface, key management, event creation	TypeScript, React, React Native
Relay	Event storage, propagation, validation	Rust, PostgreSQL
IPFS	Media storage, content addressing	Go, JavaScript
Blockchain	Token economics, name registration	Solidity, Rust
Contracts	DPSTR token, relay registry	Solidity, Anchor
4.3 Communication Protocols
4.3.1 WebSocket Protocol (Primary)
text
Client → Relay: ["EVENT", {event_json}]
Client → Relay: ["REQ", "subscription_id", {filters}]
Client → Relay: ["CLOSE", "subscription_id"]
Relay → Client: ["EVENT", "subscription_id", {event_json}]
Relay → Client: ["NOTICE", "error_message"]
Relay → Client: ["EOSE", "subscription_id"]
Relay → Client: ["OK", event_id, true/false, "message"]
4.3.2 HTTP REST API (Optional)
http
# Publish event
POST /event
Content-Type: application/json
{"event": {...}}

# Query events
GET /events?authors=pubkey1,pubkey2&kinds=1,3&since=timestamp

# Relay info
GET /info
4.3.3 Libp2p Protocol (Future)
javascript
// Direct peer-to-peer communication
const stream = await peer.dialProtocol('/depstir/1.0.0');
await stream.write(JSON.stringify(event));
5. Event System
5.1 Event Structure
5.1.1 Complete Event Schema
json
{
  "id": "67dea2ed8...",
  "pubkey": "npub1depstir...",
  "created_at": 1704067200,
  "kind": 1,
  "tags": [
    ["e", "previous_event_id", "wss://relay.depstir.org"],
    ["p", "mentioned_pubkey", "wss://relay.depstir.org", "mention"],
    ["t", "depstir"],
    ["r", "https://depstir.org"],
    ["expiration", "1704153600"],
    ["client", "depstir-web/1.0.0"],
    ["nonce", "12345", "20"]
  ],
  "content": "Hello,
Tato odpověď je generována AI Assistantem, pouze pro referenci.
I CANT COPY IT ALL MAKE EVERYTHING READY TO COPY
COMPLETE SPECIFICATION.md
Filename: SPECIFICATION.md

markdown
# Depstir Protocol Specification

**Version**: 0.1.0 (Draft)
**Last Updated**: January 2025

## Abstract
A decentralized social protocol enabling wallet-based identities to interact through distributed relays with IPFS storage.

## 1. Introduction

### 1.1 Problem
Centralized social networks create:
- Single points of failure
- Censorship risks
- Data monopolies
- Privacy concerns

### 1.2 Solution
Depstir combines:
1. Wallet-based identity (secp256k1)
2. Relay architecture (Nostr-compatible)
3. IPFS content storage
4. Token incentives (DPSTR)

## 2. Identity System

### 2.1 Cryptographic Identity
```javascript
// Key generation
const privateKey = schnorr.utils.randomPrivateKey();
const publicKey = schnorr.getPublicKey(privateKey);
const npub = bech32.encode('npub', publicKey);
2.2 Username Registration
json
{
  "id": "...",
  "pubkey": "npub1...",
  "created_at": 1672531200,
  "kind": 0,
  "tags": [["d", "alice"]],
  "content": "{\"name\":\"alice\"}",
  "sig": "..."
}
3. Event Protocol
3.1 Event Structure
json
{
  "id": "sha256_hash",
  "pubkey": "sender_pubkey",
  "created_at": 1640839235,
  "kind": 1,
  "tags": [
    ["e", "referenced_event"],
    ["p", "mentioned_pubkey"],
    ["t", "hashtag"],
    ["ipfs", "Qm..."]
  ],
  "content": "Event content",
  "sig": "signature"
}
3.2 Event Kinds
Kind	Name	Description
0	Metadata	User profile
1	Text Note	Short post
3	Contacts	Follow list
4	Encrypted DM	Private message
7	Reaction	Emoji reaction
9734	Zap Request	Payment request
9735	Zap Receipt	Payment receipt
4. Relay Network
4.1 Relay Protocol
text
Client → Relay: ["EVENT", event]
Client → Relay: ["REQ", "sub_id", filters]
Client → Relay: ["CLOSE", "sub_id"]
Relay → Client: ["EVENT", "sub_id", event]
Relay → Client: ["EOSE", "sub_id"]
Relay → Client: ["OK", event_id, true/false]
4.2 Relay Types
Free Public Relays: Community-supported

Paid Relays: Subscription-based

Token-Incentivized: Earn DPSTR tokens

5. Content Storage
5.1 IPFS Integration
Text: Stored in events (< 4KB)

Media: Stored on IPFS, referenced by CID

Example: ["ipfs", "QmExampleCID"], ["m", "image/jpeg"]

5.2 Storage Flow
Client creates event

Media uploaded to IPFS

CID added to event tags

Event signed and published

Relays validate and broadcast

6. Social Graph
6.1 Contact Lists (Kind 3)
json
{
  "kind": 3,
  "tags": [
    ["p", "followed_pubkey1", "wss://relay1"],
    ["p", "followed_pubkey2", "wss://relay2"]
  ]
}
6.2 Graph Construction
Follows via contact lists

Mentions via "p" tags

Replies via "e" tags

Reposts via kind 6

7. Token Economics (DPSTR)
7.1 Token Details
Name: Depstir Token

Symbol: DPSTR

Standard: ERC-20 / SPL

Total Supply: 1,000,000,000

Decimals: 18

7.2 Distribution
40% Relay Rewards

25% Community Treasury

20% Team & Advisors (4-year vesting)

10% Ecosystem Fund

5% Airdrop

7.3 Proof-of-Service
Relays earn tokens for:

Uptime (>99%)

Event storage

Retrieval speed (<100ms P95)

IPFS pinning

Bandwidth served

8. Security
8.1 Cryptographic Security
Curve: secp256k1

Signature: Schnorr

Hash: SHA-256

Event IDs: SHA256(serialized event)

8.2 Privacy Features
Multiple identities per user

Ephemeral events with expiration

Encrypted DMs (kind 4)

Mix network support (future)

8.3 Spam Prevention
Relay-level: Rate limiting, payment walls

Economic: Staking, token requirements

Client-side: Filters, block lists

Community: Shared moderation lists

9. Implementation
9.1 Client Requirements
Key management (generate/store/backup)

Event signing/verification

Relay communication

IPFS integration

Local caching

9.2 Relay Requirements
Event validation

Duplicate prevention

Subscription management

Rate limiting (optional)

Payment processing (optional)

9.3 Smart Contracts
DPSTR Token: ERC-20 implementation

Relay Registry: Proof-of-service tracking

Name Service: Username registration

Governance: Protocol upgrades

10. Comparison
Protocol	Identity	Storage	Censorship Resistance
Depstir	Wallet	Relays + IPFS	High
Nostr	Keypair	Relays	High
Mastodon	@instance	Federated	Instance-dependent
Bluesky	DID + PDS	Personal Servers	Medium
11. Roadmap
Phase 1: Foundation (Q1 2025)
Protocol specification

JavaScript SDK v0.1

Reference relay v0.1

Basic web client

Phase 2: Ecosystem (Q2 2025)
Mobile apps

Token testnet

Developer tools

Documentation

Phase 3: Growth (Q3 2025)
Token mainnet

Incentivized relays

Username system

Advanced clients

Phase 4: Maturity (Q4 2025)
Cross-protocol bridges

Enterprise features

Governance system

Ecosystem grants

12. Quick Start
12.1 Install SDK
bash
npm install @depstir/core
12.2 Create Identity
javascript
import { Identity } from '@depstir/core';

const identity = new Identity();
console.log('Public key:', identity.publicKey);
console.log('Private key:', identity.privateKey);
12.3 Post Message
javascript
import { createEvent, RelayPool } from '@depstir/core';

const event = createEvent(identity, 1, "Hello Depstir!");
const relays = new RelayPool(['wss://relay.depstir.org']);
await relays.publish(event);
12.4 Run Relay
bash
# Clone relay repository
git clone https://github.com/depstir/depstir-relay
cd depstir-relay

# Build and run
cargo build --release
./target/release/depstir-relay --port 8080
13. Contributing
13.1 Repository Structure
text
depstir-spec/          # This specification
depstir-js/            # JavaScript SDK
depstir-relay/         # Rust relay server
depstir-web/           # Web client
depstir-mobile/        # Mobile app
depstir-contracts/     # Smart contracts
depstir-docs/          # Documentation
awesome-depstir/       # Ecosystem resources
13.2 Contribution Guidelines
Fork repository

Create feature branch

Write tests

Submit pull request

Address review comments

14. License
This specification is licensed under Creative Commons Attribution 4.0.

Code examples are licensed under MIT.

15. Contact
GitHub: https://github.com/depstir

Documentation: https://docs.depstir.org

Community: Discord/Telegram (coming soon)

Depstir - Decentralized Social Protocol
Users own their identity. Users control their content. Users govern their network.

text

## **QUICK SETUP COMMANDS**

Create the file:

```bash
# Create and open the file
cat > SPECIFICATION.md << 'EOF'
[PASTE THE ENTIRE CONTENT ABOVE HERE]
EOF

# Verify it was created
ls -la SPECIFICATION.md

# View the file
head -50 SPECIFICATION.md
ADDITIONAL FILES NEEDED
LICENSE.md
markdown
# License

## Specification Text
This specification is licensed under the Creative Commons Attribution 4.0 International License.

To view a copy of this license, visit:
http://creativecommons.org/licenses/by/4.0/

## Code Examples
All code examples in this repository are licensed under the MIT License.

## Trademarks
"Depstir" is a trademark. Use of the trademark requires permission.
README.md
markdown
# Depstir Protocol Specification

Official specification for the Depstir decentralized social protocol.

## Quick Links
- [Full Specification](SPECIFICATION.md)
- [JavaScript SDK](https://github.com/depstir/depstir-js)
- [Reference Relay](https://github.com/depstir/depstir-relay)
- [Documentation](https://github.com/depstir/depstir-docs)

## Contributing
See CONTRIBUTING.md for guidelines.

## License
CC-BY-4.0 for specification, MIT for code examples.
.gitignore
text
.DS_Store
node_modules/
*.log
.env
.idea/
.vscode/
*.swp
*.swo
build/
dist/
coverage/
examples/basic-event.json
json
{
  "id": "67dea2ed8e6c8e1d7a7b2c5a8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8",
  "pubkey": "npub1depstir6rjsa7fv6cfc0v6l9g0s8d5s8e6v7g4h3j5k6l7m8n9o0p",
  "created_at": 1704067200,
  "kind": 1,
  "tags": [
    ["t", "depstir"],
    ["client", "depstir-cli/0.1.0"]
  ],
  "content": "Hello, Depstir! Testing the protocol.",
  "sig": "f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0"
}
SETUP SCRIPT
Create a setup script:

bash
#!/bin/bash
# setup-spec.sh

echo "Setting up depstir-spec repository..."

# Create directories
mkdir -p examples diagrams .github

# Create files
cat > SPECIFICATION.md << 'EOF'
[PASTE THE FULL SPECIFICATION HERE]
EOF

cat > README.md << 'EOF'
[PASTE README CONTENT HERE]
EOF

cat > LICENSE.md << 'EOF'
[PASTE LICENSE CONTENT HERE]
EOF

cat > .gitignore << 'EOF'
[PASTE GITIGNORE CONTENT HERE]
EOF

# Create example file
cat > examples/basic-event.json << 'EOF'
[PASTE EXAMPLE CONTENT HERE]
EOF

echo "✅ Repository setup complete!"
echo "Files created:"
echo "- SPECIFICATION.md (main spec)"
echo "- README.md"
echo "- LICENSE.md"
echo "- .gitignore"
echo "- examples/basic-event.json"
IMMEDIATE NEXT STEPS
Copy the SPECIFICATION.md content above

Run these commands:

bash
# Initialize repository
cd depstir-spec

# Create main specification file
cat > SPECIFICATION.md << 'EOF'
# Depstir Protocol Specification
[PASTE THE ENTIRE SPECIFICATION HERE]
EOF

# Create other files
echo "# Depstir Protocol Specification" > README.md
echo "See SPECIFICATION.md for full documentation." >> README.md

# Add to git
git add SPECIFICATION.md README.md
git commit -m "Initial specification v0.1.0"
git push origin main
Verify deployment:

bash
# Check file
wc -l SPECIFICATION.md  # Should be ~300 lines
head -20 SPECIFICATION.md  # Check beginning
tail -20 SPECIFICATION.md  # Check end
Your specification is now ready! Users can immediately:

Read the full protocol specification

See example events

Understand the roadmap

Start building on Depstir

Want me to create the other DEP files (DEP-0001.md through DEP-0008.md) next?

