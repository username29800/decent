# Decent (aka OpenDCDN)

> **Decentralized, energy-efficient content delivery network using layered P2P and Git.**

---

## ⚡️ Overview

**Decent** is a user-driven, Free and Open Source Software (FOSS) content distribution network that combines layered P2P architecture with Git-based versioning.  
It is designed to reduce the energy footprint of traditional CDN infrastructure by turning existing user devices into voluntary cache nodes and instance servers.

**Goals:**
- Sustainable, resilient distribution of public/static resources.
- Lower power usage by avoiding always-on, high-power CDN server farms.
- Foster sysadmin-level hobby participation via open federation.

---

## 📚 Terminology

- **dCDN** (*dynamic CDN*): A continuously evolving mesh of nodes, peers, and federated instances that connect and disconnect organically.
- **Broadcast**: Sending a request to all locally reachable peers.
- **Resource**: Web page, image, JavaScript file, etc.
- **thr** (*threshold*): The minimum avasc score required for a peer to accept a request.
- **avasc** (*availability score*): 
  - New nodes start with `75 × shared bandwidth (MB/s)`.
  - Afterward: `{7-day average uptime fraction} × shared bandwidth`.
  - New peers are treated as if they had 18h uptime over 7 days.

---

## 🌐 Architecture

Decent adopts a **layered, hybrid approach**:

### P2P Layer
- **Clients**: Request resources; maintain local cache.
- **Peers**: Local cache nodes; can serve resources to clients.

This layer is intended to **maximize locality** and reduce cross-network traffic.

### Instance Layer
- Hosts the **nameserver** service.
- Stores resources as **Git repositories** with subrepo structure.
- Maintains:
  - Nameserver database (peer metadata, certification hashes, reputation)
  - Version database (resource IDs, commit hashes, repo mapping)

---

## ⚙️ Design Principles

- **Energy efficiency**: Uses already-on devices (phones, DIY IoT, home servers).
- **Minimized traffic**: Local-first cache policy, P2P sharing.
- **Layered decentralization**: Instance federation via Git pushes.
- **Flexible participation**: Power users can choose to host peers or instances.

---

## 🗂️ Cache and Repository Structure

- **Local Cache**:
  - Default size limit: 2 GB.
  - Deletion policy: Least-maintained subrepo first.
  - Git subrepo tree matching the nameserver's structure.

- **Nameserver/Instance Storage**:
  - Main Git repository with subrepos for each resource.
  - Cloneable by any node to sync data.

This design **uses Git's own mechanisms** (branches, subrepos, remote pushes) to maintain consistency with minimal traffic.

---

## 🛰️ Protocol Flow

> Client starts by querying the latest version from the instance (nameserver).

### 📌 Client Side
1. **Initialization**: Internal trigger; calls other functions.
2. **Version Request**: Ask nameserver for latest commit hash and size.
3. **Local Cache Check**: Load if latest.
4. **Broadcast**: Send request to local peers with:
   - Resource ID
   - Size
   - Commit hash
   - Avasc threshold

### 📌 Peer Side
- Listens for broadcasts.
- Verifies:
  - Avasc threshold.
  - File size limit.
  - Resource availability.
  - Commit hash match.
- Responds with Peer ID and avasc value.

### 📌 Client Response Handling
- Collects peer responses.
- Selects peer (fastest response or highest avasc).
- Requests and downloads resource.

### 📌 Timeout Handling
- On silence after 3 attempts:
  - Broadcast with lowered threshold.
  - Enter sleep mode post-timeout.

### 📌 Fallback
- If P2P fails, client queries its subscribed nameserver.

---

## 🗃️ Decentralization Model

**Instance Node = Nameserver + Storage + Federation**

- Git repository with subrepos for resources.
- Nameserver DB:
  - Peer IDs
  - Certification hashes
  - Reputation
- Version DB:
  - Resource IDs
  - Sizes
  - Repo identifiers
  - Latest commit hashes

**Federation:**
- Trusted instances push changes to one another automatically.

**Nameserver Daemon:**
- Single executable managing:
  - Resource commits
  - Subrepo creation
  - DB updates
  - Federation pushes

---

## 💾 Index Repo Design (Experimental)

**Alternative to separate DB approach.**

- Each client/instance keeps an **index-repo**:
  - Same subrepo structure as instance's main repo.
  - Three branches per subrepo:
    - **update**: Scripts to fetch directly from original source.
    - **info**: Version metadata (e.g. `20240115-1-instanceID`) and size.
    - **cache**: Actual resource storage.

**Advantages:**
- Eliminates need for multiple DBs.
- Simplifies instance-to-instance synchronization.
- Entire state in one Git repo; federated via standard Git push/pull.

---

## 📈 Avasc (Availability Score) System

Ensures reliable peers by measuring uptime and stability.

- Each shutdown issues a random hash to the peer.
- On reconnect:
  - Hash must match.
  - Mismatch → avasc drops to 25.
- Sudden power loss or kill → no termination timestamp → avasc penalty.
- Daily decrement: -1 per ungraceful disconnect, reset at midnight.

**Logs:**
- Peer/instance clients maintain daily start/stop logs for uptime tracking.

---

## 🔄 Role Integration

**Client-Peer Integration:**
- Default mode for typical users.
- Always tries:
  1. Latest version info via instance.
  2. Local cache.
  3. Nearest peers.
- Integrated nodes improve local traffic efficiency and lower external dependency.

**Instance Integration:**
- *Discouraged* on small devices.
- Traffic-heavy; better on dedicated/powerful nodes.

---

## 🚨 Social Network Feature (Experimental & Discouraged)

> **WARNING:** Strongly discouraged. May not be implemented soon.  
> **NOTE:** IRC is recommended for real use.

**Decent SNS Concept:**
- Optional feature for powerful instance nodes.
- Entirely Git-based:
  - Each peer ID = separate branch.
  - Posts/threads stored in `dsns` repo outside main content repo.
- Federation via Git push:
  - Trending/news branches curated from most popular posts.

**Resource-Intensive:**
- Suitable only for desktops or powerful servers.
- Optional, configurable via nameserver DB subrepo.

---

## 💡 Why Git?

- Battle-tested, distributed, versioned storage.
- Familiar toolset for power users.
- Reduces need for custom database syncing.
- Federation-friendly by design.

---

## 📜 License

Free and Open Source Software (FOSS).  
*Specific license TBD.*

---

## 🤝 Contributing

We welcome:
- Developers
- Tinkerers
- Energy-conscious sysadmins

Check out [CONTRIBUTING.md](CONTRIBUTING.md) (coming soon).

---

## 📬 Contact

- Issues and feature discussions: [GitHub Issues](./issues)
- Preferred live communication for instance node operators: **IRC**

---