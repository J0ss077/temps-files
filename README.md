# Modding Support

Electric Tiles exposes a **Data Stage interface** that allows other mods to create electric variants of any tile - including tiles from third-party mods. This is the recommended way to extend Electric Tiles without modifying its source.

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

During the data stage, Electric Tiles registers a global table called **ElectricTilesDataInterface**. This table exposes the following methods:

- **adaptTilePrototype(data)** - Creates electric variants from tile/item/recipe prototypes.
- **getItemPrefix()** - Returns the prefix applied to all generated item names.
- **getTilePrefix()** - Returns the prefix applied to all generated tile names.
- **getInterfaceVersion()** - Returns the current interface version string.

The core method is **adaptTilePrototype()**. It accepts an array of entries, where each entry describes a tile variant to create. Electric Tiles will process each entry, apply defaults and overrides, and register the resulting prototypes automatically.

---

## adaptTilePrototype() Reference

Each entry passed to **adaptTilePrototype()** follows this structure:

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

### tile - TilePrototype

The tile data for the electric variant. This should mirror a standard [TilePrototype](https://lua-api.factorio.com/latest/prototypes/TilePrototype.html) definition.

**Assumed defaults** - you can omit these and Electric Tiles will fill them in:

- **type** - defaults to "tile"
- **order** - defaults to "a[base]"
- **minable** - defaults to _[TODO: describe what the default MinableStructure looks like]_
- **localised_name** - read from your locale files

**Overwritten properties** - Electric Tiles will always set these, regardless of what you pass:

- **minable.result** - set to the generated item, if item data was provided
- **placeable_by** - set to the generated item, if item data was provided

**Modified properties** - Electric Tiles adjusts these while preserving your input as a base:

- **name** - a prefix is prepended (see getTilePrefix())
- **order** - a postfix is appended
- **localised_name** - a postfix is appended

---

### item - ItemPrototype

The item used to place the electric tile. Mirrors a standard [ItemPrototype](https://lua-api.factorio.com/latest/prototypes/ItemPrototype.html).

**Assumed defaults:**

- **type** - defaults to "item"
- **order** - defaults to "a[base]"
- **subgroup** - defaults to "F077ET-terrain"
- **place_as_tile** - defaults to _[TODO: describe what the default PlaceAsTileStructure looks like]_
- **localised_name** - read from your locale files

**Overwritten properties:**

- **place_as_tile.result** - set to the generated tile

**Modified properties:**

- **name** - a prefix is prepended (see getItemPrefix())
- **icon(s)** - a copper wire overlay icon is composited on top
- **order** - a postfix is appended
- **localised_name** - a postfix is appended

---

### recipe - RecipePrototype

The crafting recipe for the electric tile. Mirrors a standard [RecipePrototype](https://lua-api.factorio.com/latest/prototypes/RecipePrototype.html).

**Assumed defaults:**

- **type** - defaults to "recipe"
- **enabled** - defaults to false
- **auto_recycle** - defaults to true
- **category** - defaults to "advanced-crafting"
- **ingredients** - defaults to 1x Iron Stick, 1x Copper Cable, 1x base tile

**Overwritten properties:**

- **name** - always set to the internally generated name with prefix
- **results** - always set to the generated item

No other properties are modified.

---

### others - Adapter flags

Optional configuration that changes how the adapter processes an entry.

- **add_copper_wire_icon** _(boolean, default: true)_ - set to false to skip the copper wire icon overlay on the item.
- **result_amount** _(integer, default: 1)_ - number of tiles produced per craft.
- **use_default_recipe** _(boolean, default: false)_ - if true and no recipe data was passed, the default recipe will be generated automatically.

---

### technology - Unlock technologies

An array of technology name strings. If a recipe is created and enabled is false, the recipe will be added as an unlock effect to each technology listed here.

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

- **You do not need pre-registered prototypes.** You can define tile, item, and recipe tables from scratch and pass them directly to **adaptTilePrototype()**. The interface will handle prototype registration.
- **Passing only a tile is valid.** Item and recipe data are optional. If omitted, the electric tile will exist in the game but will have no craftable item or recipe associated with it - useful for programmatic placement or editor use.
- **Load order matters.** Make sure Electric Tiles loads before your mod reads **ElectricTilesDataInterface**. The optional dependency declaration handles this automatically.
