# gomod.json

The manifest lives at `src/main/resources/gomod.json`. It deliberately mirrors `fabric.mod.json`:
GoCraft converts it into one when the mod loads, so these fields mean exactly what they do there.

```json
{
  "id": "${id}",
  "version": "${version}",
  "name": "My Mod",
  "description": "What it does.",
  "authors": ["You"],
  "contact": { "homepage": "https://example.com" },
  "license": "BSD-3-Clause",
  "icon": "assets/mymod/icon.png",
  "environment": "*",
  "depends": { "minecraft": ">=26.3", "gocraft": "*" }
}
```

## Fields passed through to Fabric

`id`, `version`, `name`, `description`, `authors`, `contributors`, `contact`, `license`, `icon`,
`environment`, `depends`, `recommends`, `suggests`, `conflicts`, `breaks`, `provides`, `accessWidener`,
`classTweaker` and `custom`.

`depends` can name Fabric mods and other Go mods alike. Go mods initialize after the Go mods they depend
on, and an unmet dependency stops the game with a message naming the mod.

## Fields specific to GoCraft

| Field | Meaning |
| --- | --- |
| `entrypoints` | Which packages to link: `{"main": ["pkg"], "client": ["pkg"]}`, relative to `src/main/go` and `src/client/go`. Omit it to link everything. |
| `customEntrypoints` | Extra Fabric entrypoint keys to expose, such as `"fabric-datagen"`. Handle them with `gocraft.Entrypoint`. |
| `hooks` | Injections into game code. See [Hooks](hooks.md). |
| `abi` | Added by go.jar. Do not write it yourself. |

## Tokens

`${name}` expands at build time. `id`, `version`, `minecraft_version` and `loader_version` are always
available, and `[properties]` in `build.toml` adds your own. An unknown token fails the build.
