# Modding Support

Electric Tiles exposes a **Data Stage interface** that allows other mods to create electric variants of any tile — including tiles from third-party mods. This is the recommended way to extend Electric Tiles without modifying its source.

> **Note:** The runtime-stage modding process introduced before version 1.2.8 has been deprecated due to breaking changes in Factorio 2.0.60. If your mod relied on it, migrate to the Data Stage interface described below.

---

## Requirements

Before using the interface, make sure your mod meets the following:

1. Declare an optional dependency on Electric Tiles in your `info.json`:

```json
"dependencies": ["? electric-tiles >= 1.2.8"]
```

2. Use the interface exclusively during Factorio's **data stage** (i.e., inside `data.lua` or files required from it).

---

## How It Works

During the data stage, Electric Tiles registers a global table called `ElectricTilesDataInterface`. This table exposes the following methods:

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Method</th>
      <th width="65%">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>adaptTilePrototype(data)</code></td>
      <td>Creates electric variants from tile/item/recipe prototypes.</td>
    </tr>
    <tr>
      <td><code>getItemPrefix()</code></td>
      <td>Returns the prefix applied to all generated item names.</td>
    </tr>
    <tr>
      <td><code>getTilePrefix()</code></td>
      <td>Returns the prefix applied to all generated tile names.</td>
    </tr>
    <tr>
      <td><code>getInterfaceVersion()</code></td>
      <td>Returns the current interface version string.</td>
    </tr>
  </tbody>
</table>

The core method is `adaptTilePrototype()`. It accepts an array of entries, where each entry describes a tile variant to create. Electric Tiles will process each entry, apply defaults and overrides, and register the resulting prototypes automatically.

---

## `adaptTilePrototype()` Reference

Each entry passed to `adaptTilePrototype()` follows this structure:

```lua
ElectricTilesDataInterface.adaptTilePrototype({
    {
        tile       = {},  -- [REQUIRED] TilePrototype data
        item       = {},  -- [OPTIONAL] ItemPrototype data
        recipe     = {},  -- [OPTIONAL] RecipePrototype data
        others     = {},  -- [OPTIONAL] Adapter configuration flags
        technology = {},  -- [OPTIONAL] Array of technology names that unlock the recipe
    },
    -- additional entries...
})
```

You can pass multiple entries in a single call.

---

### `tile` — TilePrototype

The tile data for the electric variant. This should mirror a standard [TilePrototype](https://lua-api.factorio.com/latest/prototypes/TilePrototype.html) definition.

**Assumed defaults** — you can omit these and Electric Tiles will fill them in:

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Default value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>type</code></td>
      <td><code>"tile"</code></td>
    </tr>
    <tr>
      <td><code>order</code></td>
      <td><code>"a[base]"</code></td>
    </tr>
    <tr>
      <td><code>minable</code></td>
      <td><em>[TODO: describe what the default MinableStructure looks like]</em></td>
    </tr>
    <tr>
      <td><code>localised_name</code></td>
      <td>Read from your locale files</td>
    </tr>
  </tbody>
</table>

**Overwritten properties** — Electric Tiles will always set these, regardless of what you pass:

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Behavior</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>minable.result</code></td>
      <td>Set to the generated item, if item data was provided</td>
    </tr>
    <tr>
      <td><code>placeable_by</code></td>
      <td>Set to the generated item, if item data was provided</td>
    </tr>
  </tbody>
</table>

**Modified properties** — Electric Tiles adjusts these while preserving your input as a base:

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Modification</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>A prefix is prepended (see <code>getTilePrefix()</code>)</td>
    </tr>
    <tr>
      <td><code>order</code></td>
      <td>A postfix is appended</td>
    </tr>
    <tr>
      <td><code>localised_name</code></td>
      <td>A postfix is appended</td>
    </tr>
  </tbody>
</table>

---

### `item` — ItemPrototype

The item used to place the electric tile. Mirrors a standard [ItemPrototype](https://lua-api.factorio.com/latest/prototypes/ItemPrototype.html).

**Assumed defaults:**

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Default value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>type</code></td>
      <td><code>"item"</code></td>
    </tr>
    <tr>
      <td><code>order</code></td>
      <td><code>"a[base]"</code></td>
    </tr>
    <tr>
      <td><code>subgroup</code></td>
      <td><code>"F077ET-terrain"</code></td>
    </tr>
    <tr>
      <td><code>place_as_tile</code></td>
      <td><em>[TODO: describe what the default PlaceAsTileStructure looks like]</em></td>
    </tr>
    <tr>
      <td><code>localised_name</code></td>
      <td>Read from your locale files</td>
    </tr>
  </tbody>
</table>

**Overwritten properties:**

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Behavior</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>place_as_tile.result</code></td>
      <td>Set to the generated tile</td>
    </tr>
  </tbody>
</table>

**Modified properties:**

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Modification</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>A prefix is prepended (see <code>getItemPrefix()</code>)</td>
    </tr>
    <tr>
      <td><code>icon(s)</code></td>
      <td>A copper wire overlay icon is composited on top</td>
    </tr>
    <tr>
      <td><code>order</code></td>
      <td>A postfix is appended</td>
    </tr>
    <tr>
      <td><code>localised_name</code></td>
      <td>A postfix is appended</td>
    </tr>
  </tbody>
</table>

---

### `recipe` — RecipePrototype

The crafting recipe for the electric tile. Mirrors a standard [RecipePrototype](https://lua-api.factorio.com/latest/prototypes/RecipePrototype.html).

**Assumed defaults:**

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Default value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>type</code></td>
      <td><code>"recipe"</code></td>
    </tr>
    <tr>
      <td><code>enabled</code></td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><code>auto_recycle</code></td>
      <td><code>true</code></td>
    </tr>
    <tr>
      <td><code>category</code></td>
      <td><code>"advanced-crafting"</code></td>
    </tr>
    <tr>
      <td><code>ingredients</code></td>
      <td>1x Iron Stick, 1x Copper Cable, 1x base tile</td>
    </tr>
  </tbody>
</table>

**Overwritten properties:**

<table width="100%">
  <thead>
    <tr>
      <th width="35%">Property</th>
      <th width="65%">Behavior</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>Always set to the internally generated name with prefix</td>
    </tr>
    <tr>
      <td><code>results</code></td>
      <td>Always set to the generated item</td>
    </tr>
  </tbody>
</table>

No other properties are modified.

---

### `others` — Adapter flags

Optional configuration that changes how the adapter processes an entry.

<table width="100%">
  <thead>
    <tr>
      <th width="30%">Key</th>
      <th width="15%">Type</th>
      <th width="15%">Default</th>
      <th width="40%">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>add_copper_wire_icon</code></td>
      <td>boolean</td>
      <td><code>true</code></td>
      <td>Set to <code>false</code> to skip the copper wire icon overlay on the item.</td>
    </tr>
    <tr>
      <td><code>result_amount</code></td>
      <td>integer</td>
      <td><code>1</code></td>
      <td>Number of tiles produced per craft.</td>
    </tr>
    <tr>
      <td><code>use_default_recipe</code></td>
      <td>boolean</td>
      <td><code>false</code></td>
      <td>If <code>true</code> and no recipe data was passed, the default recipe will be generated automatically.</td>
    </tr>
  </tbody>
</table>

---

### `technology` — Unlock technologies

An array of technology name strings. If a recipe is created and `enabled` is `false`, the recipe will be added as an unlock effect to each technology listed here.

```lua
technology = { "electric-energy-distribution-1", "rocket-silo" }
```

---

## Example

The following example creates an electric variant of the **Space Platform tile** from the mod [_Space Platform, for ground_](https://mods.factorio.com/mod/space-platform-for-ground) by Snouz. Since that mod has already registered its prototypes, we can reference them directly:

```lua
-- data.lua (runs after the dependency mod's data stage)

ElectricTilesDataInterface.adaptTilePrototype({
    {
        tile       = data.raw.tile["space-platform-for-ground"],
        item       = data.raw.item["space-platform-for-ground"],
        recipe     = data.raw.recipe["space-platform-for-ground"],
        others     = { add_copper_wire_icon = false },
        technology = { "rocket-silo" },
    }
})
```

Note that the copper wire icon is disabled here because the space platform tile already has a distinctive appearance.

---

## Additional Notes

- **You do not need pre-registered prototypes.** You can define tile, item, and recipe tables from scratch and pass them directly to `adaptTilePrototype()`. The interface will handle prototype registration.
- **Passing only a tile is valid.** Item and recipe data are optional. If omitted, the electric tile will exist in the game but will have no craftable item or recipe associated with it — useful for programmatic placement or editor use.
- **Load order matters.** Make sure Electric Tiles loads before your mod reads `ElectricTilesDataInterface`. The optional dependency declaration handles this automatically.
