# Decent (aka OpenDCDN)

> **Decentralized, energy-efficient content delivery network using layered P2P and Git.**

---

## Terminology

- **dCDN**: *dynamic CDN* — a constantly evolving network with countless nodes and federated instances.
- **Broadcast**: Sending requests to all local peers.
- **Resource**: Web page, image, JavaScript, etc.
- **thr** (*threshold*): Peer-defined availability threshold.
- **avasc** (*availability score*):  
  - Initial: `75 * shared bandwidth (MB/s)`.
  - Otherwise: `{7-day average of (daily uptime (h) / 24h)} * shared bandwidth`.
  - New nodes assume 18h uptime for the past 7 days.

---

## Description

**Decent** (from *decentralization*) is a user-driven, FOSS content distribution network based on partial P2P and layered decentralization.

It aims to:
1. Lower electricity demand by reducing the number of power-hungry global CDN servers.
2. Replace them with voluntary, large-scale networks run by users themselves.
3. Provide a rational hobby platform for sysadmins and home-server enthusiasts.

**Network architecture:**
- **P2P layer**:  
  - *Peers*: local network-level caches.
  - *Clients*: retrieve data from peers.
- **Instance layer**:  
  - Single or multiple instances with Git repositories, nameserver DB, and version DB.
  - Runs on already-in-use devices (smartphones, IoT, energy-efficient home servers).
  - Version management uses Git.

**Design highlights:**
- Transactions are minimized to save power.
- Peer nodes can set:
  - Shared bandwidth threshold (affects avasc).
  - Maximum acceptable file size.
- If no suitable peer is available (e.g. file too large), client falls back to nameserver.

**Cache policy:**
- Local caches for efficiency.
- 2 GB default limit; oldest/least-maintained subrepo evicted first.
- Cache structure mirrors the nameserver’s Git sub-repo tree.
- Possible to clone nameserver’s repository directly.

---

## Project Goals

- **Stability and sustainability** for public/static content distribution.
- **Energy savings** by replacing traditional CDN servers.
- **A fun, rational hobby** for sysadmins and home-server runners.

---

## Procedure

> Clients get latest commit hash and file size from an instance node before broadcast.

### Client
1. **Initialize**: Internal request to start procedure.
2. **Version check**: Request from version server (part of nameserver).
3. **Cache check**: If local cache has the latest version, load directly.
4. **Broadcast**: Request to P2P network (includes resource ID, size, commit hash, avasc threshold).

### Peer
- Listen for broadcasts.
- Check:
  - Avasc threshold.
  - File size limits.
  - Resource availability.
  - Version match.
- Respond with Peer ID and avasc value.

### Client
- Collect peer responses.
- Select peer (fastest or highest avasc).
- Connect and fetch resource.

### Timeout Handling
- If no response after 3 broadcasts:
  - Send update broadcast with lowered threshold.
  - Enter sleep mode after timeout.

### Fallback to Nameserver
- When peer retrieval fails:
  - Direct request to subscribed nameserver.

### Peer Updates
- Listen for update broadcasts.
- Respond with availability if threshold met.

### Client Updates
- Receive availability responses.
- Select peer and send update request.

### Peer Update Connection
- On receiving update request:
  - Fetch resource from instance.
  - Send to client.
  - Measure bandwidth and update avasc.

---

## Decentralization

**Concept summary:**

- **Instance**:
  - Git repo (stores resources as subrepos).
  - Nameserver database (list of Peer IDs, certification hashes, instance IDs, reputations).
  - Version database (resource IDs, sizes, subrepo names, latest commit hashes).
  - Nameserver service.
- **Federation**:
  - Instances push updates to trusted peers listed in their nameserver.
- **Nameserver daemon**:
  - Handles all decentralization tasks.

**Avasc integrity**:
- On shutdown, peer receives random hash from instance.
- Hash must match on next startup; otherwise, avasc drops to 25.
- Prevents uptime spoofing and accounts for abrupt shutdowns.

---

## Role Integration

**Client-Peer Integration**
- Users are clients first.
- Check latest version via instance, then local cache.
- If not cached, query local peers.
- Integrated nodes foster local traffic efficiency.
- Integrated peers maintain avasc scores.

**Instance Integration**
- *Not recommended* due to traffic/power demands.

---

## Index Repo

An experimental alternative implementation.

- Each client/instance maintains an **index-repo**:
  - Same subrepo structure under the main repo.
  - Three branches per subrepo:
    - **update**: Scripts (source addresses) for fetching latest.
    - **info**: Database entries with version, size.
    - **cache**: Stores cached resources.
- **Advantages**:
  - Simplifies node structure.
  - Instance nodes need only one repo (can be pushed/fetched via Git).

---

## Connection Quality Measurement

- **Avasc** measures peer stability.
- Each peer receives a random hash on shutdown:
  - Must match on reconnect.
  - Sudden power losses drop avasc to 25.
- **Daily decrement**:
  - Avasc drops by 1 for each ungraceful disconnect.
  - Resets at midnight (requires client restart).
- **Logging**:
  - Peer/instance clients maintain daily start/termination logs.

---

## Social Network (Optional & Discouraged)

> **WARNING:** This is strongly *discouraged* to implement. May not be available for a long time.  
> **NOTE:** IRC is strongly recommended instead.

**Concept**:
- Decent Social Network Service (*dsns*).
- Optional for instance nodes.
- Uses Git for posts/threads:
  - Each peer has its own branch.
  - Authenticated pushes to instance nodes.
  - Federation spreads changes via git push.
- Extra branches:
  - **trending** and **news** for curation.
- **Requirements**:
  - Suitable only for powerful nodes (desktops).
  - Configurable via nameserver DB subrepo.

---

## License

Free and Open Source Software (FOSS).  
Specific license to be determined.

---

## Contributing

We welcome developers, tinkerers, and energy-conscious sysadmins!  
See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines (coming soon).

---

## Contact

- Issues and discussions: [GitHub Issues](./issues)
- Recommended communication for instance nodes: **IRC**

---
