# Offline Multiplayer Game Source Research & Air-Gap Preparation Report

**Prepared:** September 18, 2026  
**Purpose:** Evaluate the supplied open-source game projects for an environment where all source, tools, packages, documentation, and runtime dependencies are downloaded first, after which the client and server run on an isolated LAN with no Internet access.

---

## 1. Target Requirements

The target environment is:

1. **Game project, not only a framework or SDK.**
2. **Server:** C or C++ preferred.
3. **Client:** Unity or Cocos preferred.
4. **Offline runtime:** after preparation, neither client nor server should require the public Internet.
5. **LAN-only communication:** the client may connect to the local game server; the server may use local databases/caches/files, but should not require an external service.
6. **Offline rebuild:** ideally, the project should also be rebuildable after the machine is disconnected from the Internet.
7. **Game preference:** Chinese game ecosystem projects, MMO/strategy games, or projects that can help build something closer to Clash of Clans.

A project is therefore evaluated at two different levels:

- **Offline runtime:** can the already-built client/server run without Internet?
- **Offline development/rebuild:** can you open, compile, restore dependencies, and rebuild the project without Internet?

The second requirement is significantly harder.

---

# 2. Executive Summary

## Best match for the original technical requirements

### GameProject3 + DemoClient

- **Server:** C++
- **Client:** Unity
- **Database:** MySQL
- **Networking:** Socket API, Boost.Asio, and libuv implementations
- **Game type:** MMORPG/demo game
- **Source:** separate server and Unity client repositories
- **Offline potential:** high once the toolchain and dependencies are archived

Server:

https://github.com/ylmbtm/GameProject3

Client:

https://github.com/ylmbtm/DemoClient

This is the strongest match among the supplied projects if **C++ server + Unity client** is the most important requirement.

---

## Best match for strategy / Clash-of-Clans direction

### OpenEmpires

- **Client:** Unity 6
- **Server:** Rust
- **Database:** PostgreSQL
- **Networking:** WebSockets
- **Multiplayer:** deterministic lockstep
- **Game type:** Age-of-Empires-inspired RTS
- **Offline potential:** high after Cargo and PostgreSQL dependencies are archived

Source:

https://github.com/Chilly5/OpenEmpires

Its server is not C/C++, but its gameplay, simulation, resources, construction, units, and multiplayer architecture are much closer to the desired strategy category.

---

## Best Chinese game-server architecture project

### UnityMMO + SkynetMMO

- **Client:** Unity, C#, Lua/xLua
- **Server:** Skynet + Lua
- **Native server core:** C
- **Protocol:** Sproto
- **Database:** MySQL
- **Server OS:** Linux
- **Offline potential:** high after careful preparation
- **Preparation difficulty:** high because resources, submodules, and some plugins are separate

Client:

https://github.com/liuhaopen/UnityMMO

Server:

https://github.com/liuhaopen/SkynetMMO

Resource repository:

https://github.com/liuhaopen/UnityMMO-Resource

---

## Easiest modern local-learning project

### arlozen/MMORPG

- **Client:** Unity 6
- **Server:** C#
- **Protocol:** Protobuf
- **Database:** MySQL
- **Offline potential:** high
- **Advantage:** client and server are in one repository and the server IP is explicitly configurable

Source:

https://github.com/arlozen/MMORPG

The primary mismatch is the C# server.

---

# 3. Project Comparison Table

| # | Game | Source | Client Stack | Server Stack | Storage / Infra | Offline Runtime | Offline Rebuild Difficulty | Main Concern |
|---|---|---|---|---|---|---|---|---|
| 1 | Mobile MMORPG | https://github.com/Ziden/MobileMMORPG | Unity, C# | .NET Core, C# | Redis | Strong | Medium | Older dependencies; asset download behavior must be checked |
| 2 | Garuna War | Server: https://github.com/eubrunomiguel/garuna | Original project used Unity/C#, but public client repo was not verified | **C++**, Boost.Asio, custom UDP | MySQL Connector | Strong for server | Medium | Complete public Unity client source is missing |
| 3 | Pirate Panic | https://github.com/heroiclabs/unity-sampleproject | Unity, C#, Nakama SDK | Nakama + TypeScript modules | PostgreSQL-compatible DB / Docker | Strong when self-hosted | Medium | Must archive container images, npm packages, Nakama, DB |
| 4 | Boss Room | https://github.com/Unity-Technologies/com.unity.multiplayer.samples.coop | Unity 6, C# | Unity host/server C# | Unity Transport / NGO; current sample also integrates Unity services | Moderate-Strong after modification | Medium-High | Current sample integrates Authentication/Sessions/cloud-oriented services |
| 5 | OpenEmpires | https://github.com/Chilly5/OpenEmpires | Unity 6, C# | Rust, Axum, Tokio, WebSockets | PostgreSQL | Strong | Medium | Server is Rust, not C/C++ |
| 6 | San Andreas Unity | https://github.com/in0finite/SanAndreasUnity | Unity, C# | Unity/C# dedicated server or host | Multiplayer networking in same project | Strong | Medium | Requires legally owned original GTA San Andreas game data |
| 7 | MMORPG (arlozen) | https://github.com/arlozen/MMORPG | Unity 6000.0.56f1, C#, QFramework | C#, custom networking, Protobuf | MySQL, Serilog | Strong | Medium | Server is C#, not C/C++ |
| 8 | UnityMMO / SkynetMMO | Client: https://github.com/liuhaopen/UnityMMO ; Server: https://github.com/liuhaopen/SkynetMMO | Unity 2019.4.28f1, C#, Lua/xLua | Skynet native C core + Lua game services | MySQL, Sproto | Strong after setup | High | Separate resources, submodules, Linux server, missing licensed plugins |
| 9 | GameProject3 / DemoClient | Server: https://github.com/ylmbtm/GameProject3 ; Client: https://github.com/ylmbtm/DemoClient | Unity/C# demo client | **C++**, Socket API / Boost.Asio / libuv | MySQL | **Strong** | Medium-High | Older toolchain; freeze working dependency versions |

---

# 4. Detailed Analysis

## 4.1 Mobile MMORPG

### Source

https://github.com/Ziden/MobileMMORPG

This is a monorepo containing a Unity client and .NET Core server.

### Stack

**Client**

- Unity
- C#

**Server**

- .NET Core
- C#

**Database / storage**

- Redis

### Local startup model

The project README describes a local workflow:

1. Start Redis.
2. Start the server.
3. Open the `GameClient` Unity project.
4. Run the client.

That architecture is naturally compatible with a LAN-only environment.

### Offline-runtime suitability

**Strong**, if every asset needed by the client is stored locally.

The README itself mentions an issue where login can sometimes remain in an asset-downloading state. That means asset retrieval must be audited before the Internet is removed.

Search the project for:

```text
http://
https://
cdn
asset
bundle
download
remote
```

Any remote asset URL should be converted to:

- embedded local assets;
- a local folder;
- `StreamingAssets`;
- or a LAN asset server.

### Offline-development preparation

Archive:

- exact Unity Editor version used successfully;
- Unity build-support modules;
- UPM packages and package cache;
- exact .NET SDK/runtime;
- all NuGet packages;
- Redis package/binary/source;
- complete repository;
- any Git LFS objects;
- any external assets.

### Recommendation

Good for learning a complete Unity + server + Redis flow.

Not a preferred final choice if C/C++ is mandatory.

---

# 4.2 Garuna War

## Source

Server:

https://github.com/eubrunomiguel/garuna

### Stack

- **C++**
- Boost.Asio
- custom UDP networking/protocol
- MySQL Connector
- custom memory management
- project documentation says a custom Unity/C# client was used

### Offline-runtime suitability

The server itself is a good offline candidate because it can operate with local native libraries and a local MySQL database.

### Major issue

A public repository containing the complete Unity client was not verified.

Therefore this project does **not** meet the full "download both client and server" requirement as cleanly as GameProject3.

### What to archive for server study

- complete Git repository;
- exact compiler;
- CMake/build tools if required;
- Boost source package;
- MySQL Connector/C++;
- MySQL server package;
- Visual C++ runtime if using Windows;
- Boost.Asio documentation;
- network packet/protocol notes.

### Recommendation

Use it as a **C++ multiplayer networking reference**, especially for UDP design.

Do not make it the main game base unless you are prepared to implement/replace the client.

---

# 4.3 Pirate Panic

## Source

https://github.com/heroiclabs/unity-sampleproject

The repository includes:

```text
PiratePanic/
ServerModules/
```

### Stack

**Client**

- Unity
- C#
- Nakama Unity SDK

**Server**

- Nakama
- TypeScript server modules/RPCs

**Infrastructure**

- Docker/Compose in the documented setup
- local Nakama endpoint
- PostgreSQL-compatible database depending on the selected deployment

### Offline-runtime suitability

**Strong if fully self-hosted.**

The sample is configured around a local Nakama endpoint such as:

```text
127.0.0.1:7350
```

Therefore a LAN deployment can use:

```text
192.168.10.10:7350
```

instead of an Internet host.

### Air-gap preparation

If you use Docker, never assume the isolated machine can pull images.

While online:

```bash
docker pull <nakama-image>:<PINNED_TAG>
docker pull postgres:<PINNED_TAG>
```

Export:

```bash
docker save -o nakama-image.tar <nakama-image>:<PINNED_TAG>
docker save -o postgres-image.tar postgres:<PINNED_TAG>
```

Offline:

```bash
docker load -i nakama-image.tar
docker load -i postgres-image.tar
```

For TypeScript modules:

```bash
cd ServerModules
npm ci
npx tsc
```

Archive:

- `package.json`
- `package-lock.json`
- npm cache
- Node.js installer/binary
- compiled output
- Nakama image/binary
- DB image/binary

### Recommendation

Excellent self-hosted multiplayer reference.

Not preferred if C/C++ server language is mandatory.

---

# 4.4 Boss Room

## Source

https://github.com/Unity-Technologies/com.unity.multiplayer.samples.coop

### Stack

- Unity 6 LTS generation
- C#
- Netcode for GameObjects
- Unity Transport
- NetworkObjects
- RPCs
- host/client game flow
- current sample integrates Unity Multiplayer Services concepts including Sessions and Authentication

### Why it is useful

Boss Room demonstrates:

- replicated objects;
- RPCs;
- latency masking;
- server-authoritative patterns;
- network object pooling;
- reconnection/session logic;
- client/server gameplay flow.

### Offline-runtime issue

Modern Boss Room is not the best "clone and immediately air-gap" project because the project documentation includes integration with Unity services.

For strict offline use, identify and replace/disable code involving:

- Authentication;
- Sessions;
- Lobby;
- Relay;
- analytics;
- telemetry;
- cloud matchmaking.

Keep LAN transport/local server logic.

### Git LFS requirement

Boss Room uses Git LFS.

Before disconnecting:

```bash
git clone https://github.com/Unity-Technologies/com.unity.multiplayer.samples.coop.git
cd com.unity.multiplayer.samples.coop

git lfs fetch --all
git lfs checkout
```

Verify no LFS pointer files remain in place of real assets.

### Recommendation

Excellent **multiplayer architecture reference**.

Not the first choice for a strict air-gapped final game unless cloud-service pieces are removed.

---

# 4.5 OpenEmpires

## Source

https://github.com/Chilly5/OpenEmpires

### Game type

Open-source Age-of-Empires-inspired RTS.

This is the closest project in the supplied list to the intended **strategy/base-building** direction.

### Stack

**Client**

- Unity 6
- C#
- URP
- New Input System

**Server**

- Rust
- Axum
- Tokio
- WebSockets

**Database**

- PostgreSQL

**Multiplayer**

- deterministic lockstep;
- dynamic input delay;
- fixed-point Q16.16 math.

### Offline-runtime suitability

**Strong.**

Target topology:

```text
Unity Client 1 ----\
Unity Client 2 -----+---- LAN WebSocket ---- Rust Server ---- PostgreSQL
Unity Client 3 ----/                            |
                                                +-- local only
```

The MaxMind geolocation database described by the project is optional; avoid external geolocation calls in an air-gapped build.

### Cargo preparation

While online:

```bash
cd backend
cargo fetch
cargo vendor vendor/
```

Keep:

```text
Cargo.toml
Cargo.lock
vendor/
.cargo/config.toml
```

Configure Cargo source replacement to use `vendor/`.

Then test:

```bash
cargo build --offline
```

Do not accept the archive until that command succeeds with WAN disabled.

### Rust documentation

Rust installed through `rustup` includes local documentation:

```bash
rustup doc
rustup doc --book
```

This makes Rust unusually convenient for isolated development.

### Recommendation

**Best gameplay foundation** for an RTS/Clash-of-Clans-like direction in this list.

If C++ is required later, use OpenEmpires to study gameplay/simulation and port server responsibilities incrementally.

---

# 4.6 San Andreas Unity

## Source

https://github.com/in0finite/SanAndreasUnity

### Stack

- Unity
- C#
- multiplayer
- host mode
- dedicated server mode
- Mono / IL2CPP supported by the project

### Offline-runtime suitability

Strong.

The project supports:

- dedicated server;
- host mode;
- client connecting to server.

This is well suited to a private LAN.

### Important dependency

The project does not redistribute GTA San Andreas content.

It asks the user for a path to a legitimate GTA installation.

Therefore an offline archive must include **your legally obtained game installation/data**, but that proprietary data should not be redistributed with your own open-source archive.

### Clone preparation

The project documentation calls for submodules, so use:

```bash
git clone --recurse-submodules https://github.com/in0finite/SanAndreasUnity.git
```

Then:

```bash
git submodule update --init --recursive
```

### Recommendation

Useful for:

- Unity multiplayer architecture;
- dedicated-server mode;
- moddable game architecture.

Less suitable as a clean foundation for a new distributable game because it depends on proprietary GTA game data.

---

# 4.7 arlozen/MMORPG

## Source

https://github.com/arlozen/MMORPG

This is a monorepo with:

```text
MMORPG/
SERVER/
Tools/
```

### Stack

**Client**

- Unity `6000.0.56f1`
- C#
- QFramework
- MVC
- state-machine-driven entities

**Server**

- C#
- custom network API/framework
- Protobuf
- Serilog

**Database**

- MySQL

### Useful offline feature

The repository documents network configuration in:

```text
SERVER\Common\Network\NetConfig.cs
```

The server address can therefore be explicitly set to a LAN address.

Example:

```text
192.168.10.10
```

### Local topology

```text
Unity Client
    |
    | LAN
    v
C# GameServer
    |
    v
Local MySQL
```

### Data generation

The repository also documents generated JSON/config data.

Run all generation steps before disconnection and archive:

- source Excel/config files;
- generated JSON;
- generated C#;
- Protobuf definitions;
- generated Protobuf C#.

### Offline-development preparation

Archive:

- Unity `6000.0.56f1`;
- exact Unity modules;
- UPM cache;
- exact .NET SDK/runtime;
- NuGet packages;
- `protoc` if you may regenerate protocol code;
- MySQL installer/package;
- DB initialization/config;
- any external Unity plugins such as Odin-related dependencies required by the project.

### Recommendation

One of the cleanest projects for building a **fully local lab**.

Its main mismatch is the C# server.

---

# 4.8 UnityMMO + SkynetMMO

## Sources

Client:

https://github.com/liuhaopen/UnityMMO

Server:

https://github.com/liuhaopen/SkynetMMO

Resources:

https://github.com/liuhaopen/UnityMMO-Resource

### Stack

**Client**

- Unity `2019.4.28f1`
- C#
- Lua/xLua
- custom Lua ECS
- AssetBundles/resources

**Server**

- Skynet
- Lua game services
- Skynet native runtime core in C
- Sproto
- MySQL
- Linux server

### Why it matters

This project is highly relevant to the original requirements because the server architecture combines:

```text
Unity
 C# + Lua
     |
   Sproto
     |
   Skynet
 C runtime + Lua services
     |
   MySQL
```

It is also from the Chinese open-source game ecosystem.

### Correct clone procedure

Client:

```bash
git clone https://github.com/liuhaopen/UnityMMO.git --recurse
```

Server:

```bash
git clone https://github.com/liuhaopen/SkynetMMO.git --recurse
```

Then verify:

```bash
git submodule status --recursive
```

### Resource preparation

The UnityMMO README says large resources are maintained separately because of their size/churn.

Copy the required resource files from:

https://github.com/liuhaopen/UnityMMO-Resource

into the locations expected by the client.

### Licensed-plugin risk

The UnityMMO README also says some plugins are not uploaded because of licensing/copyright restrictions.

This is a major air-gap issue.

Before disconnecting:

1. identify every missing plugin;
2. determine whether it is necessary for your selected build path;
3. obtain it legally;
4. preserve installer/package/license information;
5. test a clean build with WAN disconnected.

### Server setup

The SkynetMMO README documents:

```bash
cd SkynetMMO/skynet
make linux
```

and MySQL databases for:

```text
UnityMMOAccount
UnityMMOGame
```

The server is then started locally.

### Recommendation

**Best project in the list for studying a Chinese C-core/Lua server architecture with Unity.**

More difficult to preserve than GameProject3 because of resources, submodules, Linux server requirements, and missing licensed plugins.

---

# 4.9 GameProject3 + DemoClient

## Sources

Server:

https://github.com/ylmbtm/GameProject3

Unity client:

https://github.com/ylmbtm/DemoClient

### Server stack

- **C++**
- Socket API implementation
- Boost.Asio implementation
- libuv implementation
- shared memory
- lock-free queues
- object pools
- memory pools
- MySQL

### Server roles documented by the project

The project contains multiple server responsibilities such as:

```text
LoginServer
AccountServer
CenterServer
LogicServer
GameServer
DBServer
ProxyServer
LogServer
WatchServer
```

This is a useful architecture for studying a traditional distributed MMORPG backend.

### Client

The separate DemoClient is a Unity project.

The GameProject3 README describes game/client functionality including resources, mounts, pets/companions, equipment, dungeon/combat gameplay, and multiplayer functionality.

### Build notes from the project

The server README documents:

**Windows**

- Visual Studio 2017 or newer

**Linux**

- `buildall.sh`

**Database**

- MySQL
- project recommends MySQL 5.7 for its original setup
- `db_create.sql` initializes tables

**Server startup**

- `StartServer.bat` exists under the server directory for the documented Windows flow.

### Why this is the strongest technical match

It is the closest supplied project to:

```text
Unity Client
     |
     | LAN
     v
C++ Game Server
     |
     v
Local MySQL
```

And unlike Garuna, a public Unity demo-client repository is explicitly provided.

### Air-gap preparation

Because this is an older codebase, do not simply install the newest dependencies.

Instead:

1. Build it successfully online once.
2. Record the exact Git commit.
3. Record compiler version.
4. Record exact Boost version.
5. Record exact libuv version.
6. Record MySQL server and connector versions.
7. Archive Visual C++ redistributables.
8. Archive all client packages/plugins.
9. Test the complete build with network access disabled.

### Recommendation

**First choice when C++ server + Unity client is the main technical requirement.**

---

# 5. Selection Matrix

| Priority | Recommended Project | Why |
|---|---|---|
| C++ server + Unity client | **GameProject3 + DemoClient** | Best direct match among supplied projects |
| C/C++ architecture study | **GameProject3**, then **Garuna** | Native networking/server code |
| Chinese game-server architecture | **UnityMMO + SkynetMMO** | Skynet C core + Lua services + Unity client |
| RTS / Clash-of-Clans-like direction | **OpenEmpires** | Strategy gameplay, resources, construction, deterministic multiplayer |
| Simple complete LAN lab | **arlozen/MMORPG** | Explicit server-IP config, Unity + local C# server + MySQL |
| Multiplayer patterns | **Boss Room** | Strong RPC/replication/server-authority reference |
| Self-hosted backend services | **Pirate Panic** | Local Nakama, auth, matchmaking, realtime multiplayer |
| Dedicated-server Unity architecture | **San Andreas Unity** | Client/host/dedicated server modes |
| Lightweight MMO lab | **Mobile MMORPG** | Unity + .NET + Redis with straightforward local topology |

---

# 6. Recommended Final Architecture

For a long-term offline project, use a topology like:

```text
                         PUBLIC INTERNET
                               X
                               X  BLOCKED
                               X

                      +------------------+
                      | Isolated LAN     |
                      | Switch / Router  |
                      | No WAN route     |
                      +---------+--------+
                                |
            +-------------------+--------------------+
            |                                        |
   +--------+---------+                     +--------+---------+
   | Unity/Cocos      |                     | Game Server      |
   | Client           | <---- LAN only ---->| C/C++ preferred |
   +------------------+                     +--------+---------+
                                                      |
                              +-----------------------+------------------+
                              |                       |                  |
                       +------+-----+          +------+-----+     +------+------+
                       | MySQL /   |          | Redis      |     | Local Assets |
                       | PostgreSQL|          | optional   |     | / Config     |
                       +------------+          +------------+     +-------------+
```

### Client firewall intent

Allow:

- game server;
- local asset/config server;
- optional local DNS.

Deny:

- Internet;
- public DNS;
- cloud authentication;
- analytics endpoints;
- remote CDNs.

### Server firewall intent

Allow:

- game-client subnet;
- local DB/cache;
- local admin/monitoring machine.

Deny:

- WAN/public Internet.

---

# 7. What "Offline Ready" Must Mean

A project should pass **all four** stages.

## Stage 1 – Source available

- main repo downloaded;
- separate client/server repos downloaded;
- correct branches/tags preserved;
- submodules downloaded;
- Git LFS downloaded;
- external resource repos downloaded.

## Stage 2 – Dependencies available

- compiler/runtime;
- Unity editor;
- Unity modules;
- UPM packages;
- NuGet;
- npm;
- Cargo crates;
- native C/C++ libraries;
- database;
- asset plugins;
- licenses.

## Stage 3 – Clean rebuild succeeds offline

Delete generated/build/cache folders that should not be required.

Disconnect WAN.

Then rebuild client and server.

If any package manager tries to contact the Internet, the environment is not yet complete.

## Stage 4 – Runtime succeeds offline

With WAN disconnected, test:

- account/login;
- character/player creation;
- world loading;
- asset loading;
- matchmaking;
- combat;
- save;
- database persistence;
- reconnect;
- server restart;
- multiple clients.

---

# 8. Internet-Dependency Source Audit

Before accepting a project, search its source for remote dependencies.

With `ripgrep`:

```bash
rg -n -i \
'https?://|wss?://|api\.|cdn|telemetry|analytics|firebase|googleapis|unity3d|cloud|sentry|discord|steam|oauth' \
.
```

Search configuration terms:

```bash
rg -n -i \
'serverip|hostname|host|endpoint|url|remote|assetbundle|addressables|relay|lobby|auth|matchmaker|patch|update' \
.
```

For Unity projects inspect:

```text
Assets/
Packages/manifest.json
Packages/packages-lock.json
ProjectSettings/
StreamingAssets/
Resources/
Addressables/
```

Common hidden Internet requirements:

- remote Addressables catalogs;
- remote AssetBundles;
- patch/update servers;
- Unity Authentication;
- Unity Lobby;
- Unity Relay;
- Unity Analytics;
- Firebase;
- crash reporting;
- telemetry;
- OAuth;
- Google/Apple game services;
- social login;
- remote configuration;
- public DNS;
- CDN-hosted configuration;
- remote license/plugin checks.

---

# 9. Git and Source Preservation

## Clone recursively

For repositories with submodules:

```bash
git clone --recurse-submodules <REPOSITORY>
```

For an existing clone:

```bash
git submodule update --init --recursive
```

Record:

```bash
git rev-parse HEAD
git submodule status --recursive
```

Save these values in your offline manifest.

---

## Git LFS

For LFS projects:

```bash
git lfs fetch --all
git lfs checkout
```

Important:

A normal Git bundle does **not** automatically replace preservation of separate Git LFS objects.

---

## Create Git bundles

Git bundles are explicitly designed to move Git objects/refs without an active server.

Create:

```bash
git bundle create project.bundle --all
```

Verify:

```bash
git bundle verify project.bundle
```

Test:

```bash
git clone project.bundle project-offline-test
```

Keep the original working tree plus bundle.

---

# 10. Unity Air-Gap Preparation

Unity is likely to be the largest offline-development risk.

## 10.1 Install exact editor versions

Do not use "latest Unity" unless the project is known to support it.

Examples:

- arlozen/MMORPG: `6000.0.56f1`
- UnityMMO: `2019.4.28f1`
- OpenEmpires: use the version declared by its current repository
- Boss Room: use its supported Unity 6 LTS version
- San Andreas Unity: preserve a version tested against the selected project commit

Archive installers/modules for:

- Windows Build Support;
- Linux Build Support;
- Android Build Support;
- IL2CPP;
- Android SDK/NDK/JDK;
- any other required target.

---

## 10.2 Populate Unity Package Manager cache

Open every selected Unity project **before disconnecting**.

Wait until:

- package resolution completes;
- scripts compile;
- package imports finish;
- scenes load;
- a standalone build succeeds.

Preserve:

```text
Packages/manifest.json
Packages/packages-lock.json
```

Unity Package Manager uses a global cache for downloaded package contents and metadata.

Typical UPM cache locations include:

### Windows

```text
%LOCALAPPDATA%\Unity\cache\upm
```

### macOS

```text
~/Library/Caches/Unity/upm
```

### Linux

```text
~/.cache/Unity/upm
```

Back up the complete relevant Unity cache.

For embedded/local Unity packages, also preserve the package `Documentation~` directories because Unity can expose those Markdown docs while offline.

---

## 10.3 Unity licensing

This must be solved **before** air-gapping the development machine.

Unity documentation describes manual license activation for limited-connectivity/air-gapped environments, but manual activation requires an Internet-connected step that can be performed on another machine.

Unity documentation also states that the manual activation method is **not supported for Unity Personal licenses**.

Therefore distinguish:

### Built game

A properly designed standalone Unity build can run offline.

### Unity Editor

A permanently air-gapped development machine requires a license/activation plan compatible with your Unity license.

Do not disconnect the development environment until this has been tested.

---

# 11. C/C++ Offline Preparation

For GameProject3/Garuna, archive:

```text
compiler/
cmake/
ninja-or-make/
boost/
libuv/
mysql-connector/
protobuf-if-used/
debugger/
runtime-redistributables/
third-party-source/
licenses/
documentation/
```

Prefer source archives plus binaries.

For every third-party library record:

```text
Name:
Version:
Source URL:
License:
SHA256:
Build options:
Compiler:
Architecture:
```

Example:

```text
Boost
Version: <PINNED>
Compiler: MSVC <PINNED>
Architecture: x64
Purpose: Asio/networking
```

Avoid relying on the latest upstream package.

---

# 12. .NET / NuGet Offline Preparation

Applicable to:

- Mobile MMORPG
- arlozen/MMORPG
- other C# server/client tooling

While online:

```bash
dotnet restore
```

Inspect local caches:

```bash
dotnet nuget locals all --list
```

For robust offline use, create a local NuGet source containing all needed `.nupkg` files and use a local `NuGet.Config`.

Example:

```xml
<configuration>
  <packageSources>
    <clear />
    <add key="OfflinePackages" value="D:\OfflineGameKit\nuget-feed" />
  </packageSources>
</configuration>
```

Then test:

```bash
dotnet restore --ignore-failed-sources
```

with WAN blocked.

Archive:

- exact .NET SDK;
- runtime;
- NuGet packages;
- global tools;
- workload packs if used.

---

# 13. Rust / Cargo Offline Preparation

Applicable to OpenEmpires.

Fetch:

```bash
cargo fetch
```

Vendor dependencies:

```bash
cargo vendor vendor/
```

Cargo prints source-replacement configuration for `.cargo/config.toml`.

Preserve:

```text
Cargo.toml
Cargo.lock
vendor/
.cargo/config.toml
```

Validate:

```bash
cargo build --offline
```

Rust local documentation:

```bash
rustup doc
rustup doc --book
```

---

# 14. Node/npm Offline Preparation

Applicable to Pirate Panic's TypeScript server modules and any other Node tooling.

While online:

```bash
npm ci
```

Preserve:

```text
package.json
package-lock.json
npm cache
Node.js installer/runtime
compiled TypeScript output
```

Where practical, maintain an internal/local npm cache or local registry.

Test package install/build on the isolated machine before declaring the archive complete.

---

# 15. Docker Offline Preparation

Docker is convenient but can hide Internet dependencies.

Never leave image names as unpinned `latest`.

Online:

```bash
docker pull <image>:<exact-tag>
```

Save:

```bash
docker save -o images.tar \
  <image1>:<exact-tag> \
  <image2>:<exact-tag>
```

Offline:

```bash
docker load -i images.tar
docker image ls
```

Archive:

- Docker Engine/Desktop installer;
- Docker Compose;
- image tar files;
- Compose YAML;
- volumes/data initialization;
- image digests.

Test the complete stack without WAN.

---

# 16. Database Preparation

## MySQL

Used by:

- Garuna
- arlozen/MMORPG
- UnityMMO/SkynetMMO
- GameProject3

Archive:

- installer/server package;
- client tools;
- exact major/minor version;
- connector library;
- SQL schema;
- seed data;
- backup;
- restore script;
- PDF/manual.

For GameProject3, the original documentation recommends MySQL 5.7. If you modernize it, test compatibility before isolation rather than assuming MySQL 8.x is drop-in compatible.

---

## Redis

Used by Mobile MMORPG.

Archive:

- server binary/source;
- configuration;
- persistence settings;
- CLI;
- backup/restore notes.

---

## PostgreSQL

Used by OpenEmpires and commonly by self-hosted Nakama deployments.

Archive:

- exact server package;
- CLI tools;
- SQL/schema;
- roles/users creation script;
- backup/restore scripts;
- PDF documentation matching the installed major version.

---

# 17. Recommended Offline Folder Structure

```text
OfflineGameKit/
|
+-- 00-manifest/
|   +-- versions.md
|   +-- checksums.sha256
|   +-- licenses.md
|
+-- 01-repositories/
|   +-- MobileMMORPG/
|   +-- Garuna/
|   +-- PiratePanic/
|   +-- BossRoom/
|   +-- OpenEmpires/
|   +-- SanAndreasUnity/
|   +-- arlozen-MMORPG/
|   +-- UnityMMO/
|   +-- SkynetMMO/
|   +-- UnityMMO-Resource/
|   +-- GameProject3/
|   +-- DemoClient/
|
+-- 02-git-bundles/
|
+-- 03-unity/
|   +-- editors/
|   +-- modules/
|   +-- upm-cache/
|   +-- plugins/
|   +-- documentation/
|
+-- 04-cpp/
|   +-- compiler/
|   +-- cmake/
|   +-- boost/
|   +-- libuv/
|   +-- protobuf/
|   +-- mysql-connector/
|
+-- 05-dotnet/
|   +-- sdk/
|   +-- runtime/
|   +-- nuget-feed/
|
+-- 06-rust/
|   +-- toolchain/
|   +-- vendor/
|   +-- documentation/
|
+-- 07-node/
|   +-- node/
|   +-- npm-cache/
|
+-- 08-databases/
|   +-- mysql/
|   +-- redis/
|   +-- postgresql/
|
+-- 09-docker/
|   +-- installers/
|   +-- images/
|   +-- compose/
|
+-- 10-docs/
|   +-- project-readmes/
|   +-- html/
|   +-- pdf/
|   +-- language-docs/
|
+-- 11-sql/
|
+-- 12-builds/
|
+-- 13-network-tools/
|
+-- README-OFFLINE.md
```

---

# 18. Recommended Documentation & PDFs

The best offline archive includes **project-specific docs** and **general reference docs**.

---

## 18.1 Beej's Guide to Network Programming — PDF

### Priority

**Very high for C/C++ server development.**

Official guide:

https://beej.us/guide/bgnet/

Official PDF directory:

https://beej.us/guide/bgnet/pdf/

Why save it:

- sockets;
- TCP;
- UDP;
- IPv4/IPv6;
- client/server patterns;
- address resolution;
- multiplexing concepts.

Useful for:

- GameProject3;
- Garuna;
- any future custom C/C++ server.

Recommended file: choose the current A4 or US Letter PDF from the official PDF directory.

---

## 18.2 MySQL Getting Started — official PDF

Direct official PDF:

https://downloads.mysql.com/docs/mysql-getting-started-en.pdf

Useful for:

- local installation;
- starting/stopping MySQL;
- local client access;
- initial SQL operations.

This is a small PDF and should definitely be kept.

---

## 18.3 MySQL Reference Manual

Official documentation:

https://dev.mysql.com/doc/

MySQL downloadable documentation area:

https://downloads.mysql.com/docs/

Save the **reference manual matching the exact MySQL major version you install**.

This is especially important for GameProject3 if you follow its original MySQL 5.7 recommendation.

---

## 18.4 PostgreSQL Manual — PDF

Official documentation:

https://www.postgresql.org/docs/

PostgreSQL distributes version-specific PDF manuals.

Example current-major PDF pattern:

```text
https://www.postgresql.org/files/documentation/pdf/<MAJOR>/postgresql-<MAJOR>-A4.pdf
```

For a PostgreSQL 18 archive:

https://www.postgresql.org/files/documentation/pdf/18/postgresql-18-A4.pdf

Always download the PDF matching your installed major version.

Useful for:

- OpenEmpires;
- self-hosted Nakama/Pirate Panic deployments.

---

## 18.5 Lua Reference Manual — official offline documentation

Official manuals:

https://www.lua.org/manual/

The Lua site explicitly provides manuals for offline use.

For Lua 5.4:

https://www.lua.org/manual/5.4/

Download area:

https://www.lua.org/ftp/

Useful for:

- UnityMMO;
- SkynetMMO;
- Lua/C API;
- scripting game services.

**Important:** use documentation matching the Lua version embedded by the selected Skynet commit rather than blindly choosing the newest Lua release.

---

## 18.6 Rust Book — included offline with rustup

Online reference:

https://doc.rust-lang.org/book/

Once Rust is installed:

```bash
rustup doc
rustup doc --book
```

The Rust installation includes local documentation.

Useful for:

- OpenEmpires;
- standard library;
- Cargo;
- ownership/concurrency;
- async Rust.

Cargo Book:

https://doc.rust-lang.org/cargo/

Cargo vendoring:

https://doc.rust-lang.org/cargo/commands/cargo-vendor.html

Cargo source replacement:

https://doc.rust-lang.org/cargo/reference/source-replacement.html

---

## 18.7 Boost.Asio documentation

Official:

https://www.boost.org/doc/libs/latest/doc/html/boost_asio.html

Save the complete Boost source release corresponding to the version you use because its documentation is included with the distribution.

Useful for:

- Garuna;
- GameProject3;
- future native C++ networking.

Important topics to keep:

- `io_context`;
- TCP sockets;
- UDP sockets;
- asynchronous read/write;
- timers;
- strand/executor concepts;
- error handling.

---

## 18.8 CMake documentation

Official:

https://cmake.org/documentation/

CMake also has command-line local help:

```bash
cmake --help
cmake --help-command-list
cmake --help-module-list
cmake --help-property-list
cmake --help-variable-list
```

Save the documentation corresponding to the exact CMake release in your archive.

---

## 18.9 Protocol Buffers documentation

Official:

https://protobuf.dev/

Source:

https://github.com/protocolbuffers/protobuf

Archive:

- exact `protoc`;
- `.proto` files;
- C#/C++ runtime libraries;
- language guide;
- encoding guide.

Useful for:

- arlozen/MMORPG;
- future C++ ↔ Unity protocol designs.

---

## 18.10 Git bundle documentation

Official:

https://git-scm.com/docs/git-bundle

Why it matters:

Git specifically supports bundles for moving Git objects and refs without an active remote server.

Keep a local copy because it is useful for transferring repository history into an air-gapped machine.

---

## 18.11 Unity documentation

### Project-version rule

Always save documentation for the **same Unity version** used by each project.

Do not rely on current/latest docs when maintaining:

- Unity 2019;
- Unity 2020;
- Unity 6;
- older package versions.

Unity historically provides offline documentation packages for supported editor generations, and installed/local packages can also carry Markdown documentation under:

```text
Documentation~/
```

Important docs to preserve:

- Unity Manual;
- Scripting API;
- Package Manager;
- Netcode for GameObjects;
- Unity Transport;
- build pipeline;
- dedicated server;
- Addressables if used;
- AssetBundles;
- IL2CPP;
- player settings;
- command-line arguments.

Unity global Package Manager cache documentation:

https://docs.unity3d.com/6000.0/Documentation/Manual/upm-cache.html

Unity manual license activation documentation:

https://docs.unity3d.com/Manual/ManualActivationGuide.html

---

## 18.12 Nakama documentation

Official:

https://heroiclabs.com/docs/nakama/

Docker Compose/self-hosted setup:

https://heroiclabs.com/docs/nakama/getting-started/install/docker/

Useful sections:

- local install;
- configuration;
- client authentication;
- socket/realtime;
- matchmaking;
- TypeScript runtime;
- persistence;
- local Docker deployment.

Because web docs can change, preserve a local HTML/PDF print of the pages that match the Nakama version you pin.

---

## 18.13 Skynet documentation

Source:

https://github.com/cloudwu/skynet

Wiki:

https://github.com/cloudwu/skynet/wiki

Archive both the source repository and Wiki pages.

Important topics:

- service model;
- message dispatch;
- sockets;
- Lua services;
- cluster;
- database access;
- debugging.

---

# 19. PDF / Documentation Download Priority

If you have limited time, save these first:

| Priority | Document | Why |
|---|---|---|
| 1 | **Beej's Guide to Network Programming PDF** | Essential C/C++ TCP/UDP reference |
| 2 | **Database manual matching your exact DB version** | Critical for offline DB recovery/config |
| 3 | **Unity Manual + Scripting API matching exact editor** | Client development without Internet |
| 4 | **Project README/Wiki + source docs** | Exact build/run configuration |
| 5 | **Boost.Asio docs** | C++ networking for GameProject3/Garuna |
| 6 | **CMake docs** | Native build troubleshooting |
| 7 | **Lua manual + Skynet wiki** | UnityMMO/SkynetMMO |
| 8 | **Rust Book + Cargo docs** | OpenEmpires |
| 9 | **Protobuf docs** | arlozen/MMORPG and custom protocols |
| 10 | **Git bundle docs** | Moving/versioning code in isolated networks |
| 11 | **Nakama docs** | Pirate Panic local backend |
| 12 | **Docker docs for save/load/Compose** | Preserve local service images |

---

# 20. Recommended Network / Debug Tools to Download

These are valuable in an offline lab:

- Wireshark
- tcpdump
- `ss`
- `netstat`
- Windows TCPView / Sysinternals
- Process Explorer
- `curl`
- `openssl`
- `nmap` if allowed in your environment
- `dig` / `nslookup`
- Redis CLI
- MySQL CLI
- PostgreSQL `psql`
- packet capture documentation

Use them to prove that the client/server are not silently calling public services.

---

# 21. How to Prove the Game Is Truly Offline

## Step 1 – Remove WAN

Best test:

- use an isolated switch;
- or disable the WAN interface;
- or configure a router with no Internet route.

Do not only rely on "the Internet appears disconnected."

---

## Step 2 – Confirm LAN communication

Example:

```bash
ping <SERVER_IP>
```

Client → server should work.

Public Internet should fail.

---

## Step 3 – Run packet capture

Linux:

```bash
ss -tunap
sudo tcpdump -i any
```

Windows:

```powershell
Get-NetTCPConnection
```

Also inspect with Wireshark/TCPView.

---

## Step 4 – Test every major flow

- start DB;
- start cache;
- start server;
- start client;
- log in;
- create/load player;
- load assets;
- enter world;
- start match/dungeon;
- save;
- disconnect;
- reconnect;
- restart server;
- reload data.

All remote destinations should be:

```text
127.0.0.1
::1
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

or another explicitly approved private/local address.

---

# 22. Clean-Room Offline Build Test

This is the most important validation step.

Create a new VM or separate machine.

Do **not** give it Internet access.

Copy only your `OfflineGameKit`.

Then attempt:

1. Install toolchain.
2. Restore dependencies from local archive.
3. Build server.
4. Install database.
5. Import schema.
6. Open Unity project.
7. Restore Unity packages from local cache.
8. Build client.
9. Run client/server.
10. Capture traffic.

Anything that fails identifies something still missing from the offline kit.

Do this **before** the original online machine is permanently isolated.

---

# 23. Air-Gap Readiness Checklist

## Source

- [ ] Main repository cloned.
- [ ] Client repository cloned if separate.
- [ ] Server repository cloned if separate.
- [ ] Correct commit hash recorded.
- [ ] Tags/branches recorded.
- [ ] Submodules downloaded recursively.
- [ ] Git LFS fully fetched.
- [ ] Separate asset/resource repositories downloaded.
- [ ] Missing commercial plugins legally acquired.
- [ ] Git bundles created.
- [ ] SHA-256 manifest created.

## Unity

- [ ] Exact Unity editor installed.
- [ ] Required target modules installed.
- [ ] Unity license plan tested.
- [ ] Project opens with WAN blocked.
- [ ] Scripts compile.
- [ ] UPM packages resolved.
- [ ] UPM cache copied.
- [ ] `manifest.json` saved.
- [ ] `packages-lock.json` saved.
- [ ] External plugins saved.
- [ ] Local client build succeeds.

## C/C++

- [ ] Compiler archived.
- [ ] CMake/build system archived.
- [ ] Boost archived.
- [ ] libuv archived if needed.
- [ ] DB connectors archived.
- [ ] runtime DLLs/shared libraries archived.
- [ ] licenses archived.
- [ ] native server clean build succeeds offline.

## .NET

- [ ] SDK archived.
- [ ] runtime archived.
- [ ] NuGet packages/local feed archived.
- [ ] server builds with Internet disabled.

## Rust

- [ ] Rust toolchain archived/installed.
- [ ] `Cargo.lock` preserved.
- [ ] `cargo vendor` completed.
- [ ] `cargo build --offline` succeeds.
- [ ] local Rust docs installed.

## Node

- [ ] Node runtime archived.
- [ ] `package-lock.json` preserved.
- [ ] dependencies/cache archived.
- [ ] TypeScript compilation succeeds offline.

## Database

- [ ] Database installer archived.
- [ ] Exact version recorded.
- [ ] Schema SQL saved.
- [ ] Seed/test data saved.
- [ ] Backup created.
- [ ] Restore tested.
- [ ] CLI client archived.
- [ ] matching documentation/PDF saved.

## Runtime

- [ ] Client connects only to LAN server.
- [ ] Server starts without WAN.
- [ ] Login works.
- [ ] Game world loads.
- [ ] Asset loading works.
- [ ] Match/combat works.
- [ ] Save/load works.
- [ ] Reconnect works.
- [ ] Restart/recovery works.
- [ ] No unexpected public network traffic appears.

---

# 24. Recommended Order of Evaluation

Do not try to prepare all nine projects at the same depth initially.

### Phase 1 — GameProject3

Goal:

Prove that a C++ game server + Unity client can be built and run fully offline.

Download:

- GameProject3
- DemoClient
- compiler
- Boost
- libuv
- MySQL
- Unity editor/packages
- networking/database docs

### Phase 2 — OpenEmpires

Goal:

Study strategy-game simulation and multiplayer architecture.

Download:

- repository
- Unity version
- Rust toolchain
- vendored crates
- PostgreSQL
- Rust/Cargo/PostgreSQL docs

### Phase 3 — UnityMMO/SkynetMMO

Goal:

Study Chinese online-game server architecture.

Download:

- UnityMMO
- SkynetMMO
- resource repository
- every submodule
- required licensed plugins
- MySQL
- Lua manuals
- Skynet wiki
- Unity 2019.4.28f1 environment

### Phase 4 — Boss Room

Goal:

Study Unity replication/RPC/server-authority patterns.

Do not depend on its cloud integrations for the final air-gapped system.

---

# 25. Final Recommendation

## Recommended primary base: GameProject3 + DemoClient

Choose this when the highest priorities are:

- real game code;
- C++ server;
- Unity client;
- local database;
- LAN/offline operation.

Server:

https://github.com/ylmbtm/GameProject3

Client:

https://github.com/ylmbtm/DemoClient

---

## Recommended gameplay reference: OpenEmpires

Use it to study:

- strategy economy;
- resources;
- construction;
- unit simulation;
- deterministic multiplayer.

Source:

https://github.com/Chilly5/OpenEmpires

---

## Recommended China/server-architecture reference: UnityMMO + SkynetMMO

Use it to study:

- Skynet services;
- C-native runtime;
- Lua game services;
- Sproto;
- hot-update-friendly Lua design;
- Unity + Lua MMO architecture.

Client:

https://github.com/liuhaopen/UnityMMO

Server:

https://github.com/liuhaopen/SkynetMMO

---

# 26. Reference Links

## Games

- Mobile MMORPG: https://github.com/Ziden/MobileMMORPG
- Garuna: https://github.com/eubrunomiguel/garuna
- Pirate Panic: https://github.com/heroiclabs/unity-sampleproject
- Boss Room: https://github.com/Unity-Technologies/com.unity.multiplayer.samples.coop
- OpenEmpires: https://github.com/Chilly5/OpenEmpires
- San Andreas Unity: https://github.com/in0finite/SanAndreasUnity
- arlozen/MMORPG: https://github.com/arlozen/MMORPG
- UnityMMO: https://github.com/liuhaopen/UnityMMO
- SkynetMMO: https://github.com/liuhaopen/SkynetMMO
- UnityMMO resources: https://github.com/liuhaopen/UnityMMO-Resource
- GameProject3: https://github.com/ylmbtm/GameProject3
- DemoClient: https://github.com/ylmbtm/DemoClient

## Offline/technical documentation

- Beej network guide: https://beej.us/guide/bgnet/
- Beej PDF directory: https://beej.us/guide/bgnet/pdf/
- MySQL Getting Started PDF: https://downloads.mysql.com/docs/mysql-getting-started-en.pdf
- MySQL docs: https://dev.mysql.com/doc/
- PostgreSQL docs: https://www.postgresql.org/docs/
- Lua manuals: https://www.lua.org/manual/
- Rust Book: https://doc.rust-lang.org/book/
- Cargo vendor: https://doc.rust-lang.org/cargo/commands/cargo-vendor.html
- Cargo source replacement: https://doc.rust-lang.org/cargo/reference/source-replacement.html
- Boost.Asio: https://www.boost.org/doc/libs/latest/doc/html/boost_asio.html
- CMake docs: https://cmake.org/documentation/
- Protocol Buffers: https://protobuf.dev/
- Git bundle: https://git-scm.com/docs/git-bundle
- Nakama docs: https://heroiclabs.com/docs/nakama/
- Nakama Docker install: https://heroiclabs.com/docs/nakama/getting-started/install/docker/
- Skynet: https://github.com/cloudwu/skynet
- Skynet Wiki: https://github.com/cloudwu/skynet/wiki
- Unity UPM global cache: https://docs.unity3d.com/6000.0/Documentation/Manual/upm-cache.html
- Unity manual license activation: https://docs.unity3d.com/Manual/ManualActivationGuide.html

---

# 27. Bottom Line

For the original requirements, start with:

```text
#1 GameProject3 + DemoClient
   C++ Server + Unity Client

#2 OpenEmpires
   Best strategy/RTS gameplay reference

#3 UnityMMO + SkynetMMO
   Best Chinese C-core/Lua game-server architecture reference
```

The most important task is not merely downloading the repositories. The goal should be to build an **OfflineGameKit** that contains the exact compiler/editor versions, dependencies, databases, package caches, assets, licenses, documentation, and source history needed to reproduce the project on a completely disconnected machine.

The final acceptance test is a **clean-room build on a machine that never receives Internet access**.
