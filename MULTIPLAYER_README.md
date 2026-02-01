# Oh My Hungry God - Multiplayer Implementation

Complete multiplayer AR game implementation for Global Game Jam 2026.

## 🎮 What Was Built

A synchronous multiplayer system transforming the single-player AR game into a collaborative experience:

- **Authoritative .NET 9 Backend** with SignalR WebSocket communication
- **Web Host Display** showing game state on TV/projector
- **iOS AR Clients** as controllers, throwing fruits via AR

## 📦 Project Structure

The project is split across multiple repositories:

- **[GGJ2026-Backend](https://github.com/borissedov/GGJ2026-Backend)**: .NET 9 Backend
- **[GGJ2026-Frontend](https://github.com/borissedov/GGJ2026-Frontend)**: TypeScript Frontend
- **[GGJ2026-iOS](https://github.com/borissedov/GGJ2026-iOS)**: iOS ARKit App

## 🎯 How It Works

### Game Flow

```mermaid
graph LR
    Welcome[Welcome Screen] --> Lobby[Lobby]
    Lobby --> Countdown[Countdown 10s]
    Countdown --> Game[In Game]
    Game --> Results[Results]
    
    Game -->|Burnout| GameOver[Game Over]
    GameOver --> Results
```

### State Machine

1. **WELCOME**: Display shows QR code, waiting for first player
2. **LOBBY**: Players join, mark ready
3. **COUNTDOWN**: All ready → 10 second countdown
4. **IN_GAME**: Play 10 orders, each 10 seconds
5. **RESULTS**: Show final stats

### Order Resolution (Immediate Rules)

- **Over-submission**: `submitted[fruit] > required[fruit]` → Instant fail
- **Exact match**: `submitted == required` → Instant success
- **Timeout**: Timer expires → Fail

### Mood System

- Start: NEUTRAL (😐)
- +1 mood every 2 successes → HAPPY (😊)
- -1 mood per failure → ANGRY (😠)
- Below ANGRY → BURNED (💀) → Game Over

## 📡 API Reference

### SignalR Hub Methods (Client → Server)

| Method | Parameters | Description |
|--------|------------|-------------|
| `CreateRoom` | - | Display creates room, returns `{ roomId, joinCode }` |
| `JoinRoom` | `joinCode` | Player joins, returns `{ roomId, playerId }` |
| `SetReady` | `roomId, ready` | Toggle ready state |
| `ReportHit` | `roomId, hitId, fruit` | Submit fruit hit (idempotent) |
| `Ping` | `roomId` | Keep-alive |

### Server Events (Server → Clients)

| Event | Target | When |
|-------|--------|------|
| `RoomStateUpdated` | Display | Player join/leave/ready |
| `CountdownStarted` | Display | All players ready |
| `GameStarted` | Both | Countdown complete |
| `OrderStarted` | Both | New order begins |
| `OrderTotalsUpdated` | Display | Hit counted (not resolved) |
| `OrderResolved` | Both | Order success/fail |
| `MoodChanged` | Display | Mood changes |
| `GameFinished` | Both | 10 orders complete |
| `StateSnapshot` | Mobile | On join/reconnect |

## 🔑 Key Features

### Backend
- ✅ Authoritative server (all validation server-side)
- ✅ SignalR for real-time WebSocket communication
- ✅ In-memory state (ConcurrentDictionary for thread-safety)
- ✅ Idempotent hit processing (prevents duplicates)
- ✅ Background timers for countdown/order timeouts
- ✅ Automatic room cleanup

### Frontend
- ✅ QR code generation for easy joining
- ✅ Real-time lobby with player ready states
- ✅ Live order display with fruit requirements
- ✅ Mood video background system
- ✅ Results screen with stats

### iOS
- ✅ SignalR client integration
- ✅ Lobby UI with join code input
- ✅ Order overlay in AR view
- ✅ Hit reporting to server
- ✅ Maintains single-player mode option

## 🛠️ Configuration

### Backend Settings

`appsettings.json`:
```json
{
  "GameSettings": {
    "OrdersPerGame": 10,
    "OrderDurationSeconds": 10,
    "CountdownDurationSeconds": 10,
    "ResultsTimeoutSeconds": 30,
    "RoomInactivityTimeoutMinutes": 5
  }
}
```

### Frontend Settings

`.env`:
```
VITE_BACKEND_URL=http://localhost:5000/gamehub
```

For production:
```
VITE_BACKEND_URL=https://your-app.azurewebsites.net/gamehub
```

### iOS Settings

`SignalRClient.swift`:
```swift
init(hubUrl: String = "http://localhost:5000/gamehub") {
```

## 📝 Implementation Notes

### What's Complete

✅ Backend: Full implementation, tested and compiles
✅ Frontend: Full implementation, builds successfully
✅ iOS: Integration points created, SignalR package integrated

### iOS SignalR Integration

The iOS app uses `SignalR-Client-Swift` package.
All the integration points are ready:
- `GameManager` reports hits to server when in multiplayer mode
- `LobbyView` handles room joining and ready states
- `OrderOverlayView` displays current order during gameplay

## 🎨 Assets Needed

For the web display, add mood videos to `oh-my-hungry-god-display/public/assets/videos/`:

- `neutral.mp4` - Neutral mood (😐)
- `happy.mp4` - Happy mood (😊)
- `angry.mp4` - Angry mood (😠)
- `burned.mp4` - Burned/game over (💀)

Videos should:
- Be loopable
- Match the visual style of your game
- Be optimized for web (H.264, reasonable bitrate)

## 📚 Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - High-level architectural overview and infrastructure details
- **[Backend](https://github.com/borissedov/GGJ2026-Backend/blob/main/README.md)**: See repo README
- **[Frontend](https://github.com/borissedov/GGJ2026-Frontend/blob/main/README.md)**: See repo README
- **[iOS](https://github.com/borissedov/GGJ2026-iOS/blob/main/README.md)**: See repo README

## 🎓 Architecture Decisions

### Why SignalR?
- Built-in reconnection handling
- Automatic fallback (WebSockets → Server-Sent Events → Long Polling)
- Strong typing with C#
- Good iOS client library available

### Why Authoritative Server?
- Prevents cheating
- Consistent game state across all clients
- Simplified client logic
- Easier to debug

### Why In-Memory State?
- Fast access (no DB overhead)
- Simple to implement
- Sufficient for game jam scope
- Rooms are temporary anyway

### Why Vanilla TypeScript Frontend?
- No framework overhead
- Faster load times
- Simpler to understand
- Easier to customize

## 🚧 Future Enhancements

Not implemented but could be added:

- [ ] Player authentication
- [ ] Room persistence (Redis/Database)
- [ ] Replay system
- [ ] Leaderboards
- [ ] Sound effects
- [ ] Animations for order success/fail
- [ ] Multiple game modes
- [ ] Difficulty settings
- [ ] Room capacity limits

## 🙏 Credits

Created for Global Game Jam 2026

Technologies used:
- .NET 9 + ASP.NET Core
- SignalR
- TypeScript + Vite
- Swift + ARKit
- RealityKit
