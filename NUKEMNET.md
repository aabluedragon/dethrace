# NukemNet demo multiplayer build

Version: **0.10.1-nn1**, based on Dethrace v0.10.1
(`9fa3bb2b550e9ab8784134c2ba9781c28c50427b`).
This is an independent modification, not an official Dethrace release.
The original Dethrace licence and bundled dependency notices still apply.
No Carmageddon game data is included in this repository or the engine download.

## Demo editions

Both original Carmageddon demos have separate NukemNet executable profiles:

| Edition | Original archive | Identifying file | SHA-256 of tested archive |
| --- | --- | --- | --- |
| SCi, early 1997 | `carma.zip` | `CARMDEMO.EXE` | `e47f3408feee042a8cc207df6df0a9a62c4734168d446846824ccab06eb20691` |
| Interplay, May 1997 | `carmdemo.zip` | `CARMDEMO.BAT` | `b11f81f57fcd657f00af308e61dd5e5d601069a19b82c0262362773f389576b1` |

Extract each demo into a separate folder. Everyone in a room must use the
same demo edition, NN engine build and architecture. This protocol uses native
C structures; cross-architecture compatibility is not promised. Stock Dethrace,
the original DOS/Windows executables and Splat Pack are not compatible with
these demo rooms. Full-game and Splat Pack behaviour is otherwise left alone.

The engine download is for **Linux x64**, requires glibc 2.29 or newer and SDL2
(`libSDL2-2.0.so.0`). Windows x64 cross-compilation also succeeds, but Windows
gameplay has not been verified for this build and no Windows auto-download is
offered yet. Other platforms can build from source; they are not tested here.

## Using NukemNet

1. Choose Carmageddon, then the SCi or Interplay Dethrace demo in Settings → Games.
2. Download the NN engine, then specify your extracted demo in **Demo data folder**
   when hosting/joining. Alternatively, put the demo's `DATA` directory and
   identifying file beside `dethrace-nn`.
3. Join the same NukemNet room and select the room's connection method. NN carries
   Dethrace discovery and player traffic through its UDP tunnel and relay.
4. Start the game in NN. Host and clients enter the game lobby automatically.
   Wait until everyone is Ready. The host clicks **Start Now** (Interplay) or
   **Start Without Them** (SCi's original button label).
5. Accelerate with numpad 8. The host uses the Eagle; guests automatically select
   an unused included car. There are six available cars and the CITYB3 track.

Do not run two game processes on the same local network interface: Dethrace's
discovery port is UDP 12286. Tests use separate network namespaces. The NN
transport port must be different from 12286.

The SCi demo lacks complete opponent cockpits, so its network players use the
Eagle cockpit/HUD with their assigned car's model and physics. Its older menu
artwork uses the correct clickable areas without later-version button animations.
The original demo timer is retained. The single-player advertisement is skipped
between network races.

## Changes from upstream

- Explicit demo host/join CLI entry points, leaving the ordinary demo menu guard
  intact; automatic free-car selection, nicknames and bounded connection waits.
- Use the demo's included race instead of unavailable full-game network tracks.
- Keep network participants when selecting opponents, rather than replacing them
  with the demo's single-player opponent list.
- Read the early SCi GENERAL, OPPONENT, car material/header and track formats;
  load its original key map, cursor frames and animation directory. No demo data
  rewriting or full-game assets are required.
- Support one-race lists without accessing a negative array index.

The UDP relay adapter lives in NukemNet, not in this engine. NN translates
discovery and embedded player endpoints to local UDP proxies; this engine still
uses Dethrace's original networking and reliability handling.

## CLI and builds

For an NN-controlled host:

```sh
./dethrace-nn --dir /path/to/demo --nn-demo-host --nn-demo-name Host --window
```

For a client behind NN's local discovery proxy:

```sh
./dethrace-nn --dir /path/to/demo --nn-demo-join --no-bind --nn-demo-name Player --window
```

`--nn-demo-car` optionally fixes a car index: 0, 19, 7, 22, 12 or 16. Normally
omit it so the host assigns distinct cars. Names are limited to 13 bytes.
These arguments alone do not establish an Internet relay; NN supplies that part.

```sh
git clone --recursive --branch nn-demo-multiplayer https://github.com/aabluedragon/dethrace.git
cd dethrace
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DDETHRACE_PLATFORM_SDL_DYNAMIC=ON
cmake --build build -j
```

The portable Linux build uses Debian 11, GCC 10, CMake 3.31.6, SDL2 development
headers and dynamic SDL loading. The source archive includes the checked-out
BRender submodule at `44d27ffa026a0a3ea4dc0642315913bf5bc27e25` and all other
bundled build dependencies. SDL2 remains a system runtime dependency.

Optional tests:

```sh
cmake -S . -B build -DBUILD_TESTS=ON
cmake --build build -j
ctest --test-dir build --output-on-failure
```

## Validation and limits

Both SCi and Interplay were tested with two separate NukemNet instances on
Linux x64, entering a checkpoint race and driving both cars with remote movement
visible. Direct client-to-client forwarding was blocked; traffic used NN's UDP
relay, with about 60 ms simulated round-trip delay. Packet captures recorded
game traffic through the relay, with no kernel capture drops.

NukemNet's automated tests exercise all six tunnel slots and guest-to-guest
routing over IPv6 loopback, bounded endpoint rewriting, edition validation and
launch arguments. These are not six-player gameplay tests. Six actual players,
STUN-only connections, public IPv6, cross-platform play, sound quality and
completed race/repeated-race behaviour have not been verified.

The engine's existing unit suite reports 22 tests, 0 failures, 11 ignored without
full-game test data. Linux release and Windows x64 builds compile successfully.
