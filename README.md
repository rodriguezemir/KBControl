# KBControl — Minecraft PvP Knockback Control for Paper

![Java 25](https://img.shields.io/badge/Java-25-blue)
![Paper API 26.1.2](https://img.shields.io/badge/Paper%20API-26.1.2-2ea44f)
![License GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-orangered)

KBControl is a small PaperMC PvP plugin for controlling combat knockback. It
intercepts player velocity packets received shortly after tracked entity damage and scales the
horizontal and vertical knockback components independently, making it useful for
practice servers, PvP balancing, and custom knockback experiments.

> [!WARNING]
> The plugin may have issues with anti-cheats. use with caution.

## Features

- Adjust horizontal knockback (`X`/`Z`) and vertical knockback (`Y`) separately.
- Use `/knockback` or the `/kb` alias to inspect, change, or reload multipliers.
- Persist command changes back into `config.yml`.
- Filter velocity packets using a short-lived player damage marker.
- Require the `customkb.command.knockback` permission for all commands.

## Keywords / tags

`minecraft-plugin`, `papermc`, `paper-plugin`, `pvp`, `knockback`,
`custom-knockback`, `kbcontrol`, `packet-events`, `minecraft-pvp`

Suggested GitHub topics: `minecraft-plugin`, `papermc`, `pvp`, `knockback`,
`paper-plugin`, `packetevents`.

## Dependencies

* [PacketEvents](https://modrinth.com/plugin/packetevents)

## Installation

1. Install a compatible Paper server.
2. Install the PacketEvents plugin on the same server.
3. Build or download the KBControl/CustomKB JAR.
4. Place the JAR in your server `plugins/` directory.
5. Start the server once to generate the default `plugins/CustomKB/config.yml`.
6. Edit the generated config or use `/kb set ...` in game/console.
7. Run `/kb reload` after manual config changes.

## Quick start

```text
/kb get
/kb set horizontal 0.9
/kb set vertical 1.05
/kb reload
```

The defaults are neutral multipliers, so vanilla/server knockback is left
unchanged until you set a different value.

## Commands

All commands require `customkb.command.knockback`.

| Command | Alias | Arguments | Behavior |
| --- | --- | --- | --- |
| `/knockback get` | `/kb get` | none | Shows current horizontal and vertical multipliers. |
| `/knockback set horizontal <value>` | `/kb set horizontal <value>` | `value` is a decimal number `>= 0.0` | Saves the horizontal multiplier. |
| `/knockback set vertical <value>` | `/kb set vertical <value>` | `value` is a decimal number `>= 0.0` | Saves the vertical multiplier. |
| `/knockback reload` | `/kb reload` | none | Reloads values from `config.yml`. |

There is no separate help subcommand in the current source.

## Permissions

| Permission | Default | Covers |
| --- | --- | --- |
| `customkb.command.knockback` | `op` | Access to every `/knockback` and `/kb` subcommand. |

## Configuration

Generated default:

```yaml
knockback:
  # Multiplier applied to the horizontal (X/Z) component of combat knockback.
  horizontal-multiplier: 1.0
  # Multiplier applied to the vertical (Y) component of combat knockback.
  vertical-multiplier: 1.0
```

Practical PvP example:

```yaml
knockback:
  # Slightly reduce sideways/backward movement.
  horizontal-multiplier: 0.85
  # Keep upward lift close to default.
  vertical-multiplier: 1.0
```

How to read multiplier values:

- `1.0` keeps that component unchanged.
- `0.0` removes that component from handled combat knockback.
- `0.5` halves that component.
- `2.0` doubles that component.

Command input is limited to values `>= 0.0`. Manual config edits are loaded as
doubles with a fallback default of `1.0`; the current source does not add extra
validation or clamping when reading the file.

## How knockback control works

Minecraft uses entity velocity packets to communicate motion, including combat
knockback. KBControl tracks player victims of non-cancelled
`EntityDamageByEntityEvent` events, keeps that pending combat-knockback marker
for 250 milliseconds, and consumes it when the player receives their own
`ENTITY_VELOCITY` packet.

When the packet matches, the plugin rescales the velocity like this:

```text
newX = oldX * horizontal-multiplier
newY = oldY * vertical-multiplier
newZ = oldZ * horizontal-multiplier
```

This keeps horizontal and vertical tuning independent while avoiding most
non-combat velocity changes.

## Behavior and limitations

- Only player victims are tracked.
- Only velocity packets for the receiving player's own entity are modified.
- The pending damage marker is consumed once and expires after 250 ms.
- Cancelled damage events are ignored.
- Velocity packets for other entities, or without a pending damage marker within
  250 ms, are ignored. This is a temporal heuristic: unrelated velocity arriving
  during that window can be scaled and consume the marker.
- Other plugins that modify damage, velocity, packets, or anti-cheat behavior may
  affect the final result; test with your full server plugin stack.
- The current source declares Paper APIs, not Spigot/Bukkit compatibility.

## Usage examples

Show current settings:

```text
/kb get
```

Make PvP knockback lower and flatter:

```text
/kb set horizontal 0.75
/kb set vertical 0.8
```

Create high-lift practice knockback while leaving horizontal movement unchanged:

```text
/kb set horizontal 1.0
/kb set vertical 1.4
```

After editing `config.yml` manually:

```text
/kb reload
```

## Build and development

This project uses the Gradle wrapper.

```bash
./gradlew build --stacktrace
```

The configured CI build runs the same Gradle build on Java 25 for Ubuntu and
Windows. No test source files are currently present in `src/`; the Gradle build
is still the determinable repository check.

Useful project tasks:

```bash
./gradlew -q printProjectName
./gradlew -Pver=v1.0.0 release
```

The ordinary shadow build uses the Gradle project name and version in
`build/libs/`. The `release` task renames a non-snapshot release artifact to
`build/libs/CustomKB.jar`.

## License

This repository includes the GNU General Public License version 3 text in
[`LICENSE`](LICENSE).
