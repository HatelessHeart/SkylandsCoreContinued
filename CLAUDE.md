# Skylands Core — project notes

Forge mod (`skylandscore`) for Minecraft **1.20.1**, the progression backbone of the
"Escape the Skylands" modpack by HatelessHeart. Originally written by TheArchitects727
(abandoned Sep 2024); now continued here. GPLv3.

## Build & run (Windows, VS Code)
- Requires **JDK 17** (Gradle 8.1.1 cannot run on Java 21). `java -version` must say 17.
- `.\gradlew runClient` — launches Minecraft with the mod (first run: 10–20 min).
- `.\gradlew runServer` — dedicated server. `.\gradlew build` — jar in `build/libs/`.
- Forge 47.2.6, Parchment mappings `2023.06.26-1.20.1`, Mixin 0.8.5.
- Not yet verified to build since the hand-over. `libs.azuredoom.com` in `build.gradle`
  may be dead and unused; remove it if resolution fails there.

## Dependencies (build.gradle, via CurseMaven)
Biomes O' Plenty, TerraBlender, Custom Portal API Reforged (all portals),
Ecospherical Expansion (`eco:` dimension types/noise settings), Create, Create Crafts &
Additions, Patchouli, Entity Texture Features + Entity Model Features, YUNG's Better
Strongholds + YUNG's API. JEI is compile-only.

## Layout
- Java: `src/main/java/net/architects/skylandscore/` (~3k lines). Main class
  `EcoPortalsMod` (legacy name) registers everything and all portals in `addPortals`.
- Most content is data: `src/main/resources/data/` (recipes, dimensions, biomes,
  advancements, functions, loot) and `assets/` (models, textures, lang, Patchouli books).
- `ModConfiguredFeatures`/`ModPlacedFeatures`/`ModBiomeModifiers` are datagen
  bootstraps with no data generator wired up; the committed JSON is what actually runs.

## Progression design
Seven sky-island dimensions (each 53 "shattered" biomes), in order:
OG Strato (replaces the overworld via world preset) → Strato → Exo → Thermo → Meso →
Tropo → Amplified. Per stage:
1. Mine the stage ore with the previous tier's pickaxe (`needs_*_tool` tags are offset
   by one tier on purpose).
2. Create chain: crush → dust; compact (dust + clay + water) → brick; charge (Crafts &
   Additions) → charged brick; sequenced assembly → **Tablet**.
3. Tablet crafts and lights the stage's End portal frame → End → fight a named ender
   dragon ("<Stage>spherical Shard of the Dragon"). OptiFine/ETF skins in
   `assets/minecraft/optifine/random/entity/enderdragon/`.
4. Kill → advancement `kill_<stage>_dragon` → loot (`<stage>_dragon_death`: diamond
   sword + Dragon Essence) and `leave<stage>delay` → 500 ticks later teleport to the
   next dimension at 0 204 0 with an obsidian platform + slow falling.
5. Drain Essence into one of 7 coloured fluids → fill onto a Netherite Upgrade Template
   → smithing upgrades a diamond pickaxe to the next tier.
Each stage also has a return portal (lit with Dragon Essence) and a nether portal (lava).
Endgame: Glitched Stone (Jagged pickaxe), Final Dragon Shard (nether star + all 7
fluids) — currently has no use.

**The named dragons are spawned outside this repo** (modpack side, e.g. KubeJS/commands).
The `DragonFight` mixin overwrites `EndDragonFight.tick()` with an empty body, disabling
vanilla dragon-fight logic entirely.

## Known issues (found in review, not yet fixed)
1. Exploit: advancement `killwithogstrato` grants the Amplified (final) dragon loot for
   any kill with a netherite sword — looks like leftover test code.
2. Fluid overlay textures exist only for obsidian tears and red essence; the other six
   essence fluids show the missing texture when submerged.
3. No lang entries for the 8 fluid blocks or `unfinished_final_shard`; no blockstates
   for the fluid blocks.
4. `grant_lore_on_first_join` uses book id `skylandscorebook` without namespace; the
   lore book ("Transcendent Tome") has categories but no entries; `functions/dialog/`
   checks a non-existent `kill_vanilla_dragon` advancement.
5. `leaveamplified` teleports into Amplified itself — no ending yet.
6. Patchouli guide says bricks are compacted with lava; recipes use water.
7. Every ore generates in every Skylands biome (`is_shattered`) and the vanilla
   overworld; gating is by pickaxe tier only.
8. Stratified tool tier repairs with Laminated Crystal (likely copy-paste).
9. Dead/leftover: Astheno/Litho/Large Biomes dimensions and their portal blocks (no
   recipes), commented-out Stone Orb entity, unused `GlowBlock`/`OreBlock`, folders
   `modelsBack/`, `blockstatesBack/`, `models/item/New folder/`, placeholder author
   `TheArchitects, _______`.
10. Unverified risk: the mixin class implements the client-side interface
    `IForgeDimensionSpecialEffects`; check that a dedicated server still loads.
11. Placed-feature files are misspelled (`glithced_…`, `heeted_…`) but consistently
    referenced, so they work — rename both sides together if ever touched.
