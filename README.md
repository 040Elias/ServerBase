# Canvas Server Template

> **Canvas Version:** 26.1.2
> **Required Java Version:** Java 25

This repository contains a preconfigured **Canvas 26.1.2** server optimized for high performance while preserving vanilla gameplay as much as possible. The configuration was primarily built with **CrystalPvP** in mind but also serves as an excellent base for SMPs and other survival servers.

Canvas is a high-performance fork of Folia. All Folia-compatible plugins work on Canvas, while Canvas also introduces additional APIs and optimizations.

---

# Documentation

Before changing any configuration, it is highly recommended to read the official documentation.

## Canvas

https://docs.canvasmc.io/canvas/introduction

## Paper

https://docs.papermc.io/paper/

Many configuration files and concepts are inherited from Paper, making the Paper documentation applicable to most server configuration options.

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

This is the recommended update procedure used by Paper and avoids file replacement while the server is running.

---

# Updating Canvas

This setup **does not use Canvas Sculptor**.

Development for **Canvas 26.1.2** is effectively frozen, as Canvas only maintains the latest release — once a new version enters development, older versions generally stop receiving fixes. A new version of this template will be published once **Canvas 26.2** reaches a stable release.

If you ever migrate to a newer Canvas release in the meantime:

1. Create a full backup.
2. Read the changelog.
3. Verify plugin compatibility.
4. Replace `server.jar`.
5. Test everything before production.

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
   - `secret:` must be identical to the `forwarding-secret` in your Velocity's `velocity.toml`
2. `server.properties`:
   - `online-mode=false` — Velocity already authenticates players against Mojang; the backend must not do it a second time
   - `prevent-proxy-connections=true` — blocks anyone from hitting the backend port directly and bypassing the proxy
3. In `velocity.toml`, configure `connection-throttle`. This is what rate-limits incoming connections in this setup, which is why `rate-limit=0` (disabled) is left as-is in `server.properties` — the proxy is doing that job instead.

## Option B — No Proxy

1. `server.properties`:
   - `online-mode=true`
   - `rate-limit=0` (current PaperMC default — Paper does not enforce a connection rate limit here by default; adjust if you want your own limit instead of relying on a proxy)
2. `config/paper-global.yml` → `proxies.velocity.enabled: false`

**Never run `online-mode=true` together with `velocity.enabled=true`.** The server will try to re-verify players that Velocity has already verified, which breaks the connection.

---

# Security Checklist

Before opening the server to the public:

* Change every password and secret — **specifically check `rcon.password` and `management-server-secret` in `server.properties`**, since both are pre-filled with generated values in this template and must never be reused as-is.
* `enable-rcon` is `false` by default in this template. Only enable it if you actually need remote console access — every open port is one more thing to secure.
* Remove all existing operators.
* Remove the existing whitelist.
* Generate new structure seeds.
* Generate a new world seed.
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
* Set `online-mode` and `rate-limit` according to your choice in **Proxy Configuration** above

The included View Distance settings are optimized for a typical SMP and provide a good balance between gameplay and performance.

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

No required changes.

Only adjust values according to personal preference.

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

---

## config/canvas-worlds.yml

### entity-collisions-mode

Current value:

```
NO_COLLISIONS
```

Excellent for performance.

If you want completely vanilla collision behavior, use:

```
VANILLA
```

### use-legacy-blast-protections

Interesting option for CrystalPvP servers.

Some communities prefer the older explosion mechanics.

---

# Paper Configuration

---

## config/paper-global.yml

Velocity settings live here (`proxies.velocity`). See **Proxy Configuration** above for the exact values depending on whether you run behind a proxy or not — don't just replace the secret without also setting `enabled` and `online-mode` correctly for your chosen setup.

---

## config/paper-world-defaults.yml

No required changes.

Everything here is preference-based.

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

# Recommended Startup Command (14 GB)

```bash
java -Xms10000M -Xmx10000M -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true -XX:G1NewSizePercent=40 -XX:G1MaxNewSizePercent=50 -XX:G1HeapRegionSize=16M -XX:G1ReservePercent=15 -Dterminal.jline=false -Dterminal.ansi=true --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

---

# Credits

Configuration created by **040Elias**.

Built on top of the excellent work of the Canvas, Paper and Folia development teams.