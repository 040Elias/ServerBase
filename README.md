# Canvas Server Template

> **Canvas Version:** 26.2
> **Required Java Version:** Java 25

This repository contains a preconfigured **Canvas 26.2** server optimized for high performance while preserving vanilla gameplay as much as possible. The configuration was primarily built with **CrystalPvP** in mind but also serves as an excellent base for SMPs and other survival servers.

Canvas is a high-performance fork of Folia. All Folia-compatible plugins work on Canvas, while Canvas also introduces additional APIs and optimizations.

---

# Documentation

Before changing any configuration, it is highly recommended to read the official documentation.

## Canvas

https://docs.canvasmc.io/canvas/introduction

## Paper

https://docs.papermc.io/paper/

Many configuration files and concepts are inherited from Paper, making the Paper documentation applicable to most server configuration options.

**Note on Canvas docs specifically:** Canvas moves fast, and its public documentation can lag behind the actual release you're running — some config keys/formats you'll find in this repo may not match what the current docs show 1:1. If a value here looks undocumented or differently named than what you find online, trust the inline comments in the config file itself first, and treat the docs as a general reference rather than a strict source of truth for every key.

---

# System Requirements

* Java 25
* 64-bit Operating System
* SSD/NVMe storage strongly recommended
* Modern CPU with high single-core performance
* At least 8 GB RAM — **note:** this is a floor, not a target. The included `max-players=250` default in `server.properties` is a placeholder and assumes far more headroom than 8 GB provides. Size your RAM to your actual expected player count, not the other way around.

---

# Installation

1. Install Java 25.
2. Upload the server files.
3. Configure your startup command.
4. Accept the Minecraft EULA.
5. Decide on your proxy setup (see **Proxy Configuration** below) before first launch.
6. Review every configuration file before going live.
7. Start the server.

---

# Updating Plugins

Never overwrite plugins inside the `plugins` folder directly.

Instead:

```text
~/plugins/update
```

Place the new plugin `.jar` inside the `update` folder and restart the server.

This is the recommended update procedure documented by PaperMC (https://docs.papermc.io/paper/updating/). By staging updates here instead of replacing files live, you avoid replacing plugin JARs while the server is running, and all plugin updates apply together on the next restart — this also lets you update Canvas itself and your plugins in the same restart, without extra downtime. This workflow is inherited from Paper's plugin-loading layer; it hasn't been separately verified against Canvas-specific documentation, but there's no indication it behaves differently.

---

# Updating Canvas

Still in Work. Auto Updating (with the Oficial Sculptor) will be used but i am focussing on migrating at the Moemnt

---

# Startup Command

For JVM flags, use the Birdflop Flags Generator instead of asking AI.

Recommended options:

* Platform: Pterodactyl
* Software: Paper
* Server File: server.jar
* No GUI
* Auto Restart

https://www.birdflop.com/resources/flags/

---

# Proxy Configuration

Pick **one** of the two options below before going live. Do not mix settings from both — combining them breaks player connections or opens an authentication hole.

## Option A — Behind Velocity

1. `config/paper-global.yml` → `proxies.velocity`:
   - `enabled: true`
   - `online-mode: true`
   - `secret:` must be identical to the `forwarding-secret` in your Velocity's `velocity.toml`. **This template ships with a `CHANGEME` placeholder here — replace it before going live, since this secret is active as soon as `enabled: true` is set.** Unlike the RCON/Management-Server secrets below (which only matter if you enable those features), this one is live by default in a proxied setup.
2. `server.properties`:
   - `online-mode=false` — Velocity already authenticates players against Mojang; the backend must not do it a second time
   - `prevent-proxy-connections=false` — Let's Players join even tho the ISP/AS sent from the server is different from the one from Mojang Studios Authentication Servers
3. In `velocity.toml`, configure `login-ratelimit`. This is what rate-limits incoming connections in this setup, which is why `connection-throttle: -1` is set to disabled in `bukkit.yml` — the proxy is doing that job instead.

## Option B — No Proxy

1. `server.properties`:
   - `online-mode=true`
2. `bukkit.yml`:
   - `connection-throttle: 4000` - Brings back the Paper default delay between Connections from an IP Adress.
3. `config/paper-global.yml` → `proxies.velocity.enabled: false`

**Never run `online-mode=true` together with `velocity.enabled=true`.** The server will try to re-verify players that Velocity has already verified, which breaks the connection.

---

# Security Checklist

Before opening the server to the public:

* Change every password and secret — this template uses `CHANGEME` as a placeholder value everywhere a real secret is needed. Search the repo for `CHANGEME` and replace every occurrence before going live. This specifically includes:
  * `rcon.password` in `server.properties`
  * `management-server-secret` in `server.properties`
  * `proxies.velocity.secret` in `config/paper-global.yml` — **live and required if you're running behind Velocity (see Proxy Configuration)**, not just a "nice to have"
* `enable-rcon` is `false` by default in this template. Only enable it if you actually need remote console access — every open port is one more thing to secure. Also note: `console.has-all-permissions: true` is set in `config/paper-global.yml`, meaning anything with console/RCON access bypasses your permission plugin entirely. Change the RCON password *before* ever enabling RCON.
* Remove all existing operators.
* Remove the existing whitelist.
* Generate new structure seeds. (spigot.yml)
* Generate a new world seed. (server.properties)
* Review every configuration.
* Configure your firewall.
* Configure automatic backups.
* Update every plugin to the newest compatible version.

---

# Recommended Plugins

These plugins are not included but are highly recommended.

* LuckPerms
* LiteBans
* PlaceholderAPI
* ViaVersion
* ViaBackwards
* Simple Voicechat
* TAB
* Worldguard
* Worldedit
* CommandWhitelist
* Antiseedcracker

---

# Performance Notes

Performance depends much more on hardware than configuration.

The largest factors are:

* CPU single-core performance
* SSD/NVMe speed
* Entity count
* Redstone
* View Distance
* Simulation Distance
* Plugin quality

No configuration can fully compensate for insufficient hardware.

---

# Advanced Tuning (Optional)

The following options are for people comfortable debugging their own server if something goes wrong. Skip this section if you're not sure — misconfiguring it can hurt performance more than help.

## CPU Affinity / Thread Pinning

`config/canvas-server.yml` → `region-scheduler.affinity-scheduler`:

```yaml
tick-region-affinity: [ ]
enable-affinity-scheduler-cpu-affinity: false
```

This pins Canvas' tick threads to specific CPU cores, which can reduce scheduling jitter caused by the OS moving threads between cores. It only makes sense on a **dedicated, bare-metal machine** where you fully control what else runs on the CPU — on shared or virtualized hosting (most Pterodactyl nodes with multiple tenants) it can actively hurt you, since you'd be pinning to cores the hypervisor is also juggling for other tenants.

Before touching this:

1. Know your actual physical core count (not thread/logical count) — `lscpu` on Linux.
2. Understand whether your CPU has SMT/Hyper-Threading, since pinning to logical cores that share a physical core defeats the purpose.
3. Start by leaving `tick-region-affinity` empty (auto) and only hand-pin cores if you've profiled a real, measurable problem — not preemptively.

If you're not confident about steps 1–3, leave this disabled. The default (no pinning) is safe and is what this template ships with.

### Region Scheduler tuning (already applied)

Two settings that *are* already tuned away from Canvas' defaults, matching the official Canvas recommendation for a real performance boost:

* `enable-work-stealing: true`
* `enable-mid-tick-tasks: true`

Both are off by default upstream. This template enables both — no further action needed unless you have a specific reason to disable them.

---

# Configuration Guide

---

## server.properties

Recommended changes:

* Change Bug Report Link
* Generate a new world seed
* Set Max Players — the default `250` is a placeholder, not a recommendation. Match it to your actual hardware (see System Requirements).
* Change the RCON password
* Change the Management Server secret (or leave `management-server-enabled=false` if you don't use the feature)
* Adjust View Distance
* Adjust Simulation Distance
* Set `online-mode` according to your choice in **Proxy Configuration** above
* Take a look at the Code of Conduct and inform yourself about it. A very Vanilla way of letting People read the Rules and force to "accept" them.

The included View Distance (`6`) and Simulation Distance (`4`) settings are optimized for a typical SMP and provide a good balance between gameplay and performance. Simulation Distance of `4` in particular is an aggressive, deliberate choice — it's the single biggest performance lever in this file. Keep in mind that redstone/farms more than 4 chunks from any player will pause; make sure your playerbase understands this if they build AFK farms.

**Optional tuning:** `network-compression-threshold` (default `256`) trades bandwidth for CPU time. On a PvP-focused server where low latency matters more than bandwidth, a higher value (e.g. `512`–`1024`) can reduce per-packet compression overhead. Test with your expected player count before committing to a value.

**Enabling RCON:** `enable-rcon` is `false` by default in this template for security — an open RCON port with a weak or default password is a common attack vector. If you need it (e.g. for a control panel or automation script):

1. Set `enable-rcon=true` in `server.properties`.
2. Change `rcon.password` to a strong, unique value — never the template default.
3. Open the RCON port (default `25575`) — in Pterodactyl via the port allocation for the server, or directly in your firewall if not using Pterodactyl.
4. Restrict access to that port to trusted IPs only if your setup allows it; don't expose RCON to the open internet unless you have to.

---

## spigot.yml

Recommended changes:

### Structure Seeds

Generate new 8-digit integer seeds for every structure. Avoid leading zeros (e.g. `06512439`) — Paper's config parser (SnakeYAML, YAML 1.1 rules) can read a leading-zero number as a string instead of an integer if it isn't a valid octal value. Bukkit's config API will likely coerce it back into a number regardless, but there's no reason to rely on that — just don't generate seeds with a leading zero.

### Commands

After development:

```yaml
send-namespaced: false
```

Players should only see commands they are allowed to execute.

Although permission plugins usually solve this already, disabling namespaced commands keeps operator command completion much cleaner.

### hopper-amount

```yaml
hopper-amount: 12
```

Not vanilla (vanilla moves 1 item per hopper transfer). This template intentionally raises it to 12 as a deliberate SMP design choice: farms move items much faster, and it can act as a performance benefit too, since large farms clear their output faster rather than accumulating a backlog of unmoved item entities. If you want strict vanilla hopper behavior, set this back to `1`.

---

## permissions.yml

No action required.

This is Paper's legacy permission system.

Use LuckPerms or another modern permission plugin instead.

---

## eula.txt

If you do not agree to Minecraft's EULA:

```text
eula=false
```

The server will refuse to start.

---

## commands.yml

This file allows command aliases.

Several gamemode aliases are already included as examples.

---

## bukkit.yml

Mostly preference-based, with one deliberate performance choice worth knowing about:

```yaml
spawn-limits:
  monsters: 30      # vanilla default: 70
ticks-per:
  monster-spawns: 6  # vanilla default: 1
```

Both are intentionally throttled well below vanilla to reduce hostile-mob load on the server (fewer monsters alive at once, and spawn attempts checked 6x less often). Combined with `mob-spawn-range: 4` in `spigot.yml`, this is a consistent, deliberate trade-off: noticeably fewer monsters overall, in exchange for TPS headroom. If your community relies heavily on classic mob-farm output, you may want to raise these back toward vanilla values.

---

## banned-players.json

No action required.

The file regenerates automatically.

Using LiteBans or another punishment plugin is recommended.

---

## banned-ips.json

No action required.

Also regenerates automatically.

---

# Canvas Configuration

---

## config/canvas-server.yml

Recommended options:

### server-mod-name

Changes the server name displayed in the F3 debug screen.

### default-respawn-dimension-key

Allows players to respawn in another world (for example a Spawn world).

### enderchest-six-rows

Not vanilla.

Useful for many SMP servers but should be considered carefully.

### log-cleaner

Highly recommended.

Helps reduce unnecessary personal information inside logs and can simplify GDPR/privacy requests.

### chat.disable-chat-reporting

Enabled by default in this template. This disables Minecraft's signed chat reporting system, which is a deviation from vanilla behavior. It's kept enabled here for a practical reason beyond anti-report: **Bedrock players connecting via Geyser/Floodgate cannot participate in Java's signed chat sessions**, and leaving chat reporting enforced can cause chat issues for them. If you don't run Geyser and want full vanilla chat-signing behavior, you can set this back to `false`.

### networking / filter-velocity-packet

Enabled by default. Filters the entity-velocity packet, which can account for a large share of network traffic on busy servers, without changing the visible Vanilla motion of entities other than a few explicitly excluded types (squids, item entities, ender eyes, shulker bullets). No reason to disable this unless you're debugging a networking issue.

---

## config/canvas-worlds.yml

### entity-collision-mode

Current value:

```
NO_COLLISIONS
```

Set deliberately for two reasons: performance (no per-tick collision-push calculations at all), and gameplay — it stops players from being able to push other players out of safe zones. This is the most aggressive of Canvas' four collision modes. Know the trade-off before keeping it: classic Vanilla mechanics that rely on entity pushing (boat/minecart stacking via collision, piston-based mob sorting/elevators that use pushing) will not work as expected.

Other available modes, in case you want something less aggressive:

* `VANILLA` — completely vanilla collision behavior
* `ONLY_PUSHABLE_PLAYERS_LARGE` / `ONLY_PUSHABLE_PLAYERS_SMALL` — only players push each other (achieves the "no pushing out of safe zones" goal without disabling all entity collisions), searching in a normal or small radius respectively

### use-legacy-blast-protections

Interesting option for CrystalPvP servers.

Some communities prefer the older explosion mechanics.

---

# Paper Configuration

---

## config/paper-global.yml

Velocity settings live here (`proxies.velocity`). See **Proxy Configuration** above for the exact values depending on whether you run behind a proxy or not — don't just replace the secret without also setting `enabled` and `online-mode` correctly for your chosen setup.

Other notable deviations from Paper defaults in this file:

* `anticheat.obfuscation.enable-item-obfuscation: true` — hides item data (enchantments, lore, etc.) of other players from cheat clients. Off by default upstream. If your resource pack shows different textures based on another player's item enchantments/lore, this can break that — otherwise safe to leave on.
* `packet-limiter.overrides.minecraft:place_recipe` — tightened well below the default rate limit as a hardening measure against known recipe-spam exploits. No action needed.

---

## config/paper-world-defaults.yml

Not just preference-based — several deliberate performance and behavior choices live here:

* `misc.update-pathfinding-on-block-update: false` — Paper's own docs recommend this exact change for servers with lots of entities and automated farms/redstone clocks. Matches this template's use case directly.
* `environment.optimize-explosions: true` — caches entity lookups during explosions instead of recalculating them, noticeably speeding up explosion-heavy scenarios (TNT, crystals). Off by default upstream.
* `chunks.prevent-moving-into-unloaded-chunks: true` — stops players from moving into unloaded chunks. Off by default upstream, good general hardening.
* `hopper.disable-move-event: true` — a significant hopper performance win, but it **completely disables `InventoryMoveItemEvent` for hoppers**. This template ships without a claim/protection plugin, so it's safe here. **If you add a claims plugin (GriefPrevention, WorldGuard, etc.) that relies on this event to stop hopper theft across land claims, that protection will silently stop working.** Set this back to `false` if you add such a plugin.
* `misc.redstone-implementation: ALTERNATE_CURRENT` — changed from the Vanilla default. This meaningfully reduces redstone-dust lag by optimizing power calculations and cutting down block/shape updates, which matters on a farm-heavy SMP. Trade-off: it changes some redstone-dust behavior versus Vanilla, and can break very specific, complicated farms that depend on exact Vanilla timing/quasi-connectivity quirks. For most builds this isn't something to worry about — if you do run into a farm that breaks and can't be adapted, `EIGENCRAFT` is a documented middle-ground alternative implementation to try before falling back to `VANILLA`.

---

# Backups

Always create regular backups of:

* worlds/
* plugins/
* config/
* server.properties
* spigot.yml
* bukkit.yml

(`paper-global.yml` and `paper-world-defaults.yml` are already covered by the `config/` folder backup above — no need to list them separately. `spigot.yml` and `bukkit.yml` live in the server root, outside `config/`, so they'd be missed if not listed explicitly.)

Never rely solely on your hosting provider for backups.

---

# Recommended Startup Command (14 GB with Overhead)

```bash
java -Xms10000M -Xmx10000M -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true -XX:G1NewSizePercent=40 -XX:G1MaxNewSizePercent=50 -XX:G1HeapRegionSize=16M -XX:G1ReservePercent=15 -Dterminal.jline=false -Dterminal.ansi=true --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

---

# Credits

Configuration created by **040Elias**.

Built on top of the excellent work of the Canvas, Paper and Folia development teams.