# mapcrate plugin registry

The catalog for the **mapcrate** map viewer's plugins. This repo is **curated**: the app fetches
`registry.json` from here, and plugin packages are hosted as **GitHub Release assets** on this repo.
There is no upload API or account system — plugins are reviewed and added manually.

## For users — getting plugins
Open the map viewer → **Plugins** → install from the list. You can also drop a plugin `.zip` or
folder into the app's `plugins/` directory and it is picked up automatically.

## For plugin authors — suggesting a plugin
Plugins are added manually after review. Two ways to submit:
- **GitHub:** open a [plugin suggestion issue](../../issues/new?template=suggest-plugin.yml).
- **Email:** send it to `mapcrate@proton.me` — attach the `.zip` if small, or send a download link
  if large.

Your package must be a `.zip` containing a `plugin.json` (`id`, `name`, `version`, `description`)
plus its content (`maps/`, `tiles/`, …). It must load by dropping it into the app's `plugins/` folder.

## `registry.json` shape
```jsonc
{
  "plugins": [
    // simple plugin — a single zip:
    { "id": "example", "name": "Example", "description": "…", "kind": "data",
      "version": "1.0.0", "url": "…/example-1.0.0.zip", "sha256": "…", "size": 12345, "patchNotes": "…" },

    // composite plugin — several components, each updated independently:
    { "id": "example-pack", "name": "Example Pack", "description": "A collection of maps.", "kind": "data",
      "maps": [
        { "id": "map-a", "version": "1.0.0", "url": "…/map-a-1.0.0.zip", "sha256": "…", "size": 12345, "patchNotes": "Initial." }
      ] }
  ]
}
```
`kind` is `data` (free) or `code` (a plugin that ships a DLL). The app installs simple plugins as one
zip; for a composite plugin it installs each map component separately and updates only the ones whose
`version` changed.

## Owner — add / update a plugin
1. **Review** the suggested `.zip`: drop it into the app's `plugins/` folder — does it load and render?
2. **Zip** it as `<id>-<version>.zip`.
3. **Cut a release** and upload the asset:
   ```
   gh release create <id>-<version> <id>-<version>.zip \
       -R mapcrate/plugin-registry -t "<name> <version>" -n "<patch notes>"
   ```
   The asset URL is then `https://github.com/mapcrate/plugin-registry/releases/download/<id>-<version>/<id>-<version>.zip`.
4. **Edit `registry.json`**: add or bump the plugin's entry (`version`, `url`, `sha256`, `size`,
   `patchNotes`). For a **composite** plugin, bump only the changed map's entry — users then
   re-download only that map, not the whole pack.
5. **Commit** `registry.json`. (`raw.githubusercontent.com` has a ~5-minute cache, so the change
   takes a few minutes to reach clients.)

Use per-version release tags (`<id>-1.0.0`, `<id>-1.1.0`, …) so asset URLs are immutable and old
versions stay downloadable.
