# SKILLS.md — Amara Diallo

## Core Competencies

### 1. Godot 4 MultiplayerAPI
- ENet server/client setup with `ENetMultiplayerPeer`
- `peer_connected` / `peer_disconnected` signal handling
- `multiplayer.is_server()` and peer ID management
- Connection and disconnection lifecycle

### 2. Authority Model
- `set_multiplayer_authority(peer_id)` on dynamically spawned nodes
- `is_multiplayer_authority()` guards on all state mutations
- Server (peer 1) ownership of all gameplay-critical state
- Per-entity authority: player nodes owned by their respective peer

### 3. RPC Design and Security
- `@rpc("any_peer")` — client-to-server requests with mandatory server validation
- `@rpc("authority")` — server-to-client confirmations
- `@rpc("call_local")` — caller also receives the RPC
- `multiplayer.get_remote_sender_id()` — always check, always validate
- Reliable vs. unreliable RPC mode decision criteria

### 4. MultiplayerSynchronizer
- Property path configuration for replicated state
- `REPLICATION_MODE_ALWAYS` / `REPLICATION_MODE_ON_CHANGE` / `REPLICATION_MODE_NEVER`
- Authority-to-peers broadcast pattern
- Invalid property path debugging

### 5. MultiplayerSpawner
- `spawn_path` configuration
- Auto-spawn on authority add_child
- Peer-synchronized despawn via `queue_free()`
- Node naming for authority lookup (name = peer_id string)

### 6. Network Transport
- ENet configuration for LAN and internet play
- WebRTC `WebRTCPeerConnection` and `WebRTCMultiplayerPeer`
- STUN/TURN server configuration for NAT traversal
- Signaling server design for WebRTC SDP exchange

### 7. Latency Testing
- Simulated 100ms/200ms latency in development
- Reconnection handling and orphan node prevention
- Dead reckoning for position interpolation

### 8. Advanced Networking
- Custom binary packet protocol with `PackedByteArray`
- Delta compression for state updates
- Nakama open-source backend integration for matchmaking
- Relay server architecture for firewall-friendly P2P

## Secondary Skills
- GDScript typed patterns (working with Yuki on signal architecture)
- Lobby and matchmaking flow design
- Server-side cheat detection logging
