# Custom Model Data leltár (resource pack-hez)

Az IceSMP minden egyedi itemje **numerikus custom model data (CMD)** értéket kap a Paper
`CustomModelDataComponent` API-ján át (float-listaként, pl. `1001.0`). Az alábbi táblázat a
teljes, kódból ellenőrzött leltár — egy resource pack ezekre az érték+alapanyag párokra tud
modellt kötni.

## Teljes lista (CMD szerint rendezve)

| CMD | Alapanyag | Item | Megjelenített név | Forrás |
|-----|-----------|------|-------------------|--------|
| 1001 | PAPER | Piros valuta | Parázsló Parals | `data/CurrencyType.java:6` |
| 1002 | PAPER | Kék valuta | Hópihér-veret | `data/CurrencyType.java:7` |
| 1003 | PAPER | Semleges valuta | Creutzér | `data/CurrencyType.java:8` |
| 1004 | PAPER | Sötét valuta | Csontveret | `data/CurrencyType.java:9` |
| 4101 | GOLDEN_AXE | Mételytépő relikvia | Mételytépő | `managers/RelicManager.java` |
| 4201 | ELYTRA | Szárny-relikvia (Piros) | Főnix-szárny | `managers/RelicManager.java:102` |
| 4202 | ELYTRA | Szárny-relikvia (Kék) | Zúzmara-szárny | `managers/RelicManager.java:110` |
| 4203 | ELYTRA | Szárny-relikvia (Semleges) | Vándorszél | `managers/RelicManager.java:118` |
| 4204 | ELYTRA | Szárny-relikvia (Sötét) | Csontszárny | `managers/RelicManager.java:126` |
| 5201 | ENCHANTED_BOOK | Lélekkapocs — Varázsló | Caldesterai Rúnakódex | `items/CatalystItemFactory.java:52` |
| 5202 | GOAT_HORN | Lélekkapocs — Harcos | Sárkánykirály Kürtje | `items/CatalystItemFactory.java:56` |
| 5203 | RABBIT_HIDE | Lélekkapocs — Íjász | Soleil Vadásztarsolya | `items/CatalystItemFactory.java:59` |
| 5204 | FLINT | Lélekkapocs — Orgyilkos | Homály-szilánk | `items/CatalystItemFactory.java:62` |
| 5205 | OAK_SAPLING | Lélekkapocs — Druida | Aetrinita Sarja | `items/CatalystItemFactory.java:65` |
| 5206 | BELL | Lélekkapocs — Paladin | Hajnaltűz Harangja | `items/CatalystItemFactory.java:68` |
| 5207 | WITHER_SKELETON_SKULL | Lélekkapocs — Halállovag | Néma Rúnakoponya | `items/CatalystItemFactory.java:71` |
| 5208 | TOTEM_OF_UNDYING | Lélekkapocs — Sámán | Ősvihar Totemje | `items/CatalystItemFactory.java:74` |
| 5209 | BAMBOO | Lélekkapocs — Szerzetes | Élet Ága | `items/CatalystItemFactory.java:77` |
| 5210 | WHITE_CANDLE | Lélekkapocs — Pap | Asterlayna Gyertyája | `items/CatalystItemFactory.java:80` |
| 5211 | SOUL_LANTERN | Lélekkapocs — Boszorkánymester | Kárhozat Lámpása | `items/CatalystItemFactory.java:83` |
| 5212 | ENDER_EYE | Lélekkapocs — Démonvadász | Hasadék Szeme | `items/CatalystItemFactory.java:86` |
| 5213 | DRAGON_BREATH | Lélekkapocs — Idéző | Sárkányvér-fiola | `items/CatalystItemFactory.java:89` |
| 6000 | IRON_NUGGET | Egyedi szakma-alapanyag | Tiszta Vasesszencia | `config/profession-materials.yml` |
| 6001 | GLOW_BERRIES | Egyedi szakma-alapanyag | Gyógy-kivonat | `config/profession-materials.yml` |
| 6002 | COPPER_INGOT | Egyedi szakma-alapanyag | Rezgő Rézötvözet | `config/profession-materials.yml` |
| 6003 | STRIPPED_OAK_WOOD | Egyedi szakma-alapanyag | Keményfa Gerenda | `config/profession-materials.yml` |
| 6004 | GLOWSTONE_DUST | Egyedi szakma-alapanyag | Rúnapor | `config/profession-materials.yml` |
| 6010 | PHANTOM_MEMBRANE | Mob-only alapanyag | Vad Esszencia | `config/profession-materials.yml` |
| 6011 | ECHO_SHARD | Mob-only alapanyag | Szörny Mag | `config/profession-materials.yml` |
| 6012 | SCULK_VEIN | Mob-only alapanyag | Árnyékpor | `config/profession-materials.yml` |
| 6013 | NETHER_STAR | Mob-only alapanyag (boss) | Fekete Villám Szilánk | `config/profession-materials.yml` |
| 6101 | RED_MUSHROOM | Vadgomba-bomba (druida spell) | — | `spells/WildMushroomSpell.java` |
| 6102 | STONE | Rúnakő (halállovag spell) | — | `spells/RuneStrikeSpell.java` |
| 6103 | SUGAR | Démoni Só (boszorkánymester spell) | — | `spells/DemonicCircleSpell.java` |
| 6104 | STICK | Kiűzés Botja (szerzetes spell) | Kiűzés Botja | `spells/ExpelHarmSpell.java` |
| 6201 | TRIPWIRE_HOOK | Kereskedő Kulcs (crate) | Kereskedő Kulcs | `config/crates.yml` |
| 6202 | TRIPWIRE_HOOK | Kincses Kulcs (crate) | Kincses Kulcs | `config/crates.yml` |

Összesen **33 hozzárendelés, mind egyedi** (a Mételytépő korábban az 1001-en
osztozott a Parázsló Paralsnel; 4101-re számoztuk át, ami a relics.yml-ben dokumentált érték).

## Tartományok

| Tartomány | Cél |
|-----------|-----|
| 1001–1004 | Frakció-valuták (PAPER) |
| 4101 | Mételytépő (GOLDEN_AXE) |
| 4201–4204 | Frakció-szárnyak (ELYTRA) |
| 5201–5213 | Kaszt-Lélekkapocsok (kasztonként saját alapanyag) |
| 6000–6004 | Szakma által gyártott egyedi köztes alapanyagok |
| 6010–6013 | Csak-mobból-eső egyedi szakma-alapanyagok |

## CMD nélküli egyedi itemek (csak PDC-tag azonosítja őket)
- Befogó eszközök (`CaptureItemFactory`): LEAD ill. GHAST_TEAR
- Ostromágyú (`SiegeWeaponFactory`): TNT_MINECART

Ha ezekhez is modell kell, előbb CMD-t kell kapniuk a factory-jukban.

## Config/kód viszony
- Az `economy.yml` `model-data` és a `relics.yml` `custom-model-data`/`material` kulcsait a
  kód **nem olvassa** — a tényleges értékek a fenti Java-forrásokban vannak hardkódolva; a
  config-értékek dokumentációként vannak velük szinkronban tartva (a Mételytépő
  GOLDEN_AXE + 4101 mindkét helyen). Resource pack készítésénél ez a táblázat a mérvadó.
