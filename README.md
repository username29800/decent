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

## Index-Repo Design (Revision 2)

This version redesigns the subrepo layout to simplify maintenance, enable partial Git operations, and support automation for fetching, updating, and distributing content. This is now the mainstream approach for implentation.

---

### 🌳 Repo Tree Structure

_main repo\
\
__info subrepo\
___node-info db / csv\
___metadata db / json or csv\
\
__data subrepo\
___branch per resource\
\
__script subrepo\
___source db / csv: resource id, source\
___fetch script\
___db update automation\

### 📌 Subrepo Roles

- **_info subrepo**
  - `node-info db`: CSV listing peer IDs, certification hashes, instance reputations.
  - `metadata db`: JSON/CSV mapping resource IDs to version info, commit hashes, sizes.

- **_data subrepo**
  - Each *branch* contains one resource's entire history.
  - Clients/peers can `git fetch` or `git pull` only the branches they need.
  - Greatly reduces bandwidth and storage requirements.

- **_script subrepo**
  - `source db`: Maps resource IDs to upstream/original sources.
  - Automation scripts for:
    - Fetching from source
    - Committing updates
    - Updating metadata in `_info`

---

### 🗂️ Node File Tree Examples

#### <client node>

_client executable\
_config dir\
_log dir(optional)\
_repo tree\
__info subrepo (readonly)\
___databases(updated every request)\
__data subrepo (probably readonly)\
___resource per branch\

✅ Clients only **read** data and info.  
✅ Local decisions about what to fetch are guided by `_info` data.  
✅ Efficient selective sync.

---

#### <peer node>

_peer executable\
_log dir(mandatory)\
_config dir\
_repo tree (same as the above)\

✅ Same repo structure as clients.  
✅ Responds to broadcasts with avasc score.  
✅ Serves resources locally from `_data`.  
✅ Logs uptime to maintain avasc score.

---

#### <instance node>

_instance node executable\
_log dir(mandatory)\
_config dir\
_full repo tree (as described in the specification)\

✅ Maintains authoritative `_info` and `_data`.  
✅ Automates crawling, fetching, committing, and pushing to other instances.  
✅ Uses `_script` subrepo for automation.

---

### ⚡️ Design Advantages

✅ Separation of metadata, content, and automation.  
✅ Lightweight, selective Git fetch/pull.  
✅ Easy federation between instance nodes via Git push.  
✅ Local cache stays up to date without needing a central server.  
✅ Supports automated resource acquisition from source URLs.

---

### 🤖 Automation Flow Example

1. Instance fetches from a *source* using `_script/fetch`.
2. Commits new/updated content to `_data` subrepo.
3. Updates `_info` metadata DB (version, size, commit hash).
4. Pushes repo updates to other trusted instances.

---

### 🧭 Client/Peer Usage Flow

- On request:
  - Check `_info` subrepo for latest version/commit hash.
  - If already cached → load locally.
  - Otherwise → fetch required branch from instance or other peers.

---

### 🔗 Notes on Federation

- Instances share repos via Git push/pull.  
- Peers/clients can clone and selectively fetch.  
- Metadata updates are small and quick to sync.

---

### 💡 Pending

- Scripts in `_script` can include:
  - Bandwidth measurement
  - Avasc scoring automation
  - Cache cleaning based on size limits
  - Log management
- `_info` can be designed to support JSON for richer metadata.

---

### ⚠️ Important

- **Clients and Peers:** should treat `_info` and `_data` as read-only.  
- **Instances:** are responsible for writing, updating, and sharing.  
- Automation requires careful access control and key management.

---

## 📌 Example Use Cases

✅ Low-power home servers acting as peers.  
✅ Instance nodes on more capable hardware.  
✅ Resource federation across organizations or communities.  
✅ Reducing redundant large CDN traffic.  

---

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