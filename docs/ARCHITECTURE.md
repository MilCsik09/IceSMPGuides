# IceSMP — Fejlesztői architektúra- és bővítési útmutató

> **Cél:** hogy a rendszer *átlátható, karbantartható és könnyen bővíthető* legyen. Ez a dokumentum
> a tényleges kódra épül: leírja, hogyan áll össze a plugin, milyen mintákat követünk, és
> lépésről lépésre **hogyan adj hozzá új tartalmat** anélkül, hogy bármit eltörnél.
>
> Kapcsolódó dokumentumok: [`README.md`](../README.md) (áttekintés),
> [`PLAYER_GUIDE.md`](PLAYER_GUIDE.md) (játékos-kézikönyv),
> [`ADMIN_GUIDE.md`](ADMIN_GUIDE.md#release-acceptance-checklist) (tesztelés) és
> [`ROADMAP.md`](../ROADMAP.md) (nyitott fejlesztések).

---

## Frakció és Suttogó: tiszta indulási szerződés

A 2026-09-07-i döntés szerint nincs régi szerveradat-migráció. Az aktív szerep,
nyomok, kilépés és várakozás továbbra is a FACTION PlayerProfile-szekció WAL-ján él.
Az eseményazonos nyom és annak felhasználása egy tranzakció; a kilépés nem módosítja
a jogi mezőket. `WhisperSightline` korlátos voxelbejárást oszt a blokkok saját
régióira; az aktor állapota kizárólag saját entity-scheduleren készített pillanatkép.
A jelölt tanúellenőrzése után az erőforrások újraellenőrzése megelőzi a rítusbizonylatot.
A régi tagságiadó-helper nem része a kassza működésének. Ismeretlen, kevert rítusmentés
adminvizsgálatot kér; az újraindítási integritás nem kompatibilitási adapter.

## 1. Nagy kép — életciklus

```
IceSMP (JavaPlugin)            ← Bukkit/Paper belépő (onEnable/onDisable)
  └─ IceSMPCore                ← a teljes rendszer összeszerelése
       ├─ konstruktor          → ~96 manager felépítése (szigorú sorrend), registerSpells()
       ├─ enable()             → config + perzisztens store-ok betöltése, listenerek + parancsok
       │                         regisztrálása, ütemezett feladatok indítása
       └─ disable()            → perzisztens store-ok mentése, majd futó rendszerek leállítása
```

- **`IceSMP`** (`hu.taliann.icesmp.IceSMP`): csak delegál a `IceSMPCore`-nak.
- **`IceSMPCore`** (`core/`): az egyetlen „összeszerelő" osztály. Itt jön létre minden manager,
  itt regisztrálódik minden spell (`registerSpells()`), parancs (`registerCommands()`) és
  listener (`registerListeners()`), és innen indulnak az ütemezett feladatok.
- **Folia-kompatibilis** (`folia-supported: true`): **nincs** globális fő-szál. Minden szálkezelés
  a megfelelő Folia ütemezőn megy (lásd 4. szakasz). Ez nem opcionális — a rossz szálon végzett
  entitás-hozzáférés crashel.

---

## 2. Csomagtérkép

| Csomag | Fájlok | Szerep |
|--------|-------:|--------|
| `core/` | 5 | `IceSMPCore` — összeszerelés, életciklus, ütemezés — + az élő config-apply hidak (`ConfigRuntimeReloadBridge`, `AdvancedConfigRuntimeBridge`) és a megőrzött Paper-parancsok életcikluskapuja (`CommandLifecycle`). |
| `managers/` | 128 | Üzleti logika és állapot (gazdaság, frakciók, kasztok, szakmák, loot/raritás, recept-katalógus, pet, territórium-védelem, stb.). |
| `listeners/` | 126 | Bukkit eseménykezelők (gameplay + GUI-klikk + loot/craft/védelem + esemény-spawn debug); a procedural daily listenert az authored quest authority kiváltotta. |
| `spells/` | 61 | Spell-rendszer: `Spell` SPI, `BaseSpell`, `ConfiguredSpell` builder, `SpellCatalog`, egyedi spellek. |
| `commands/` | 96 (65 + al-csomagok) | Parancsok. A `commands/<terület>/` al-csomagok a dispatch-stílusú alparancsokat tartják. |
| `classrelic/` | 14 | Class Relic Framework: pure resolver/katalógus/jelzések + Paper homlokzat (`ClassRelicService`). |
| `quest/` | 11 | Quest Framework v2 pure magja: forrás-policy + kontextus, kategória/láthatóság szótárak, gráf-validátor, választó-token registry, marker-paletta, közös quest-valuta resolver, az izolált content-integrity runtime probe, valamint az első belépés üdvözlő-szövegének egyetlen szabálya (`OnboardingWelcomeCopy`: canonical copy + elavult stock-config felismerése, custom szöveg érintetlenül). |
| `gui/` | 72 | Inventory-menük + `GuiUtil` közös helperek + adat-vezérelt `CommandMenu` rendszer + staged config-editor lapok (root/kategória/operational/world/crate + reward-editor). |
| `crates/` | 14 | Dependency-free crate domain: strict validáció, selector/key plan, atomi opening lifecycle, recovery/kompenzáció, scheduler gate, audit és thread-safe formázás. |
| `factions/` | 18 | Immutable passzív-config snapshot, tiszta damage/exhaustion/target policy, központi combat-marker katalógus, mobkontextus-resolver, mulandó retaliation state és a központi frakció-névszín paletta; a tartós tagság és bűnállapot a PlayerProfile faction szekciójában él. |
| `data/` | 15 | Enumok és értékobjektumok (`CurrencyType`, `FactionType`, `JobType`, `SpecializationType`, `Territory`/`TerritoryType`, `BlockCuboid`…). |
| `relics/` | 12 (9 + `ability/`) | Relikvia-keret: `RelicRegistry`, `RelicDefinition`, triggerek, transfer-elvárás, immutable világ-pillanatkép + single-writer store. |
| `items/` | 14 | Item-gyárak (katalizátor/Lélekkapocs, befogó item, tervrajz, egyedi alapanyag…), viselhető és közös ritkaság-prezentáció. |
| `trash/` | 48 | A 330 elemű Ócska katalógus és 27 lifecycle phase, item factory, kategória-első/context-súlyozott loot-választó, fishing/mob/ambient források, singleton history/state split, bounded delta-journalos history authority, a 42 zárt anomaly behavior és a 23 zárt consuming behavior bounded Folia runtime-ja, crash-safe spatial-fracture journal, a Profile v2-backed rejtett régészeti tudásrendszer és player-only tooltip bridge, identity-mentes aggregált runtime telemetry, opt-in Paper/Folia smoke probe, Felvásárló- és tartós recycle-integráció, valamint a rejtett diagnosztika. |
| `security/` | 1 | Immutable, permissiontől és OP-státusztól független fejlesztői authority a rejtett tartalomfelületekhez. |
| `warrior/` | 2 | Harcos gameplay vertical slice: transiens harci állapot + konkrét runtime (Csatatempó, Berserker, Guardian). |
| `evoker/` | 2 | Sárkányidéző gameplay vertical slice: transiens állapot + konkrét runtime (Felerősítés, Vörös–Kék Eszencia, Visszhang/Időlenyomat). |
| `archer/` | 3 | Íjász gameplay vertical slice: transiens állapot + konkrét runtime (Szélolvasás, Pontossági lánc, Kötelék) + a repülő nyilak korlátos, magától lejáró fegyelem-nyilvántartása (`ArcherShotLedger`). |
| `shaman/` | 2 | Sámán gameplay vertical slice: transiens állapot + konkrét runtime (Totemkerék-rezonancia, Maelstrom-ritmus, Dagály↔Apály). |
| `monk/` | 2 | Szerzetes gameplay vertical slice: transiens állapot + konkrét runtime (Áramlás, Harcművészeti Lánc, Stagger, Ködszál). |
| `paladin/` | 2 | Paplovag gameplay vertical slice: transiens állapot + konkrét runtime (Meggyőződés/Eskü, Fényjelző, Ítélet-jelek, Pajzstöltet). |
| `demonhunter/` | 2 | Démonvadász gameplay vertical slice: transiens állapot + konkrét runtime (Kárhozat-terhelés, Lélektöredék/Momentum, Fájdalom/Sigil). |
| `druid/` | 2 | Druida gameplay vertical slice: transiens állapot + konkrét runtime (Harmónia/Évszak másodlagos mechanika, Természeti Erő primary resource, kombó+Szagnyom, Nap–Hold mérleg/Eclipse, Kéregrétegek/Gyökérháló, Mag→érés→Virágzás). |
| `priest/` | 2 | Pap gameplay vertical slice: transiens állapot + konkrét runtime (Litánia-versek, Engesztelés rekurzió-őrrel + pajzsháló, Velő/Osszárium, Őrület-Küszöb). |
| `deathknight/` | 2 | Halállovag gameplay vertical slice: transiens állapot + konkrét runtime (Rúnakör Vér/Fagy/Halál, fix méretű Vér Emlékezete, Fagyjelek, Dögvész + ghúl-mutáció). |
| `assassin/` | 2 | Orgyilkos gameplay vertical slice: transiens állapot + konkrét runtime (Lehetőség négy nyitányból, háromhelyes Toxinkészlet + Dózis, Észleltség/időkorlátos rejtőzés, korlátos Járvány-nyilvántartás). |
| `warlock/` | 2 | Boszorkánymester gameplay vertical slice: transiens állapot + konkrét runtime (Paktum/Lélekadósság, háromhelyes Átokgrimoár + Lélekfonal, Izzó Parázs/Túlhevülés). A Demonológus paktum NEM transziens: egyetlen authorityja a durable `demonologist.roster` companion névsor, amit a runtime csak a közös `ClassSpecCatalog.companionProjection` szabállyal olvas, és a `PetManager` companion-gatewayen keresztül, durable-first módon mutál. |
| `wizard/` | 2 | Varázsló gameplay vertical slice: transiens állapot + konkrét runtime (Rúnaszövés öt tételes párral, három ráhangolódás Konvergenciával/Elemi Koronával; a lecsengés rögzített horgonyból számol, ezért lekérdezés-gyakoriságtól független). A Holtak Udvara NEM transziens: egyetlen authorityja a durable `necromancer.court` companion névsor, és ugyanaz a felvételi szabály (`ClassSpecCatalog.admitsCompanion`) dönt a cast előtt és a commitban. |
| `storage/` | 10 | `YamlStore` (atomikus írás) + `PersistentStore` SPI + fail-closed életciklus-koordinátor. |
| `session/` | 1 | `PlayerStateCleanup` SPI (per-player állapot takarítása). |
| `utils/` | 28 | `MessageManager`, `ExperienceUtil`, `TerritoryDestination`, `PlatformCapabilities`, egyebek. |
| `integration/` | 6 | Soft-depend reflexiós hidak: PlaceholderAPI, LibsDisguises, FancyNpcs, WorldGuard, LuckPerms. |

---

## 3. Architektúra-minták (ezeket kövesd)

A rendszer egységes mintákra épül. **Új kódnál mindig a meglévő mintát használd** — ne vezess be
párhuzamos megoldást.

### 3.1 Konfiguráció — több-fájlos merge
`ConfigManager.load()` egyesíti a `config/<alrendszer>.yml` fájlokat (alapértékek), majd rájuk
olvassa a fő `config.yml`-t (override, ez nyer). A betöltött fájlokat a `CONFIG_FILES` tömb sorolja
fel. Minden hívó a megszokott `getInt/getDouble/getString("alrendszer.kulcs", default)` API-t
használja — a kulcs-útvonalak a fájlok között oszthatatlanok.

A `config.yml`-t az **ingame config-vezérlés** is ezt a réteget írja: `/icesmp config
get|set|unset|list|find` (node: `icesmp.admin.config`) bármely kulcsot lekér/felülbírál/töröl,
set/unset után azonnali reload + `ConfigValidator` fut. Mivel a managerek túlnyomó része
használat idején olvassa a configot, a legtöbb érték azonnal él. A `spell-balance.<id>.*`
kulcsok (cooldown, cost-amount, resource-cost, damage, radius, range, self-damage, heal-self,
feed-self, ignite-/freeze-ticks, knockback) kivétel nélkül CAST-időben olvasódnak
(`BaseSpell.balance` + a `ConfiguredSpell` live-accessorai + `ResourceManager.costOf`), tehát
a deklaratív spelleknél sem kell restart. Ami továbbra is indításkor dől el: a scheduler-tick
periódusok, a parancs-/listener-regisztráció és a konstruktorban cache-elt értékek.

Betöltés után a `ConfigValidator.validate(...)` **konvenció-alapú** ellenőrzést futtat a teljes
kulcstéren (soha nem dob, csak a konzolra figyelmeztet): a `material`/`materials` kulcsok valós
`Material`-t adnak-e, a `currency` kulcsok `OWN`/valuta-nevek-e, a `…percent` kulcsok a 0–100
tartományban vannak-e, a `…-minutes/-hours/-seconds/-ticks/-millis` kulcsok nem negatívak-e. Így az
admin-elgépelések (rossz item-név, kilógó százalék) tiszta log-figyelmeztetésként jelennek meg
ahelyett, hogy némán az alapértékre esnének vissza.

#### 3.1.1 Natív szerverlista-MOTD — immutable snapshot + generációkapu

A `MotdListener` nem olvas fájlt és nem járja be a konfigurációt a server-list ping szálán.
A `/icesmp reload`, a `motd.*` config-parancs és a config GUI ugyanazon célzott reload-hookot
hívja: a listener előbb szigorúan felépít egy immutable snapshotot, azonnal üríti a korábbi
ikoncache-t, majd külön async taskban csomagolja ki és olvassa a `plugins/IceSMP/icons/*.png`
fájlokat. A könyvtár és minden fájl `SecureDirectoryStream` handle-en, `NOFOLLOW_LINKS` mellett
nyílik meg; a méretellenőrzés és a dekódolás ugyanazon fájldescriptoron fut. A Bukkit
`CachedServerIcon` létrehozása a global-region scheduleren történik.

- választási mód: időalapú vagy seedelt, időablakon belül stabil random;
- eseményprioritás: vérhold → világboss → szezonzárás → normál pool;
- tokenek: kizárólag `{online}` és `{max}`; minden más brace-token config hiba; opcionális max-player override;
- a vanished count kizárólag a moderációs `VanishManager` thread-safe UUID-cache-ét használja;
- ikonmód: `NONE`, `DEFAULT`, `VARIANT`, `RANDOM`;
- ikonkapuk: symlinkmentes root/köztes/fájl útvonal, jóváhagyott data-rooton belüli secure open,
  legfeljebb 1 MiB és 64 fájl, valódi PNG, pontosan 64×64;
- a reload-generáció és a `SchedulerCallbackGate`-et újrahasznosító `MotdGenerationGate`
  megakadályozza, hogy régi, visszautasított vagy disable után befejeződő callback publikáljon;
  az ikonmap és a rendezett ID-lista egyetlen volatile immutable cache;
- hiányzó scalar a dokumentált defaultot használja; jelen lévő hibás boolean, lebegőpontos vagy
  tartományon kívüli egész, hibás enum, üres/túl nagy pool, duplikált normalizált ID és hibás
  strict MiniMessage csak a MOTD feature-t tiltja le, nem a teljes plugint.

A dependency-free `MotdSelector` tesztelhetővé teszi a rotációt és eseményprioritást. A
`motdRegressionTest` a negatív epoch floor-mod viselkedést, a random stabilitást/pool-lefedést,
a teljes signed-`long` és strict boolean szabályokat, a placeholder whitelistet, a symlink/TOCTOU
ikonvédelmet, a generációs interleavinget és a jarban szállított ikonok 64×64 dekódolását is ellenőrzi. Ez nem helyettesíti a valódi Folia ping/reload és proxy nélküli runtime playtestet.


### 3.1.2 Natív HUD scoreboard — konfigurálható layout

A jobb oldali natív scoreboard sorait a `hud.sidebar.layout` lista írja le; a dinamikus
játékállapot nem akadálya a szerkeszthetőségnek. A `text`, `spacer`, `separator`, `target`,
`resource`, `info` és `party` sortípusok sablonjai futásidőben kapják meg a dokumentált
`{token}` értékeket. A fejléc címe és a layout reload után élőben frissül, hibás vagy hiányzó
lista esetén pedig a beépített alapelrendezés lép életbe.

A teljes, már kibontott layout legfeljebb 15 scoreboard-sort használ. Túlcsorduláskor a
`hud.sidebar.eviction-order` szerinti opcionális szekciók esnek ki; a combat target csak harcban,
a resource csak aktív kaszt-erőforrásnál, a party pedig tagonként bővül. Az alaplayout első
`spacer` sora választja el a resource-packből érkező cím-glyphöt a felső vonaltól. A glyph
`height`/`ascent` metrikája továbbra is a resource pack font-JSON-jának felelőssége.

### 3.1.3 Mob/Encounter 2.0 — authored réteg és survival fallback

A `pve/` csomag dependency-free domainje az authority a mob ID, schema, rank,
archetype, ability, affix, levelgörbe, encounter snapshot és contribution szabályokhoz.
A `MobTemplateRegistry` a `content/pve/enemies.yml` 93 elemű handcrafted katalógusát fail-fast tölti: invalid entity,
rank/archetype, hiányzó ability/loot profile, Bestiary ID-ütközés vagy schemahiba nem
eredményez részleges registryt. Természetes vanilla mobhoz nem kötelező template;
`MobTemplateRegistry.naturalTemplate` biome-, dimension-, depth-, night- és weather-tag
specificitás alapján választ. A `natural-context.required-any` OR-biome-családokat tud
megadni, így a biome-native variáns nem csak súlyozottan, hanem ténylegesen is kizárható
az inkompatibilis környezetből; üres találatán a vanilla fallback él tovább.

A level resolution precedenciája: encounter override → authored location → explicit
MobTemplate → survival földrajzi alap. Az utolsó réteg a wilderness-distance alap fölé
territory-, biome/dimension-, depth- és event/Vérhold-bónuszt tesz, majd 70-nél clampel;
a normál távolsági görbe önmagában 1–50. A 70 fölötti display level csak explicit
authored boss/encounter útvonalon engedett. A HP és damage külön, monoton és bounded:
alapértelmezésben `min(8, 1 + (level-1)×0.08)` és
`min(3, 1 + (level-1)×0.025)`, amelyre a template/rank szorzók kerülnek; abszolút
védőkorlát is érvényes. Az `EquippedCombatPowerService` a player owner-threadjén csak a
main/offhand és négy armor slot valid, UUID-duplikátummentes canonical itemjeit mintavételezi,
majd immutable cache-t publikál a cross-region boss-snapshotnak. A
`EquippedCombatPowerModel` tényleges statot, item levelt, Signature-tier kontextust, szettet és
rúnát ad a bounded `CombatPowerEstimator`-nak; malformed/stale/rossz slot fail-closed kimarad.
Az invalidálás inventory/equipment eseményvezérelt; a plugin saját mutation-, craft-, market-,
crate- és admin inventory útjai explicit owner-thread refresh hookot hívnak. Nincs periodikus
equipment polling. A set transient modifier stabil `NamespacedKey`-t használ, és refreshkor
eltávolítja az előző példányt az új hozzáadása előtt. Ez belső telemetry/snapshot input, nem
publikus gear score és nem loot-authority.

Az ability authority továbbra is a `MobAbilityDefinition` → `MobAbilityRegistry` →
`MobAbilityRuntime` lánc. A #137 tizenegy `Kind` technikája source-compatible maradt, mellette
nyolc jelenleg használt `COMPOSITE` definíció typed triggerből, legfeljebb nyolc conditionből
és legfeljebb nyolc actionből épül. A bounded vocabulary csak a jelenlegi contenthez szükséges
`DAMAGE`, `KNOCKBACK`, `DASH`, `RETREAT`, `GUARD` primitive-eket, valamint `ON_TIMER`,
`ON_COMBAT_ENTER`, `ON_PROVOKED`, `ON_DAMAGED` triggereket tartalmazza; nincs expression
language, általános scripting DSL vagy speciesenkénti Java mechanic. A target rule és az action
target külön typed mező. A registry hibás trigger/condition/action/ability referenciára fail-fast,
a runtime pedig cooldown, telegraph, recovery, interrupt és cast epoch mellett hajt végre.

A `CreatureSpeciesRegistry` a `content/pve/enemies.yml` egyetlen `creature-species` matrixát atomikusan
publikálja. Runtime teljességi authority a Paper 1.21.11 `EntityType.values()` azon halmaza, ahol
`isAlive && isSpawnable`, player nélkül; minden típusnak pontosan egy explicit row kell. A 91 row
közös level/rank/stat/ability authorityra vetít, és category, disposition, temperament,
provocation, social, reward, baby és tame policy szerint különbözik. Hiányzó runtime lookup
`NON_COMBAT/VANILLA_ONLY` fallback, invalid config pedig startup-hiba: random agresszió nincs.

A `CreatureProfileService` spawnkor PDC-be rögzíti a profile verziót, spawn source-ot,
dispositiont, temperamentet, stabil reakciót és reward profilt. A level/rankot ugyanaz a
`MobScalingManager` számolja Cow, Wolf, Zombie és Skeleton esetén; chunk load/restart nem reroll,
mert a meglévő PDC marker authoritative. PASSIVE soha nem kezdeményez player combatot pusztán
level vagy rank miatt. Valid provocation csak direkt player, player projectile vagy player-owned
tameable damage; környezeti sebzés, etetés, tenyésztés, fejés, nyírás, mount, tame és lead nem
provokáció. A UUID-seeded temperament és reaction entitynként stabil: az outcome `FLEE`, vagy a
config szerint `WARN/FIGHT`, nem hitenként új RNG.

`FIGHT` esetén a passzív creature ugyanabba a `MobAbilityRuntime` target/cooldown/cast/telegraph/
interrupt/cleanup életciklusba lép, mint a hostile mob. A korábbi `WildlifeRetaliationService` és
`WildlifeRetaliationPolicy` megszűnt, ezért nincs legacy+új double damage vagy double assist.
Timeout, invalid/logout target, death, unload, leash-szerű távolságvesztés és shutdown cast epoch
invalidációval bontja az authored combatot. A runtime legfeljebb 2048 aktív state-et tart, nincs
world scan vagy per-tick YAML parse. PASSIVE timer technique csak authored combatban, NEUTRAL
timer technique csak vanilla target mellett futhat; így Enderman/Wolf/Bee/Piglin vanilla trigger
identityje nem válik proximity aggróvá.

A social policy relationt, sugarat (max. 16), jelöltet (max. 32), asszisztenst (max. 6), szükséges
temperamentet és cooldown-t deklarál. A shipped Cow policy ennél szűkebb: 6 blokk, 12 jelölt,
2 asszisztens. Nincs rekurzív propagáció; a remote ally kizárólag saját entity schedulerén kap
state-et. Bee/Wolf/Goat/Llama vanilla social/AI authorityt tart meg. Baby alapból csak identityt,
nem combat kitet kap; owner-safe tameable az owner ellen nem lép authored combatba.

A combat profile és reward profile külön authority. A normal survival wildlife mindig
`VANILLA_ONLY`, tehát Elite Cow sem kap canonical gear-, soulstone- vagy class-XP faucetet.
Spawner, spawn egg, breeding, command és custom forrású hostile profile sem kap automatikus
faucetet; explicit event/template út `EXPLICIT_AUTHORED` markert használ. Rank a stat- és
technique-komplexitást növelheti, dispositiont nem. A `CombatTelemetry` csak bounded species,
provocation, outcome, social assist és technique aggregate-eket tart, PII nélkül.

Az engine szándékosan nem encounter DSL. Boss phase, wave, objective, branching, delay/repeat,
richer targeter és teljes threat authority a későbbi „Composable Encounter & Boss Authoring
Runtime” scope határa; új primitive csak konkrét IceSMP content use case miatt kerülhet ide.

A világboss startkor immutable résztvevő-snapshotot készít. A HP létszámgörbéje
`1 + 0.65×(n-1)^0.8` (configolt és capelt), a damage csak logaritmikusan, legfeljebb
1.18×-ra nő; late join hozzájárulhat, de a boss HP-ja nem ugrál. A bounded
`ContributionLedger` elutasítja a pre-combat és self-support paddinget. A Monk és Paladin
owner-thread heal/shield runtimeja tényleges ally-hatást jelent a ledgernek; a kijelölt
világboss-zónából kitérő, már aktív résztvevő bounded objective-et kap. A ledger egyszeri
settlement claimet ad, majd encounter-endkor lezár. A Profile-receipt alapú személyes
reward az Itemization 2.0 boss-component source authorityja.

Az item mutation crash policy közös exact snapshot-mátrixot használ reroll/rúna/ascension
művelethez: prepare előtti/utáni exact-before abort, inventory publish utáni exact-after
commit, mixed state kézi review. Az encounter reward PREPARED receiptje nulla markernél
kézbesít, egy exact markernél commitol, több markernél fail-closed kézi vizsgálat.
Rúnánál az insert, a kiválasztott foglalat remove-ja és az old→new replace egyaránt egyetlen
whole-inventory before/after WAL-bejegyzés. A replace nem két egymás utáni mutation;
UUID-t, provenance-t, ascensiont és a másik rúnát ugyanabban az immutable candidate-ben őrzi.

### 3.2 Üzenetek — több-fájlos merge + formátum-tudatos rendering
`MessageManager.load()` egyesíti a `messages/<csoport>.yml` fájlokat (a `MESSAGE_GROUPS` szerint),
majd a fő `messages.yml`-t override-ként. Rendering: a `get`/`getMessage`/`getComponent` **mind**
formátum-tudatos — **MiniMessage** ha a szövegben `<...>` tag van ÉS nincs legacy `&`/`§` kód,
egyébként legacy. Sose feltételezd egyik formátumot sem; használd a generikus API-t.

### 3.3 Perzisztencia — atomikus írás + életciklus SPI
- **`storage/PlayerInventoryCommit`**: a vanilla inventoryval együtt tárolt technikai
  nyugtát a tényleges playerdata-fájlból visszaolvassa, majd a fájlt és könyvtárát
  tartósítja. A `saveData()` visszatérése önmagában nem commit-igazolás; bizonytalan
  fizikai mentés után nem indulhat külső tárgyhatás vagy kifizetés. A személyes
  fejlődés és egyenleg továbbra is a PlayerProfile authorityjához tartozik.
- **`storage/YamlStore.saveAtomic(file, yaml)`**: egyedi temp-fájl + atomikus rename (konkurens-biztos).
  **Minden** YAML-mentés ezen át megy — soha ne `yaml.save(file)` közvetlenül.
- **`storage/PersistentStore { load(); save(); }`**: a 39 fájlt-író store implementálja. Az
  `IceSMPCore` egy `List<PersistentStore>`-t iterál: `load()` az enable-ben, `save()` a disable-ben
  (a player-cleanup ELŐTT, hogy ne vesszen adat).
- **`storage/PersistentStoreCoordinator`**: az enable során **fail-closed** tölti be a teljes
  registryt; az első hibánál az indulás megszakad, részlegesen betöltött állapot nem menthető.
  Autosave és shutdown csak a teljesen betöltött registryt írhatja, egymással szerializálva.
- **Write-ahead napló (WAL) — ahol a mentés-időpont nem elég:** két rendszernek a következő
  autosave-ig sem szabad kockáztatnia, mert közben a világból/inventoryból már eltűnt valami.
  - **`storage/BlockRegenJournal`** (block-regen.yml checkpoint + `block-regen.wal`):
    a tile-entity snapshot tartósan lemezre kerül a konténer kiürítése előtt, és a pending
    rekordok restart után újrapróbálhatók. Az `APPLYING/APPLIED` átmenet csökkenti az elvesző
    restore-ok esélyét, de valódi Folia + process-kill fault-injection nélkül nem állítunk
    pontosan-egyszeri konténer-NBT alkalmazást.
  - **`storage/TransactionJournal`** (market-journal.yml): a prepare és a szigorú séma
    jelentősen csökkenti a félbehagyott listing/pénz/item műveletek elvesztését, és normál
    restartnál recoveryt ad. A wallet, market YAML és player inventory között nincs formális
    több-store atomicitás vagy exactly-once bizonyítás; a globális currency gate külön
    egyszerűsítési és runtime-validációs scope.
  - **`storage/ItemMutationJournal`** (`item-mutation-journal.yml`): kizárólag a
    reroll/rúna/ascension/salvage egy-játékosos inventory-határára szolgáló szűk WAL, nem
    általános transaction framework. A domain előbb immutable candidate-et épít; a WAL
    exact teljes before/after inventory snapshotot ír, majd ugyanazon owner threaden
    payment+item publish és `player.saveData()` történik. Boot/join recovery csak a két
    exact állapotot fogadja el; mixed snapshot kézi review. Az itembe írt bounded operation
    receipt és revision védi a retry/double-click utat. Bizonytalan írás után a napló
    további módosítást nem fogad el; a tényleges lemezállapot újraolvasása szükséges.
  - **Encounter reward receipt/outbox** (PlayerProfile v2 `OPERATIONS`): a világboss
    meaningful-contribution küszöbénél először bounded eligibility receipt készül.
    Settlementkor ez COMMITTED állapotba kerül, majd a személyes delivery külön PREPARED
    receiptet kap. A sorrend `receipt → inventory → player.saveData() → COMMITTED`;
    full inventory nem dob tárgyat a földre, az exact markeres item reconnect után commitolható.
    A boss transient, ezért restart után a COMMITTED eligibility újrakézbesíthető, a csak
    PREPARED jelölt exact-before állapotként rollbackelhető.
  - **Frakcióváltás**: a `PlayerProfileFactionStore` a tagságot, historyt, díjat és
    szezonváltási számlálót egy PlayerProfile WAL-tranzakcióban rögzíti. A külön DARK-join
    ugyanabban a faction-szekció commitban ellenőrzi az Exile/Oath előfeltételt és a szezonplafont.
    Az adóproducer végleg üres; a régi outbox formátum kompatibilitási maradvány, nem új adóforrás.
  - **Suttogó**: a `PlayerProfileWhisperStore` egyetlen faction-szekció mutációban váltja be
    a tanú–gyanúsított bizonyítékot, lépteti a fokozatot és leleplezéskor rögzíti az Exile-t,
    a szerep megszűnését és a 24 órás visszatérési határidőt. Logout csak a routing cache-t törli.
    A rítus sorrendje: tartós intent → exact inventory/HP újraellenőrzés → owner-thread
    item/HP + `player.saveData()` → role commit → sikerjelzés. A két inventory-pillanatkép
    egyikével sem egyező helyreállítás zárolva marad, adminvizsgálattal; vak visszaadás nincs.
  - **Személyes szezonrészvétel**: faction-profilbeli, szezonra és tagsági időpontra kötött
    aktivitásnyugták. Kategóriánként és UTC-naponként egy minősített esemény számít. A
    betöltött projekció offline tagokra is megmarad; a szezon nem zárul a betöltése előtt.
    Létszám: az elmúlt hét minősített résztvevői; a pont-osztó `sqrt(max(1, aktív/reference))`,
    egész pontokra kerekítve, pozitív pontforrásnál minimum egy ponttal.

  - **Szezon–community generation commit** (`season.yml` → `community-goals.yml`): a community store tartós `season.number` markerrel jelöli, melyik szezonhoz tartozik a progressz. A zárás a community monitor alatt előbb rendezi az outboxot, majd commitolja az új `season.yml` generációt, és csak ezután nullázza/menti a community progresszt. Crash a két commit között egyetlen generációnyi marker-lemaradást hagy; bootkor ez idempotens resetként reconciliálódik. Függő régi payout, előreszaladt vagy több generációt átugró marker fail-closed.

- **DEV-item jutalom — arányos easter-egg state:** a Csodálatos Bingulus egyetlen runtime ownerhez
  kötött DEV-item, amely alapértelmezetten 10 perc aktív online birtoklás után sorsol random,
  konfigurált jutalmat. A manager egy immutable state-et tart (owner, singleton instance, issued,
  aktív idő, exact pending `ItemStack`, pity), egy lockkal és egy minimális tick gate-tel. A már
  kisorsolt exact item az inventory módosítása előtt a `dev-items-state.yml` fájlba kerül; teljes
  inventory és normál restart után ugyanaz próbálható újra.
- **Live owner reload:** `/icesmp reload` közben az új owner candidate state-je előbb kiíródik, majd
  válik aktívvá; az instance, progress, pity és pending jutalom megmarad. A tick az owner UUID-t a
  kezdéskor, az inventoryba adás előtt és a pending törlése előtt ellenőrzi. Mismatch esetén a régi
  tick egyszerűen visszatér. Nincs generation counter, owner-transition framework vagy tranzakciós
  rollback-protokoll.
- **DEV garanciahatár:** nincs receipt, grant ID, recipient binding, migration vagy exactly-once
  garancia. Process kill az inventory mutation és a completion YAML között, ritka write race,
  hardverhiba vagy extrém owner-transfer verseny esetén jutalomvesztés vagy duplikáció elfogadható.
  A DEV state hibája kizárólag a Bingulus progresszét, sorsolását és kiosztását állítja le; a market,
  wallet, currency és season store-ok ettől nem állnak le.
- **DEV regressziók:** a `devItemRewardRegressionTest` Gradle `JavaExec` task a `check` lifecycle
  része. Az intervalt, exact pending/restartot, full-inventory retryt, egyszerű owner reloadot,
  write-failure határt, strict state-et és a gate normal/exception/retired/rejection/null útjait
  teszteli. A `scripts/test_dev_item_state.py` csak tiltott legacy/overengineered tokeneket ellenőriz,
  majd ugyanezt a Gradle taskot hívja; nincs második, párhuzamos tesztrendszer. A tartós `IceSMP CI`
  workflow Java 21-en clean buildet, Gradle-suite markert, célzott Python futtatást,
  `git diff --check`-et és base/head consistency-deltát ellenőriz `contents: read` jogosultsággal.

### 3.3.1 Ócska loot-ökoszisztéma

- **Restart-only authority:** a `content/trash/catalog.yml` egyszerre tartja a 330 base identityt,
  a 27 authored lifecycle phase-t és a phase-transformation kapcsolatokat,
  a 6,5% fishing / 11% mob / 25% ambient source-esélyt, a pontosan 100%-os zárt
  kategóriasúlyokat, az identity-affinitásokat, a 8%-os displaced rollt, az 50%-os
  recycle-helyettesítést, valamint az ambient idő-/távolság-/TTL- és density-határokat. A `TrashCatalog`
  mindezt egy immutable snapshotként, fail-closed tölti be.
- **Category first:** a `TrashLootSelector` előbb kategóriát választ, utána épít cache-elt,
  source/context/displaced kulcsú identity-disztribúciót. A context ezért nem képes a zárt
  kategóriaesélyt módosítani. Luck, Looting, mobrank, boss és profession adat nem jut el a
  selectorhoz.
- **Források:** a fishing listener a hook régiójára ugrik és külön fizikai dropot ad; a mob
  listener a közös `MobKillUtil.FLAVOR` Survival/AFK/spawner/minion/synthetic gate-et használja;
  az ambient manager csak player-move eseményből ütemez, target-region threaden ellenőriz és
  spawnol, chunkot nem tölt be.
- **Ambient ownership:** a density index csak world/chunk koordinátát és entity UUID-t tart.
  A tényleges itemet mindig az owner scheduler módosítja; a chunk cap 1, a 3×3 cap 4. Claim,
  territory és WorldGuard-válasz fail-closed kizáró ok. Pickup/hopper leveszi a markert és
  felszabadítja a sűrűséghelyet; merge tiltott, TTL/shutdown entity scheduleren takarít.
- **Vendor/recycle:** a `BuyerService` a generic PDC-elutasítás előtt delegál a
  `TrashVendorService`-nek. Az authored apparent price a közös napi keretet és fizikai
  frakcióvaluta-kifizetést használja, felismerési warning nélkül. Az eladott, visszaforgatható
  history-bearing instance az atomi `trash-recycle.yml` store-ba kerül; csak a normál
  category/base-identity roll után, pontosan azonos base identityhez vehető ki, így a pool nem
  emel ritkaságot. Egy amountból nem klónoz instance tokent: minden eligible unit külön historyt kap.
- **History/state split:** a friss item csak stack-equivalent origin markert hord. Az első jelentős
  event egyetlen unitot választ le, opaque instance UUID-t és monoton history revisiont ír a fizikai
  itemre; a részletes események és owner-set a külön, atomi `trash-history.yml` authorityban élnek.
  Stale vagy duplikált token nem mutálható. A history store a recycle store előtt töltődik.
- **Lifecycle/repair:** az activation előbb singleton splitet végez, majd a catalog authored phase
  prezentációját alkalmazza és `TRANSFORMED` eventtel lépteti az authorityt. Pickup csak új ownert,
  death/Nether transit/Mending csak jelentős eventet rögzít; nincs tickes inventory scan. Minden
  ItemMeta/PDC írás után újraalkalmazódik az `ITEM_MODEL`, így a data-component prezentáció nem vész el.
- **Anomaly behavior authority:** a 42 catalog behavior egy zárt `TrashAnomalyBehavior` enumra
  validálódik, ezért hiányzó vagy ismeretlen viselkedés startupkor fail-closed hibát ad. A fizikai
  item továbbra sem hord kind- vagy behavior-markert; a runtime kizárólag az opaque base identityből
  oldja fel a belső definíciót. A zárt enum 16 typed primitívsávot különít el; ezek a dobás/fizika,
  contextual prezentáció, hang, konténer/inventory, redstone/mechanizmus, történeti felismerés és
  pair/memory nagyobb szerződései köré rendeződnek.
- **Bounded Phase D runtime:** világonként legfeljebb 256 anomaly item kap entity-scheduleres fizikát,
  a seek sugár legfeljebb 12 blokk és iterációnként legfeljebb 24 entity; delayed echo-ból globálisan
  legfeljebb 256 lehet. Nincs chunk load, globális entity/inventory scan vagy legacy Bukkit scheduler.
  A mechanizmus-attachment claim- és territory-preflight után singleton instance-ként kerül a világba,
  a pontos fogadó blokk következő authored rising edge-jét egyszer nyeli el, majd a catalog success
  phase-ébe transzformálódik. Legfeljebb 128 attachment élhet, a tartós lejárat 5 perc; fogadócsere,
  lejárat vagy shutdown feloldja a rögzítést, chunk-visszatéréskor ugyanaz a korlát érvényes.
  A stopper és a lokális death counter bounded, atomi `trash-anomaly-state.yml` authorityban él.
- **Rejtett régészeti tudás:** a `HiddenDiscipline.ARCHAEOLOGY` nem `ProfessionType`, nem foglal
  profession slotot és nem kapcsolódik combat/craft/loot/vendor bónuszhoz. A 30 tickes Brush-session
  a Brush-sal ellentétes kézben tartott tárgy snapshotját vizsgálja, mindkét kézelrendezésben; korai item-use release, kéz/slot/inventory változás,
  drop, halál vagy session-teardown megszakítja. A nyomva tartott jobb gomb ismétlődő interakciója
  frissíti a sessiont; 8 tick inputhiány megszakítja, a befejezéshez a 30. tick utáni friss input kell.
  Natív Brush-use nem indul, így a vizsgálat nem kefél világblokkot. A katalógus kézzel írt anyagi
  megfigyeléseket, 75 történeti tárgyhoz 2–2 egyedi tényt és 25 finom anyagi ellentmondást tartalmaz;
  nem a technikai hordozóanyagból vagy loot-súlyokból következtet. Ismeretlen tárgy vizsgálható,
  de nem ad kitalált történeti tényt vagy insightot. A family/domain/familiarity/insight és a bounded
  knowledge-signature ledger a canonical Profile v2 `AchievementSection.extensions` CAS-írásán él.
  Duplicate signature nem ad insightot, az unlock a már korábban teljesült breadth után érkező új,
  magasabb rendű facthez kötött, a szint küszöbe `round(0.55*l² + 4.5*l)` és legfeljebb 50.
- **Régészeti prezentáció:** a canonical item lore-ja nem változik. A verzió-pinnelt
  `TooltipPacketBridge_1_21_11` a vizsgált kéz eredeti menüslotjának player-only display copyját küldi;
  inventory transaction előtt canonical resync történik, runtime probe-hibánál pedig szöveges
  fallback működik. Disconnect, reload és slot change takarítja az overlay/session állapotot.
- **Hardening telemetry:** a runtime kizárólag összesített behavior-error, inspection
  start/complete/cancel, unlock és text-fallback számlálókat tart. Item identityt, holdert,
  hidden kindot vagy behavior-paramétert nem tárol és nem logol; a snapshot csak a hardcoded DEV
  authority mögötti staging diagnosztikában jelenik meg.
- **Secret surface:** a 42 belső identity, behavior és állapot nem kerül player/admin/feature/changelog/lore
  dokumentációba, normál logba vagy chatre. A contextual mondatok kizárólag a jogosult item viselkedésének
  pillanatnyi, player-only prezentációi; az item canonical neve/lore-ja és stack-equivalence-e nem változik.
- **Nincs runtime gate:** a rendszer nem kap master vagy ambient enable kapcsolót és nem jelenik meg
  az admin config GUI-ban. A stack addig marad draft/unmerged, amíg a teljes implementáció elkészül;
  a runtime density limitek is a restart-only Git-authored catalog részei.

### 3.4 Parancsok — két stílus
- **Dispatch (preferált, alparancsos):** `AbstractDispatchCommand` bázis + `Subcommand` SPI.
  A bázis kezeli a map-et, a diszpécst, a helpet és a tab-complete-et; a parancs a konstruktorára
  zsugorodik (lásd `CurrencyCommand`, `JobCommand`, `FactionCommand`, `BankCommand`). Üzenet-kulcsok:
  `messages.<név>-unknown-subcommand`, `messages.<név>-help-header`, `messages.<név>-help-<alparancs>`.
- **Egyrészes / implicit-default:** néhány parancs (Market, Pet, Soul, Spell, Events…) üres argra
  műveletet végez (nem helpet ad), vagy nem `args[0]`-ra diszpécsel. Ezek szándékosan külön
  `BasicCommand`-ok — a dispatch-bázis nem modellezi ezt a szemantikát.
- **Permissionök:** kanonikus séma a `core/Permissions` osztályban (konstansok + `register()` az
  `enable()` elején). Minden admin-node `icesmp.admin.<domain>` (default: OP), az
  `icesmp.admin.all` regisztrált szülő-node az összeset megadja egyben; a régi nevek
  (`icesmp.admin`, `icesmp.job.admin`, `icesmp.currency.admin`, `icesmp.faction.admin`,
  `icesmp.relic.admin`) alias-Permissionként a kanonikus gyereküket adják — meglévő
  LP-beállítás nem törik. Új admin-parancsnál: konstans a `Permissions`-be + a `register()`
  canonical-map-jébe egy sor.

### 3.5 Spellek — registry + builder + katalógus
- **`SpellRegistry`**: id → `Spell` map (`register`, `getById`, `getAll`).
- **`Spell` SPI** (`spells/Spell.java`): id/név/cooldown/költség + `executeSpell()` (true = hatás
  történt; false = no-op → nincs költség/cooldown) + `describe()` (spellbook-leírás) + `clearPlayerState()`
  (per-player takarítás, alapból no-op).
- **`ConfiguredSpell.builder(...)`**: adat-vezérelt spellek kód nélkül — láncolható hatások
  (`damage`, `healSelf`, `selfEffect`, `targetEffect`, `ignite`, `freeze`, `knockback`, `dash`,
  `particle`, `sound`, `aoe`, `target`, `friendly`…). A számok automatikusan a `describe()`-ba kerülnek.
- **`SpellCatalog`**: a kaszt-/spec-spellkészletek deklaratív regisztrációja (`ConfiguredSpell`-ekből).
- **Egyedi (bespoke) spellek**: ha a hatás nem fér a builderbe (pl. `HideSpell`), `extends BaseSpell`.
- **Config-driven balansz-felülbírálás** (`config/spells-balance.yml`): a `spell-balance.<id>.*` kulcsok
  **LIVE_READ**-ek — a `ConfiguredSpell` accessorai (`getDamage`, `getRange`, `getRadius`, …) és a bespoke
  spellek `BaseSpell.balance()` / `balanceInt()` segédei is CAST-időben olvassák a configot, ezért
  `/icesmp reload` után restart nélkül élnek. A `IceSMPCore.applySpellBalanceOverrides()` (`enable()`,
  `configManager.load()` után) csak az indulási log és az ismeretlen spell-id figyelmeztetés miatt fut le
  (`ConfiguredSpell.withBalanceOverrides`, immutable copy). **RESTART_ONLY** marad, ami nem érték, hanem
  szerkezet: a spell-regisztráció maga (új spell/unlock-lista), a scheduler-tick periódusok és a
  konstruktorban cache-elt értékek.

### 3.6 GUI — közös helperek + adat-vezérelt menük
- **`GuiUtil`**: közös item-/lore-építők (`icon`, `filler`, `fill`, `label`, `accent`, `grey`).
  Új menü-ikonnál ezeket használd, ne építs inline `ItemMeta`-t.
- **`CommandMenu` rendszer** (adat-vezérelt): a legtöbb menü a `CommandMenus` definíciókból + a
  `CommandMenuHolder`/`CommandMenuListener` párosból épül. Új „gombmenühöz" ezt preferáld a
  bespoke GUI helyett.

### 3.7 Player-state takarítás — registry-iterált
A `PlayerSessionCleanupListener` kilépéskor/kickkor: (a) végigmegy a regisztrált
`List<PlayerStateCleanup>`-on (managerek), és (b) a `SpellRegistry.getAll()`-on, minden spell
`clearPlayerState(uuid)`-jét hívva. A spell-ágon **nincs hardkódolt lista** — új állapotos spell
automatikusan bekerül; a manager-ág viszont kézzel karbantartott konstruktor-lista (lásd 5.7/5.8
recept: új állapotos managert fel kell venni a `stateOwners` listába).

### 3.8 Kaszt-erőforrás (`ResourceManager`) — hibrid költség
Per-kaszt „erő" 0–max meter, a HUD-oldalsávban megjelenítve (`HudManager.buildLines` hív egy
`hudLine`-t — **nem** külön boss-bar, hogy ne ütközzön a világboss-sávval). A csík **lazy módon
regenerálódik** (minden hozzáférés krediteli az eltelt időt — nincs scheduler), UUID-kulcsos
concurrent map (Folia-safe, nem nyúl entitáshoz a saját szálán kívül). `PlayerStateCleanup`-ot
implementál.

**Hibrid költségmodell** — `ResourceManager.usesResource(spell)` dönti el spellenként, mi a költség:
- `HEALTH` → marad HP (vér-mágia);
- `XP ≥ xp-ritual-threshold` (alap 80) → marad XP (nagy rituálé/idézés/időjárás/ulti);
- `HUNGER ≥ hunger-heavy-threshold` (alap 8) → marad éhség (nehéz fizikai);
- minden más → a kaszt-erőforrás.

A cast-pipeline (`AbilityCatalystListener`) ez alapján ágazik: `usesResource` spellnél
`canAfford`/`consume`/`refund` a `ResourceManageren` (a költség `Spell.getResourceCost()`,
cooldown-szint alapján); egyébként a spell saját `hasRequiredCost`/`consumeCost`/`refundCost`
(éhség/XP/HP) útja. Ha `spells.resource.enabled=false`, MINDEN spell a régi éhség/XP/HP útra esik.

> A korábbi „teli állapotban kirobbanás + empowered ablak" jutalom-mechanika **megszűnt** — a csík
> most költség (spend-modell), ami ugyanazon a sávon kizárta a build→discharge-ot.

### 3.8.1 Kaszt/spec rework — Profile v2 authority és adapterhatárok

A 13 kaszt / 35 specializáció reworkje elkészült és a Profile v2 mindig aktív,
egyetlen kaszt/spec authorityjára épül; nincs legacy gameplay fallback vagy
runtime rollout-kapcsoló. Az `IceSMPCore.enable()` a gameplay store-ok
betöltése előtt futtatja a `ClassSpecDependencyPreflight` ellenőrzést. Aktív
dependency enforcement mellett a hiányzó vagy verzióeltérő kötelező komponens
fail-closed startup hibát okoz, nem félkész class runtime-ot.

A játékosfelületek nem olvassák külön-külön a loadout mezőket. A
`ClassProgressView` a `ProfileDiagnostic` és az aktuális `ClassSpecSection`
read-only projekciója: mindkét slot, doctrine, mastery, capstone és seal egy
snapshotban jelenik meg. A vanilla Kasztműhely ezt fogyasztja; ugyanennek a
projekciónak kell maradnia a későbbi native kliensfelület bemenetének is, így
a kliens nem válhat új authorityvá.

Az inventory-prezentáció közös resource-pack szerződése a `ClassUiAssets`:
nyolc képernyőtípus négy frakciótémával, 13 kasztjelvény és 35
specializáció-jelvény. A generált bitmap font csak megjelenítés; a menük az
aktuális Profile v2, `ResourceManager`, class-mechanika és relic service
read-only projekciójából építߎwښ$z{-�霪םr, Blood Moon és spawn reason környezet. `MobNaturalContext` required/excluded tagot, opcionális affinityt, relatív weightet és ±12 level offsetet tárol. Nincs koordináta-roster, world progression, local danger, kill heat vagy ecology memory.

### Daylight Undead

A nappali felszíni natural undead template explicit `no-daylight-burn` forrást kap; night/deep-only undead nem. A `DaylightProtectionPolicy` az authored, territory és event forrást OR-semantikával kompozálja. Az első védelem előtt rögzített carrier baseline csak az utolsó forrás megszűnésekor áll vissza; nincs helmet/equipment workaround.

### Technique Design és Telegraph / FX Language

A registry fizikai és mágikus technique-et ugyanabban a common runtime-ban kezel. A veszélyes cast anticipation/telegraph → execution → impact/recovery ciklust kap. Az authorolt `Presentation` particle- és sound cue-ja castonként legfeljebb 64 particle; hiányakor kind-default lép életbe. A vanilla cue gameplay-authority, az optional Client FX csak enhancement. Pontosan azonos full identity/kit fail gate-et kap.

### World Boss Design

A tíz stable boss ID teljesen új, egyedi kitet kapott. Minden boss kitje tartalmaz HEALTH_THRESHOLD fázist, positioning problémát és bestiary counterplayt. A Warden carrier különösen alacsony template HP-multiplierrel normalizálódik, így a vanilla Warden alapstat és a rank/encounter scaling nem robban össze. Phase graph vagy új encounter DSL nem készült.

### Event Enemy Design

Invasion, Prologue, Cultist, Corruption, Wild Hunt, Escort és dungeon producer `AuthoredCreatureSpawnService.Request.template` kérést ad le. Az invasion hullámok determinisztikusan váltanak frontline/ranged/control szerepeket; nincs raw compatible EntityType casino. Summoned add egyszerűbb marad a bossnál, de stable template identityt visel.

### Future Boundaries

Nem része ennek a rendszernek: persistent world progression, local pressure/heat/ecology, custom model vagy texture pack, új weapon/off-hand tartalom és economy rewrite. Ezek csak külön, a gameplay staging elfogadása utáni scope-ok lehetnek.

## A leltárból korábban kimaradt komponensek

A következő táblázat a forrásfájl és a nyilvántartott felelősségi csoport pontos
összerendelése. A típus/besorolás technikai forrásleltár, nem új játékosfunkció
vagy szerveres teszteredmény. A működési szerződést az adott kaszt-, PvE-, tárgy-
és perzisztencia-fejezetek, illetve a hivatkozott forrás rögzíti.

| Komponens | Típus | Felelősségi csoport | Forrás |
| --- | --- | --- | --- |
| `ArcherCombatState` | `COMPONENT` | `` | [ArcherCombatState.java](../src/main/java/hu/taliann/icesmp/archer/ArcherCombatState.java) |
| `ArcherGameplayService` | `SERVICE` | `feature.archer-gameplay` | [ArcherGameplayService.java](../src/main/java/hu/taliann/icesmp/archer/ArcherGameplayService.java) |
| `ArcherShotLedger` | `COMPONENT` | `` | [ArcherShotLedger.java](../src/main/java/hu/taliann/icesmp/archer/ArcherShotLedger.java) |
| `AssassinCombatState` | `COMPONENT` | `` | [AssassinCombatState.java](../src/main/java/hu/taliann/icesmp/assassin/AssassinCombatState.java) |
| `AssassinGameplayService` | `SERVICE` | `feature.assassin-gameplay` | [AssassinGameplayService.java](../src/main/java/hu/taliann/icesmp/assassin/AssassinGameplayService.java) |
| `ClassGameplaySignal` | `COMPONENT` | `` | [ClassGameplaySignal.java](../src/main/java/hu/taliann/icesmp/classrelic/ClassGameplaySignal.java) |
| `ClassRelicResonanceContext` | `COMPONENT` | `` | [ClassRelicResonanceContext.java](../src/main/java/hu/taliann/icesmp/classrelic/ClassRelicResonanceContext.java) |
| `PossessionSnapshot` | `COMPONENT` | `` | [PossessionSnapshot.java](../src/main/java/hu/taliann/icesmp/classrelic/PossessionSnapshot.java) |
| `GameplayV2ClassPolicy` | `COMPONENT` | `` | [GameplayV2ClassPolicy.java](../src/main/java/hu/taliann/icesmp/classspec/application/GameplayV2ClassPolicy.java) |
| `TargetRegistry` | `COMPONENT` | `` | [TargetRegistry.java](../src/main/java/hu/taliann/icesmp/classspec/application/TargetRegistry.java) |
| `ClassHudMechanics` | `COMPONENT` | `` | [ClassHudMechanics.java](../src/main/java/hu/taliann/icesmp/classspec/integration/ClassHudMechanics.java) |
| `ClassHudMetric` | `COMPONENT` | `` | [ClassHudMetric.java](../src/main/java/hu/taliann/icesmp/classspec/integration/ClassHudMetric.java) |
| `ClassHudSlot` | `COMPONENT` | `` | [ClassHudSlot.java](../src/main/java/hu/taliann/icesmp/classspec/integration/ClassHudSlot.java) |
| `ClassHudState` | `COMPONENT` | `` | [ClassHudState.java](../src/main/java/hu/taliann/icesmp/classspec/integration/ClassHudState.java) |
| `ClassHudStateAdapter` | `COMPONENT` | `` | [ClassHudStateAdapter.java](../src/main/java/hu/taliann/icesmp/classspec/integration/ClassHudStateAdapter.java) |
| `ClientCapability` | `COMPONENT` | `` | [ClientCapability.java](../src/main/java/hu/taliann/icesmp/client/ClientCapability.java) |
| `ClientHandshake` | `COMPONENT` | `` | [ClientHandshake.java](../src/main/java/hu/taliann/icesmp/client/ClientHandshake.java) |
| `ClientRateLimiter` | `COMPONENT` | `` | [ClientRateLimiter.java](../src/main/java/hu/taliann/icesmp/client/ClientRateLimiter.java) |
| `ClientSession` | `COMPONENT` | `` | [ClientSession.java](../src/main/java/hu/taliann/icesmp/client/ClientSession.java) |
| `ClientSessionRegistry` | `COMPONENT` | `` | [ClientSessionRegistry.java](../src/main/java/hu/taliann/icesmp/client/ClientSessionRegistry.java) |
| `IceSmpClientBridge` | `INTEGRATION` | `feature.ice-smp-client` | [IceSmpClientBridge.java](../src/main/java/hu/taliann/icesmp/client/IceSmpClientBridge.java) |
| `ClientFactionProjector` | `COMPONENT` | `` | [ClientFactionProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientFactionProjector.java) |
| `ClientHudProjector` | `COMPONENT` | `` | [ClientHudProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientHudProjector.java) |
| `ClientPartyProjector` | `COMPONENT` | `` | [ClientPartyProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientPartyProjector.java) |
| `ClientProfessionProjector` | `COMPONENT` | `` | [ClientProfessionProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientProfessionProjector.java) |
| `ClientProfileProjector` | `COMPONENT` | `` | [ClientProfileProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientProfileProjector.java) |
| `ClientQuestProjector` | `COMPONENT` | `` | [ClientQuestProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientQuestProjector.java) |
| `ClientRecipeProjector` | `COMPONENT` | `` | [ClientRecipeProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientRecipeProjector.java) |
| `ClientRelicProjector` | `COMPONENT` | `` | [ClientRelicProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientRelicProjector.java) |
| `ClientTalentProjector` | `COMPONENT` | `` | [ClientTalentProjector.java](../src/main/java/hu/taliann/icesmp/client/projection/ClientTalentProjector.java) |
| `AbilityKitPayload` | `COMPONENT` | `` | [AbilityKitPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/AbilityKitPayload.java) |
| `ActionResultPayload` | `COMPONENT` | `` | [ActionResultPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/ActionResultPayload.java) |
| `BossStatePayload` | `COMPONENT` | `` | [BossStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/BossStatePayload.java) |
| `BrowseRecipesPayload` | `COMPONENT` | `` | [BrowseRecipesPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/BrowseRecipesPayload.java) |
| `CastSlotPayload` | `COMPONENT` | `` | [CastSlotPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/CastSlotPayload.java) |
| `ClientHello` | `COMPONENT` | `` | [ClientHello.java](../src/main/java/hu/taliann/icesmp/client/protocol/ClientHello.java) |
| `ClientMessageCodec` | `COMPONENT` | `` | [ClientMessageCodec.java](../src/main/java/hu/taliann/icesmp/client/protocol/ClientMessageCodec.java) |
| `ClientProtocol` | `COMPONENT` | `` | [ClientProtocol.java](../src/main/java/hu/taliann/icesmp/client/protocol/ClientProtocol.java) |
| `ClientProtocolException` | `COMPONENT` | `` | [ClientProtocolException.java](../src/main/java/hu/taliann/icesmp/client/protocol/ClientProtocolException.java) |
| `FactionStatePayload` | `COMPONENT` | `` | [FactionStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/FactionStatePayload.java) |
| `FxEventPayload` | `COMPONENT` | `` | [FxEventPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/FxEventPayload.java) |
| `HudStatePayload` | `COMPONENT` | `` | [HudStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/HudStatePayload.java) |
| `MessageEnvelope` | `COMPONENT` | `` | [MessageEnvelope.java](../src/main/java/hu/taliann/icesmp/client/protocol/MessageEnvelope.java) |
| `PartyStatePayload` | `COMPONENT` | `` | [PartyStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/PartyStatePayload.java) |
| `ProfessionActionPayload` | `COMPONENT` | `` | [ProfessionActionPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/ProfessionActionPayload.java) |
| `ProfessionStatePayload` | `COMPONENT` | `` | [ProfessionStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/ProfessionStatePayload.java) |
| `ProfileStatePayload` | `COMPONENT` | `` | [ProfileStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/ProfileStatePayload.java) |
| `ProtocolReject` | `COMPONENT` | `` | [ProtocolReject.java](../src/main/java/hu/taliann/icesmp/client/protocol/ProtocolReject.java) |
| `QuestStatePayload` | `COMPONENT` | `` | [QuestStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/QuestStatePayload.java) |
| `QuestTrackPayload` | `COMPONENT` | `` | [QuestTrackPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/QuestTrackPayload.java) |
| `RecipePagePayload` | `COMPONENT` | `` | [RecipePagePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/RecipePagePayload.java) |
| `RelicAttachmentPayload` | `COMPONENT` | `` | [RelicAttachmentPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/RelicAttachmentPayload.java) |
| `RelicStatePayload` | `COMPONENT` | `` | [RelicStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/RelicStatePayload.java) |
| `ServerHello` | `COMPONENT` | `` | [ServerHello.java](../src/main/java/hu/taliann/icesmp/client/protocol/ServerHello.java) |
| `SpellActionPayload` | `COMPONENT` | `` | [SpellActionPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/SpellActionPayload.java) |
| `SpellbookStatePayload` | `COMPONENT` | `` | [SpellbookStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/SpellbookStatePayload.java) |
| `TalentActionPayload` | `COMPONENT` | `` | [TalentActionPayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/TalentActionPayload.java) |
| `TalentStatePayload` | `COMPONENT` | `` | [TalentStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/TalentStatePayload.java) |
| `TerritoryStatePayload` | `COMPONENT` | `` | [TerritoryStatePayload.java](../src/main/java/hu/taliann/icesmp/client/protocol/TerritoryStatePayload.java) |
| `PrologueCommand` | `COMMAND` | `feature.prologue` | [PrologueCommand.java](../src/main/java/hu/taliann/icesmp/commands/PrologueCommand.java) |
| `FactionStatusSubcommand` | `COMPONENT` | `` | [FactionStatusSubcommand.java](../src/main/java/hu/taliann/icesmp/commands/faction/FactionStatusSubcommand.java) |
| `DeathKnightCombatState` | `COMPONENT` | `` | [DeathKnightCombatState.java](../src/main/java/hu/taliann/icesmp/deathknight/DeathKnightCombatState.java) |
| `DeathKnightGameplayService` | `SERVICE` | `feature.death-knight-gameplay` | [DeathKnightGameplayService.java](../src/main/java/hu/taliann/icesmp/deathknight/DeathKnightGameplayService.java) |
| `DemonHunterCombatState` | `COMPONENT` | `` | [DemonHunterCombatState.java](../src/main/java/hu/taliann/icesmp/demonhunter/DemonHunterCombatState.java) |
| `DemonHunterGameplayService` | `SERVICE` | `feature.demon-hunter-gameplay` | [DemonHunterGameplayService.java](../src/main/java/hu/taliann/icesmp/demonhunter/DemonHunterGameplayService.java) |
| `DruidCombatState` | `COMPONENT` | `` | [DruidCombatState.java](../src/main/java/hu/taliann/icesmp/druid/DruidCombatState.java) |
| `DruidGameplayService` | `SERVICE` | `feature.druid-gameplay` | [DruidGameplayService.java](../src/main/java/hu/taliann/icesmp/druid/DruidGameplayService.java) |
| `EvokerCombatState` | `COMPONENT` | `` | [EvokerCombatState.java](../src/main/java/hu/taliann/icesmp/evoker/EvokerCombatState.java) |
| `EvokerGameplayService` | `SERVICE` | `feature.evoker-gameplay` | [EvokerGameplayService.java](../src/main/java/hu/taliann/icesmp/evoker/EvokerGameplayService.java) |
| `WhisperSightline` | `COMPONENT` | `` | [WhisperSightline.java](../src/main/java/hu/taliann/icesmp/factions/WhisperSightline.java) |
| `ClassGameplayConfigMenuGUI` | `GUI` | `feature.class-gameplay-config-menu` | [ClassGameplayConfigMenuGUI.java](../src/main/java/hu/taliann/icesmp/gui/ClassGameplayConfigMenuGUI.java) |
| `ConfigStagedBatchValidator` | `GUI_COMPONENT` | `feature.config-staged-batch-validator` | [ConfigStagedBatchValidator.java](../src/main/java/hu/taliann/icesmp/gui/ConfigStagedBatchValidator.java) |
| `ItemForgeGUI` | `GUI` | `feature.item-forge` | [ItemForgeGUI.java](../src/main/java/hu/taliann/icesmp/gui/ItemForgeGUI.java) |
| `ItemForgeHolder` | `GUI_HOLDER` | `feature.item-forge` | [ItemForgeHolder.java](../src/main/java/hu/taliann/icesmp/gui/ItemForgeHolder.java) |
| `ClassXpProgress` | `COMPONENT` | `` | [ClassXpProgress.java](../src/main/java/hu/taliann/icesmp/hud/ClassXpProgress.java) |
| `HudComponent` | `COMPONENT` | `` | [HudComponent.java](../src/main/java/hu/taliann/icesmp/hud/HudComponent.java) |
| `HudComponentLayout` | `COMPONENT` | `` | [HudComponentLayout.java](../src/main/java/hu/taliann/icesmp/hud/HudComponentLayout.java) |
| `HudEditorAccessPolicy` | `COMPONENT` | `` | [HudEditorAccessPolicy.java](../src/main/java/hu/taliann/icesmp/hud/HudEditorAccessPolicy.java) |
| `HudEditorStateMachine` | `COMPONENT` | `` | [HudEditorStateMachine.java](../src/main/java/hu/taliann/icesmp/hud/HudEditorStateMachine.java) |
| `HudLayoutPreset` | `COMPONENT` | `` | [HudLayoutPreset.java](../src/main/java/hu/taliann/icesmp/hud/HudLayoutPreset.java) |
| `HudLayoutSnapshot` | `COMPONENT` | `` | [HudLayoutSnapshot.java](../src/main/java/hu/taliann/icesmp/hud/HudLayoutSnapshot.java) |
| `HudPreviewCatalog` | `COMPONENT` | `` | [HudPreviewCatalog.java](../src/main/java/hu/taliann/icesmp/hud/HudPreviewCatalog.java) |
| `HudPreviewSelection` | `COMPONENT` | `` | [HudPreviewSelection.java](../src/main/java/hu/taliann/icesmp/hud/HudPreviewSelection.java) |
| `IceSmpHudBackend` | `COMPONENT` | `` | [IceSmpHudBackend.java](../src/main/java/hu/taliann/icesmp/hud/IceSmpHudBackend.java) |
| `IceSmpHudModel` | `COMPONENT` | `` | [IceSmpHudModel.java](../src/main/java/hu/taliann/icesmp/hud/IceSmpHudModel.java) |
| `IceSmpHudRenderer` | `COMPONENT` | `` | [IceSmpHudRenderer.java](../src/main/java/hu/taliann/icesmp/hud/IceSmpHudRenderer.java) |
| `PartyHudRenderer` | `COMPONENT` | `` | [PartyHudRenderer.java](../src/main/java/hu/taliann/icesmp/hud/PartyHudRenderer.java) |
| `PartyHudState` | `COMPONENT` | `` | [PartyHudState.java](../src/main/java/hu/taliann/icesmp/hud/PartyHudState.java) |
| `PlayerHudState` | `COMPONENT` | `` | [PlayerHudState.java](../src/main/java/hu/taliann/icesmp/hud/PlayerHudState.java) |
| `SurvivalHudRenderer` | `COMPONENT` | `` | [SurvivalHudRenderer.java](../src/main/java/hu/taliann/icesmp/hud/SurvivalHudRenderer.java) |
| `SurvivalHudState` | `COMPONENT` | `` | [SurvivalHudState.java](../src/main/java/hu/taliann/icesmp/hud/SurvivalHudState.java) |
| `TargetFrameMetadataPolicy` | `COMPONENT` | `` | [TargetFrameMetadataPolicy.java](../src/main/java/hu/taliann/icesmp/hud/TargetFrameMetadataPolicy.java) |
| `TargetFrameTracker` | `COMPONENT` | `` | [TargetFrameTracker.java](../src/main/java/hu/taliann/icesmp/hud/TargetFrameTracker.java) |
| `TargetHudRenderer` | `COMPONENT` | `` | [TargetHudRenderer.java](../src/main/java/hu/taliann/icesmp/hud/TargetHudRenderer.java) |
| `TargetHudState` | `COMPONENT` | `` | [TargetHudState.java](../src/main/java/hu/taliann/icesmp/hud/TargetHudState.java) |
| `ArmorFamily` | `COMPONENT` | `` | [ArmorFamily.java](../src/main/java/hu/taliann/icesmp/itemization/ArmorFamily.java) |
| `ArmorFamilyProfile` | `COMPONENT` | `` | [ArmorFamilyProfile.java](../src/main/java/hu/taliann/icesmp/itemization/ArmorFamilyProfile.java) |
| `AtomicCursorRehome` | `COMPONENT` | `` | [AtomicCursorRehome.java](../src/main/java/hu/taliann/icesmp/itemization/AtomicCursorRehome.java) |
| `BuildAwareLootService` | `SERVICE` | `feature.build-aware-loot` | [BuildAwareLootService.java](../src/main/java/hu/taliann/icesmp/itemization/BuildAwareLootService.java) |
| `CanonicalPhysicalState` | `COMPONENT` | `` | [CanonicalPhysicalState.java](../src/main/java/hu/taliann/icesmp/itemization/CanonicalPhysicalState.java) |
| `EquipmentBudgetModel` | `COMPONENT` | `` | [EquipmentBudgetModel.java](../src/main/java/hu/taliann/icesmp/itemization/EquipmentBudgetModel.java) |
| `EquipmentCatalogValidator` | `COMPONENT` | `` | [EquipmentCatalogValidator.java](../src/main/java/hu/taliann/icesmp/itemization/EquipmentCatalogValidator.java) |
| `EquipmentProficiencyPolicy` | `COMPONENT` | `` | [EquipmentProficiencyPolicy.java](../src/main/java/hu/taliann/icesmp/itemization/EquipmentProficiencyPolicy.java) |
| `EquipmentProficiencyService` | `SERVICE` | `feature.equipment-proficiency` | [EquipmentProficiencyService.java](../src/main/java/hu/taliann/icesmp/itemization/EquipmentProficiencyService.java) |
| `EquipmentRehomeTransaction` | `COMPONENT` | `` | [EquipmentRehomeTransaction.java](../src/main/java/hu/taliann/icesmp/itemization/EquipmentRehomeTransaction.java) |
| `ItemHistoryEvent` | `COMPONENT` | `` | [ItemHistoryEvent.java](../src/main/java/hu/taliann/icesmp/itemization/ItemHistoryEvent.java) |
| `ItemIdentityService` | `SERVICE` | `feature.item-identity` | [ItemIdentityService.java](../src/main/java/hu/taliann/icesmp/itemization/ItemIdentityService.java) |
| `ItemInstance` | `COMPONENT` | `` | [ItemInstance.java](../src/main/java/hu/taliann/icesmp/itemization/ItemInstance.java) |
| `ItemInstanceCodec` | `COMPONENT` | `` | [ItemInstanceCodec.java](../src/main/java/hu/taliann/icesmp/itemization/ItemInstanceCodec.java) |
| `ItemMutationCoordinator` | `COMPONENT` | `` | [ItemMutationCoordinator.java](../src/main/java/hu/taliann/icesmp/itemization/ItemMutationCoordinator.java) |
| `ItemMutationFaultMatrix` | `COMPONENT` | `` | [ItemMutationFaultMatrix.java](../src/main/java/hu/taliann/icesmp/itemization/ItemMutationFaultMatrix.java) |
| `ItemMutationRecoveryPolicy` | `COMPONENT` | `` | [ItemMutationRecoveryPolicy.java](../src/main/java/hu/taliann/icesmp/itemization/ItemMutationRecoveryPolicy.java) |
| `ItemMutationService` | `SERVICE` | `feature.item-mutation` | [ItemMutationService.java](../src/main/java/hu/taliann/icesmp/itemization/ItemMutationService.java) |
| `ItemRarity` | `COMPONENT` | `` | [ItemRarity.java](../src/main/java/hu/taliann/icesmp/itemization/ItemRarity.java) |
| `ItemSalvageService` | `SERVICE` | `feature.item-salvage` | [ItemSalvageService.java](../src/main/java/hu/taliann/icesmp/itemization/ItemSalvageService.java) |
| `ItemSetDefinition` | `COMPONENT` | `` | [ItemSetDefinition.java](../src/main/java/hu/taliann/icesmp/itemization/ItemSetDefinition.java) |
| `ItemStatCatalog` | `COMPONENT` | `` | [ItemStatCatalog.java](../src/main/java/hu/taliann/icesmp/itemization/ItemStatCatalog.java) |
| `ItemStatScaling` | `COMPONENT` | `` | [ItemStatScaling.java](../src/main/java/hu/taliann/icesmp/itemization/ItemStatScaling.java) |
| `ItemState` | `COMPONENT` | `` | [ItemState.java](../src/main/java/hu/taliann/icesmp/itemization/ItemState.java) |
| `ItemTemplate` | `COMPONENT` | `` | [ItemTemplate.java](../src/main/java/hu/taliann/icesmp/itemization/ItemTemplate.java) |
| `ItemTemplateCatalogIndex` | `COMPONENT` | `` | [ItemTemplateCatalogIndex.java](../src/main/java/hu/taliann/icesmp/itemization/ItemTemplateCatalogIndex.java) |
| `ItemTemplateRegistry` | `COMPONENT` | `` | [ItemTemplateRegistry.java](../src/main/java/hu/taliann/icesmp/itemization/ItemTemplateRegistry.java) |
| `ItemTransformationPolicy` | `COMPONENT` | `` | [ItemTransformationPolicy.java](../src/main/java/hu/taliann/icesmp/itemization/ItemTransformationPolicy.java) |
| `LootDiversityState` | `COMPONENT` | `` | [LootDiversityState.java](../src/main/java/hu/taliann/icesmp/itemization/LootDiversityState.java) |
| `PaperSourceIntegrityRuntimeProbe` | `COMPONENT` | `` | [PaperSourceIntegrityRuntimeProbe.java](../src/main/java/hu/taliann/icesmp/itemization/PaperSourceIntegrityRuntimeProbe.java) |
| `RuneMutationPolicy` | `COMPONENT` | `` | [RuneMutationPolicy.java](../src/main/java/hu/taliann/icesmp/itemization/RuneMutationPolicy.java) |
| `SignatureEffectRegistry` | `COMPONENT` | `` | [SignatureEffectRegistry.java](../src/main/java/hu/taliann/icesmp/itemization/SignatureEffectRegistry.java) |
| `RarityPresentationService` | `SERVICE` | `feature.rarity-presentation` | [RarityPresentationService.java](../src/main/java/hu/taliann/icesmp/items/RarityPresentationService.java) |
| `WearablePresentation` | `ITEM` | `feature.wearable-presentation` | [WearablePresentation.java](../src/main/java/hu/taliann/icesmp/items/WearablePresentation.java) |
| `EquipmentProficiencyListener` | `LISTENER` | `feature.equipment-proficiency` | [EquipmentProficiencyListener.java](../src/main/java/hu/taliann/icesmp/listeners/EquipmentProficiencyListener.java) |
| `RareGatheringListener` | `LISTENER` | `feature.rare-gathering` | [RareGatheringListener.java](../src/main/java/hu/taliann/icesmp/listeners/RareGatheringListener.java) |
| `VanillaCraftingBoundaryListener` | `LISTENER` | `feature.vanilla-crafting-boundary` | [VanillaCraftingBoundaryListener.java](../src/main/java/hu/taliann/icesmp/listeners/VanillaCraftingBoundaryListener.java) |
| `ClientFxRoute` | `COMPONENT` | `` | [ClientFxRoute.java](../src/main/java/hu/taliann/icesmp/managers/ClientFxRoute.java) |
| `DonationTransferLifecycle` | `COMPONENT` | `` | [DonationTransferLifecycle.java](../src/main/java/hu/taliann/icesmp/managers/DonationTransferLifecycle.java) |
| `QuestPhysicalRewardDeliveryService` | `SERVICE` | `feature.quest-physical-reward-delivery` | [QuestPhysicalRewardDeliveryService.java](../src/main/java/hu/taliann/icesmp/managers/QuestPhysicalRewardDeliveryService.java) |
| `MonkCombatState` | `COMPONENT` | `` | [MonkCombatState.java](../src/main/java/hu/taliann/icesmp/monk/MonkCombatState.java) |
| `MonkGameplayService` | `SERVICE` | `feature.monk-gameplay` | [MonkGameplayService.java](../src/main/java/hu/taliann/icesmp/monk/MonkGameplayService.java) |
| `PaladinCombatState` | `COMPONENT` | `` | [PaladinCombatState.java](../src/main/java/hu/taliann/icesmp/paladin/PaladinCombatState.java) |
| `PaladinGameplayService` | `SERVICE` | `feature.paladin-gameplay` | [PaladinGameplayService.java](../src/main/java/hu/taliann/icesmp/paladin/PaladinGameplayService.java) |
| `DeathEscrowDeliveryPlan` | `COMPONENT` | `` | [DeathEscrowDeliveryPlan.java](../src/main/java/hu/taliann/icesmp/playerprofile/application/DeathEscrowDeliveryPlan.java) |
| `EconomyReceiptLedger` | `COMPONENT` | `` | [EconomyReceiptLedger.java](../src/main/java/hu/taliann/icesmp/playerprofile/application/EconomyReceiptLedger.java) |
| `PlayerProfileLootDiversityStore` | `PERSISTENT_STORE` | `feature.player-profile-loot-diversity` | [PlayerProfileLootDiversityStore.java](../src/main/java/hu/taliann/icesmp/playerprofile/application/PlayerProfileLootDiversityStore.java) |
| `PlayerProfileSeasonParticipationStore` | `PERSISTENT_STORE` | `feature.player-profile-season-participation` | [PlayerProfileSeasonParticipationStore.java](../src/main/java/hu/taliann/icesmp/playerprofile/application/PlayerProfileSeasonParticipationStore.java) |
| `QuestRewardDeliveryProtocol` | `COMPONENT` | `` | [QuestRewardDeliveryProtocol.java](../src/main/java/hu/taliann/icesmp/playerprofile/application/QuestRewardDeliveryProtocol.java) |
| `PriestCombatState` | `COMPONENT` | `` | [PriestCombatState.java](../src/main/java/hu/taliann/icesmp/priest/PriestCombatState.java) |
| `PriestGameplayService` | `SERVICE` | `feature.priest-gameplay` | [PriestGameplayService.java](../src/main/java/hu/taliann/icesmp/priest/PriestGameplayService.java) |
| `BlueprintRecoveryPolicy` | `COMPONENT` | `` | [BlueprintRecoveryPolicy.java](../src/main/java/hu/taliann/icesmp/professions/BlueprintRecoveryPolicy.java) |
| `ProfessionCraftQualityPolicy` | `COMPONENT` | `` | [ProfessionCraftQualityPolicy.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionCraftQualityPolicy.java) |
| `ProfessionCraftTransaction` | `COMPONENT` | `` | [ProfessionCraftTransaction.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionCraftTransaction.java) |
| `ProfessionEconomyTelemetry` | `COMPONENT` | `` | [ProfessionEconomyTelemetry.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionEconomyTelemetry.java) |
| `ProfessionEffectiveCraftPlan` | `COMPONENT` | `` | [ProfessionEffectiveCraftPlan.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionEffectiveCraftPlan.java) |
| `ProfessionMaterialRegistry` | `COMPONENT` | `` | [ProfessionMaterialRegistry.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionMaterialRegistry.java) |
| `ProfessionSpecializationEconomyPolicy` | `COMPONENT` | `` | [ProfessionSpecializationEconomyPolicy.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionSpecializationEconomyPolicy.java) |
| `ProfessionsPaperRuntimeProbe` | `COMPONENT` | `` | [ProfessionsPaperRuntimeProbe.java](../src/main/java/hu/taliann/icesmp/professions/ProfessionsPaperRuntimeProbe.java) |
| `BlockRewardOriginTracker` | `COMPONENT` | `` | [BlockRewardOriginTracker.java](../src/main/java/hu/taliann/icesmp/progression/BlockRewardOriginTracker.java) |
| `ItemAcquisitionPolicy` | `COMPONENT` | `` | [ItemAcquisitionPolicy.java](../src/main/java/hu/taliann/icesmp/progression/ItemAcquisitionPolicy.java) |
| `BreachSeverity` | `COMPONENT` | `` | [BreachSeverity.java](../src/main/java/hu/taliann/icesmp/prologue/BreachSeverity.java) |
| `PrologueCeasefireListener` | `LISTENER` | `feature.prologue-ceasefire` | [PrologueCeasefireListener.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueCeasefireListener.java) |
| `PrologueContentPolicy` | `COMPONENT` | `` | [PrologueContentPolicy.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueContentPolicy.java) |
| `PrologueEncounterEngine` | `COMPONENT` | `` | [PrologueEncounterEngine.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueEncounterEngine.java) |
| `PrologueFinaleManager` | `MANAGER` | `feature.prologue-finale` | [PrologueFinaleManager.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueFinaleManager.java) |
| `PrologueFinalePhase` | `COMPONENT` | `` | [PrologueFinalePhase.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueFinalePhase.java) |
| `PrologueFinaleRunState` | `COMPONENT` | `` | [PrologueFinaleRunState.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueFinaleRunState.java) |
| `PrologueFinaleSafety` | `COMPONENT` | `` | [PrologueFinaleSafety.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueFinaleSafety.java) |
| `PrologueFinaleSettlement` | `COMPONENT` | `` | [PrologueFinaleSettlement.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueFinaleSettlement.java) |
| `PrologueHudController` | `COMPONENT` | `` | [PrologueHudController.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueHudController.java) |
| `PrologueManager` | `MANAGER` | `feature.prologue` | [PrologueManager.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueManager.java) |
| `PrologueParticipantTracker` | `COMPONENT` | `` | [PrologueParticipantTracker.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueParticipantTracker.java) |
| `ProloguePauseClock` | `COMPONENT` | `` | [ProloguePauseClock.java](../src/main/java/hu/taliann/icesmp/prologue/ProloguePauseClock.java) |
| `PrologueProgression` | `COMPONENT` | `` | [PrologueProgression.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueProgression.java) |
| `PrologueRewardService` | `SERVICE` | `feature.prologue-reward` | [PrologueRewardService.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueRewardService.java) |
| `PrologueRuntime` | `COMPONENT` | `` | [PrologueRuntime.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueRuntime.java) |
| `PrologueRuntimeConfigOverlay` | `COMPONENT` | `` | [PrologueRuntimeConfigOverlay.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueRuntimeConfigOverlay.java) |
| `PrologueScaling` | `COMPONENT` | `` | [PrologueScaling.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueScaling.java) |
| `PrologueSeasonTransition` | `COMPONENT` | `` | [PrologueSeasonTransition.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueSeasonTransition.java) |
| `PrologueStage` | `COMPONENT` | `` | [PrologueStage.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueStage.java) |
| `PrologueState` | `COMPONENT` | `` | [PrologueState.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueState.java) |
| `PrologueTimelineController` | `COMPONENT` | `` | [PrologueTimelineController.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueTimelineController.java) |
| `PrologueWorldAccess` | `COMPONENT` | `` | [PrologueWorldAccess.java](../src/main/java/hu/taliann/icesmp/prologue/PrologueWorldAccess.java) |
| `AuthoredCreatureSpawnService` | `SERVICE` | `feature.authored-creature-spawn` | [AuthoredCreatureSpawnService.java](../src/main/java/hu/taliann/icesmp/pve/AuthoredCreatureSpawnService.java) |
| `AuthoredPveContentValidator` | `COMPONENT` | `` | [AuthoredPveContentValidator.java](../src/main/java/hu/taliann/icesmp/pve/AuthoredPveContentValidator.java) |
| `CombatPowerEstimator` | `COMPONENT` | `` | [CombatPowerEstimator.java](../src/main/java/hu/taliann/icesmp/pve/CombatPowerEstimator.java) |
| `CombatTelemetry` | `COMPONENT` | `` | [CombatTelemetry.java](../src/main/java/hu/taliann/icesmp/pve/CombatTelemetry.java) |
| `ContextualWeightedSelector` | `COMPONENT` | `` | [ContextualWeightedSelector.java](../src/main/java/hu/taliann/icesmp/pve/ContextualWeightedSelector.java) |
| `ContributionLedger` | `COMPONENT` | `` | [ContributionLedger.java](../src/main/java/hu/taliann/icesmp/pve/ContributionLedger.java) |
| `CreatureProfileService` | `SERVICE` | `feature.creature-profile` | [CreatureProfileService.java](../src/main/java/hu/taliann/icesmp/pve/CreatureProfileService.java) |
| `CreatureSpeciesPolicy` | `COMPONENT` | `` | [CreatureSpeciesPolicy.java](../src/main/java/hu/taliann/icesmp/pve/CreatureSpeciesPolicy.java) |
| `CreatureSpeciesRegistry` | `COMPONENT` | `` | [CreatureSpeciesRegistry.java](../src/main/java/hu/taliann/icesmp/pve/CreatureSpeciesRegistry.java) |
| `DaylightProtectionPolicy` | `COMPONENT` | `` | [DaylightProtectionPolicy.java](../src/main/java/hu/taliann/icesmp/pve/DaylightProtectionPolicy.java) |
| `EliteAffix` | `COMPONENT` | `` | [EliteAffix.java](../src/main/java/hu/taliann/icesmp/pve/EliteAffix.java) |
| `EncounterRewardDeliveryService` | `SERVICE` | `feature.encounter-reward-delivery` | [EncounterRewardDeliveryService.java](../src/main/java/hu/taliann/icesmp/pve/EncounterRewardDeliveryService.java) |
| `EncounterRewardRecoveryPolicy` | `COMPONENT` | `` | [EncounterRewardRecoveryPolicy.java](../src/main/java/hu/taliann/icesmp/pve/EncounterRewardRecoveryPolicy.java) |
| `EncounterScalingPolicy` | `COMPONENT` | `` | [EncounterScalingPolicy.java](../src/main/java/hu/taliann/icesmp/pve/EncounterScalingPolicy.java) |
| `EquippedCombatPowerModel` | `COMPONENT` | `` | [EquippedCombatPowerModel.java](../src/main/java/hu/taliann/icesmp/pve/EquippedCombatPowerModel.java) |
| `EquippedCombatPowerService` | `SERVICE` | `feature.equipped-combat-power` | [EquippedCombatPowerService.java](../src/main/java/hu/taliann/icesmp/pve/EquippedCombatPowerService.java) |
| `MobAbilityDefinition` | `COMPONENT` | `` | [MobAbilityDefinition.java](../src/main/java/hu/taliann/icesmp/pve/MobAbilityDefinition.java) |
| `MobAbilityRegistry` | `COMPONENT` | `` | [MobAbilityRegistry.java](../src/main/java/hu/taliann/icesmp/pve/MobAbilityRegistry.java) |
| `MobAbilityRuntime` | `COMPONENT` | `` | [MobAbilityRuntime.java](../src/main/java/hu/taliann/icesmp/pve/MobAbilityRuntime.java) |
| `MobArchetype` | `COMPONENT` | `` | [MobArchetype.java](../src/main/java/hu/taliann/icesmp/pve/MobArchetype.java) |
| `MobBehaviorProfile` | `COMPONENT` | `` | [MobBehaviorProfile.java](../src/main/java/hu/taliann/icesmp/pve/MobBehaviorProfile.java) |
| `MobNaturalContext` | `COMPONENT` | `` | [MobNaturalContext.java](../src/main/java/hu/taliann/icesmp/pve/MobNaturalContext.java) |
| `MobProgressionPolicy` | `COMPONENT` | `` | [MobProgressionPolicy.java](../src/main/java/hu/taliann/icesmp/pve/MobProgressionPolicy.java) |
| `MobRank` | `COMPONENT` | `` | [MobRank.java](../src/main/java/hu/taliann/icesmp/pve/MobRank.java) |
| `MobRankLootPolicy` | `COMPONENT` | `` | [MobRankLootPolicy.java](../src/main/java/hu/taliann/icesmp/pve/MobRankLootPolicy.java) |
| `MobTechniqueAction` | `COMPONENT` | `` | [MobTechniqueAction.java](../src/main/java/hu/taliann/icesmp/pve/MobTechniqueAction.java) |
| `MobTechniqueCondition` | `COMPONENT` | `` | [MobTechniqueCondition.java](../src/main/java/hu/taliann/icesmp/pve/MobTechniqueCondition.java) |
| `MobTemplate` | `COMPONENT` | `` | [MobTemplate.java](../src/main/java/hu/taliann/icesmp/pve/MobTemplate.java) |
| `MobTemplateRegistry` | `COMPONENT` | `` | [MobTemplateRegistry.java](../src/main/java/hu/taliann/icesmp/pve/MobTemplateRegistry.java) |
| `OnboardingWelcomeCopy` | `COMPONENT` | `` | [OnboardingWelcomeCopy.java](../src/main/java/hu/taliann/icesmp/quest/OnboardingWelcomeCopy.java) |
| `QuestCategory` | `COMPONENT` | `` | [QuestCategory.java](../src/main/java/hu/taliann/icesmp/quest/QuestCategory.java) |
| `QuestChoiceRegistry` | `COMPONENT` | `` | [QuestChoiceRegistry.java](../src/main/java/hu/taliann/icesmp/quest/QuestChoiceRegistry.java) |
| `QuestCurrencyResolver` | `COMPONENT` | `` | [QuestCurrencyResolver.java](../src/main/java/hu/taliann/icesmp/quest/QuestCurrencyResolver.java) |
| `QuestGraphValidator` | `COMPONENT` | `` | [QuestGraphValidator.java](../src/main/java/hu/taliann/icesmp/quest/QuestGraphValidator.java) |
| `QuestItemContentIntegrityPaperRuntimeProbe` | `COMPONENT` | `` | [QuestItemContentIntegrityPaperRuntimeProbe.java](../src/main/java/hu/taliann/icesmp/quest/QuestItemContentIntegrityPaperRuntimeProbe.java) |
| `QuestMarkerPalette` | `COMPONENT` | `` | [QuestMarkerPalette.java](../src/main/java/hu/taliann/icesmp/quest/QuestMarkerPalette.java) |
| `QuestSourceContext` | `COMPONENT` | `` | [QuestSourceContext.java](../src/main/java/hu/taliann/icesmp/quest/QuestSourceContext.java) |
| `QuestSourcePolicy` | `COMPONENT` | `` | [QuestSourcePolicy.java](../src/main/java/hu/taliann/icesmp/quest/QuestSourcePolicy.java) |
| `QuestVisibility` | `COMPONENT` | `` | [QuestVisibility.java](../src/main/java/hu/taliann/icesmp/quest/QuestVisibility.java) |
| `RelicTransferExpectation` | `COMPONENT` | `` | [RelicTransferExpectation.java](../src/main/java/hu/taliann/icesmp/relics/RelicTransferExpectation.java) |
| `RelicWorldStateSnapshot` | `COMPONENT` | `` | [RelicWorldStateSnapshot.java](../src/main/java/hu/taliann/icesmp/relics/RelicWorldStateSnapshot.java) |
| `RelicWorldStateStore` | `PERSISTENT_STORE` | `feature.relic-world-state` | [RelicWorldStateStore.java](../src/main/java/hu/taliann/icesmp/relics/RelicWorldStateStore.java) |
| `HiddenDevAuthority` | `COMPONENT` | `` | [HiddenDevAuthority.java](../src/main/java/hu/taliann/icesmp/security/HiddenDevAuthority.java) |
| `ShamanCombatState` | `COMPONENT` | `` | [ShamanCombatState.java](../src/main/java/hu/taliann/icesmp/shaman/ShamanCombatState.java) |
| `ShamanGameplayService` | `SERVICE` | `feature.shaman-gameplay` | [ShamanGameplayService.java](../src/main/java/hu/taliann/icesmp/shaman/ShamanGameplayService.java) |
| `CastModifiers` | `SPELL_COMPONENT` | `feature.cast-modifiers` | [CastModifiers.java](../src/main/java/hu/taliann/icesmp/spells/CastModifiers.java) |
| `CastOutcome` | `SPELL_COMPONENT` | `feature.cast-outcome` | [CastOutcome.java](../src/main/java/hu/taliann/icesmp/spells/CastOutcome.java) |
| `DurableCompanionCallSpell` | `SPELL` | `feature.durable-companion-call` | [DurableCompanionCallSpell.java](../src/main/java/hu/taliann/icesmp/spells/DurableCompanionCallSpell.java) |
| `SpellExecutionContext` | `SPELL_COMPONENT` | `feature.spell-execution-context` | [SpellExecutionContext.java](../src/main/java/hu/taliann/icesmp/spells/SpellExecutionContext.java) |
| `ItemMutationJournal` | `PERSISTENT_STORE` | `feature.item-mutation-journal` | [ItemMutationJournal.java](../src/main/java/hu/taliann/icesmp/storage/ItemMutationJournal.java) |
| `ArchaeologyTooltipBridge` | `INTEGRATION` | `feature.archaeology-tooltip` | [ArchaeologyTooltipBridge.java](../src/main/java/hu/taliann/icesmp/trash/ArchaeologyTooltipBridge.java) |
| `HiddenDiscipline` | `COMPONENT` | `` | [HiddenDiscipline.java](../src/main/java/hu/taliann/icesmp/trash/HiddenDiscipline.java) |
| `TooltipPacketBridge_1_21_11` | `COMPONENT` | `` | [TooltipPacketBridge_1_21_11.java](../src/main/java/hu/taliann/icesmp/trash/TooltipPacketBridge_1_21_11.java) |
| `TossableObjectRuntime` | `COMPONENT` | `` | [TossableObjectRuntime.java](../src/main/java/hu/taliann/icesmp/trash/TossableObjectRuntime.java) |
| `TrashAmbientManager` | `MANAGER` | `feature.trash-ambient` | [TrashAmbientManager.java](../src/main/java/hu/taliann/icesmp/trash/TrashAmbientManager.java) |
| `TrashAnomalyBehavior` | `COMPONENT` | `` | [TrashAnomalyBehavior.java](../src/main/java/hu/taliann/icesmp/trash/TrashAnomalyBehavior.java) |
| `TrashAnomalyPolicy` | `COMPONENT` | `` | [TrashAnomalyPolicy.java](../src/main/java/hu/taliann/icesmp/trash/TrashAnomalyPolicy.java) |
| `TrashAnomalyRuntime` | `COMPONENT` | `` | [TrashAnomalyRuntime.java](../src/main/java/hu/taliann/icesmp/trash/TrashAnomalyRuntime.java) |
| `TrashAnomalyStateStore` | `PERSISTENT_STORE` | `feature.trash-anomaly-state` | [TrashAnomalyStateStore.java](../src/main/java/hu/taliann/icesmp/trash/TrashAnomalyStateStore.java) |
| `TrashArchaeologyFactEngine` | `COMPONENT` | `` | [TrashArchaeologyFactEngine.java](../src/main/java/hu/taliann/icesmp/trash/TrashArchaeologyFactEngine.java) |
| `TrashArchaeologyListener` | `LISTENER` | `feature.trash-archaeology` | [TrashArchaeologyListener.java](../src/main/java/hu/taliann/icesmp/trash/TrashArchaeologyListener.java) |
| `TrashArchaeologyProfileStore` | `PERSISTENT_STORE` | `feature.trash-archaeology-profile` | [TrashArchaeologyProfileStore.java](../src/main/java/hu/taliann/icesmp/trash/TrashArchaeologyProfileStore.java) |
| `TrashArchaeologyService` | `SERVICE` | `feature.trash-archaeology` | [TrashArchaeologyService.java](../src/main/java/hu/taliann/icesmp/trash/TrashArchaeologyService.java) |
| `TrashCatalog` | `COMPONENT` | `` | [TrashCatalog.java](../src/main/java/hu/taliann/icesmp/trash/TrashCatalog.java) |
| `TrashContext` | `COMPONENT` | `` | [TrashContext.java](../src/main/java/hu/taliann/icesmp/trash/TrashContext.java) |
| `TrashContextResolver` | `COMPONENT` | `` | [TrashContextResolver.java](../src/main/java/hu/taliann/icesmp/trash/TrashContextResolver.java) |
| `TrashDefinition` | `COMPONENT` | `` | [TrashDefinition.java](../src/main/java/hu/taliann/icesmp/trash/TrashDefinition.java) |
| `TrashDevCommand` | `COMMAND` | `feature.trash-dev` | [TrashDevCommand.java](../src/main/java/hu/taliann/icesmp/trash/TrashDevCommand.java) |
| `TrashFishingListener` | `LISTENER` | `feature.trash-fishing` | [TrashFishingListener.java](../src/main/java/hu/taliann/icesmp/trash/TrashFishingListener.java) |
| `TrashHistoryEvent` | `COMPONENT` | `` | [TrashHistoryEvent.java](../src/main/java/hu/taliann/icesmp/trash/TrashHistoryEvent.java) |
| `TrashHistoryJournal` | `COMPONENT` | `` | [TrashHistoryJournal.java](../src/main/java/hu/taliann/icesmp/trash/TrashHistoryJournal.java) |
| `TrashHistoryListener` | `LISTENER` | `feature.trash-history` | [TrashHistoryListener.java](../src/main/java/hu/taliann/icesmp/trash/TrashHistoryListener.java) |
| `TrashHistoryService` | `SERVICE` | `feature.trash-history` | [TrashHistoryService.java](../src/main/java/hu/taliann/icesmp/trash/TrashHistoryService.java) |
| `TrashHistoryStore` | `PERSISTENT_STORE` | `feature.trash-history` | [TrashHistoryStore.java](../src/main/java/hu/taliann/icesmp/trash/TrashHistoryStore.java) |
| `TrashItemFactory` | `ITEM_FACTORY` | `feature.trash-item` | [TrashItemFactory.java](../src/main/java/hu/taliann/icesmp/trash/TrashItemFactory.java) |
| `TrashKind` | `COMPONENT` | `` | [TrashKind.java](../src/main/java/hu/taliann/icesmp/trash/TrashKind.java) |
| `TrashLifecyclePhase` | `COMPONENT` | `` | [TrashLifecyclePhase.java](../src/main/java/hu/taliann/icesmp/trash/TrashLifecyclePhase.java) |
| `TrashLootSelector` | `COMPONENT` | `` | [TrashLootSelector.java](../src/main/java/hu/taliann/icesmp/trash/TrashLootSelector.java) |
| `TrashLootService` | `SERVICE` | `feature.trash-loot` | [TrashLootService.java](../src/main/java/hu/taliann/icesmp/trash/TrashLootService.java) |
| `TrashLootSource` | `COMPONENT` | `` | [TrashLootSource.java](../src/main/java/hu/taliann/icesmp/trash/TrashLootSource.java) |
| `TrashLootTuning` | `COMPONENT` | `` | [TrashLootTuning.java](../src/main/java/hu/taliann/icesmp/trash/TrashLootTuning.java) |
| `TrashMobDropListener` | `LISTENER` | `feature.trash-mob-drop` | [TrashMobDropListener.java](../src/main/java/hu/taliann/icesmp/trash/TrashMobDropListener.java) |
| `TrashProductionRuntimeProbe` | `COMPONENT` | `` | [TrashProductionRuntimeProbe.java](../src/main/java/hu/taliann/icesmp/trash/TrashProductionRuntimeProbe.java) |
| `TrashRecyclePool` | `COMPONENT` | `` | [TrashRecyclePool.java](../src/main/java/hu/taliann/icesmp/trash/TrashRecyclePool.java) |
| `TrashRelicActivationService` | `SERVICE` | `feature.trash-history` | [TrashRelicActivationService.java](../src/main/java/hu/taliann/icesmp/trash/TrashRelicActivationService.java) |
| `TrashRelicBehavior` | `COMPONENT` | `` | [TrashRelicBehavior.java](../src/main/java/hu/taliann/icesmp/trash/TrashRelicBehavior.java) |
| `TrashRelicPolicy` | `COMPONENT` | `` | [TrashRelicPolicy.java](../src/main/java/hu/taliann/icesmp/trash/TrashRelicPolicy.java) |
| `TrashRelicRuntime` | `COMPONENT` | `` | [TrashRelicRuntime.java](../src/main/java/hu/taliann/icesmp/trash/TrashRelicRuntime.java) |
| `TrashRuntimeTelemetry` | `COMPONENT` | `` | [TrashRuntimeTelemetry.java](../src/main/java/hu/taliann/icesmp/trash/TrashRuntimeTelemetry.java) |
| `TrashSourceBias` | `COMPONENT` | `` | [TrashSourceBias.java](../src/main/java/hu/taliann/icesmp/trash/TrashSourceBias.java) |
| `TrashSpatialFractureStore` | `PERSISTENT_STORE` | `feature.trash-spatial-fracture` | [TrashSpatialFractureStore.java](../src/main/java/hu/taliann/icesmp/trash/TrashSpatialFractureStore.java) |
| `TrashVendorService` | `SERVICE` | `feature.trash-vendor` | [TrashVendorService.java](../src/main/java/hu/taliann/icesmp/trash/TrashVendorService.java) |
| `PlatformCapabilities` | `COMPONENT` | `` | [PlatformCapabilities.java](../src/main/java/hu/taliann/icesmp/utils/PlatformCapabilities.java) |
| `SpellHealingUtil` | `COMPONENT` | `` | [SpellHealingUtil.java](../src/main/java/hu/taliann/icesmp/utils/SpellHealingUtil.java) |
| `WarlockCombatState` | `COMPONENT` | `` | [WarlockCombatState.java](../src/main/java/hu/taliann/icesmp/warlock/WarlockCombatState.java) |
| `WarlockGameplayService` | `SERVICE` | `feature.warlock-gameplay` | [WarlockGameplayService.java](../src/main/java/hu/taliann/icesmp/warlock/WarlockGameplayService.java) |
| `WarriorCombatState` | `COMPONENT` | `` | [WarriorCombatState.java](../src/main/java/hu/taliann/icesmp/warrior/WarriorCombatState.java) |
| `WarriorGameplayService` | `SERVICE` | `feature.warrior-gameplay` | [WarriorGameplayService.java](../src/main/java/hu/taliann/icesmp/warrior/WarriorGameplayService.java) |
| `WizardCombatState` | `COMPONENT` | `` | [WizardCombatState.java](../src/main/java/hu/taliann/icesmp/wizard/WizardCombatState.java) |
| `WizardGameplayService` | `SERVICE` | `feature.wizard-gameplay` | [WizardGameplayService.java](../src/main/java/hu/taliann/icesmp/wizard/WizardGameplayService.java) | 

## Immersive UX foundation

The immersive presentation layer is split into four bounded authorities:

- `ux/TooltipEngine`: deterministic semantic section composition. It accepts canonical item snapshots plus a player-aware context, but has no persistence API. Generated archaeology observations therefore remain presentation-only.
- `ux/DialogueEngine`: player-isolated, replaceable dialogue sessions. Timed nodes use the owning player's Folia scheduler and are cancelled on replacement, death, world change, quit and plugin shutdown. Quest dialogue remains defined by `config/quests.yml`; `QuestManager` delegates only sequencing to the engine.
- `ux/MusicDirector`: per-player priority arbitration for resource-pack sound events. The server selects and transitions contexts; looping is authored by the client sound event. No raw audio thread or global player state is used.
- `ux/UnifiedGuiManager` + `GuiSession`/`GuiComponent`: session-owned inventory components. Managed top-inventory clicks and drags are cancelled before dispatch, including shift-click, number-key, cursor and hotbar paths. Session state is cleared on close, replacement, quit and shutdown.

Existing item identity, authored lore, archaeology packet projection and individual GUI implementations remain authoritative until migrated through these seams. This avoids a second item-persistence or quest-definition system. The current PR wires quest dialogue and archaeology presentation; existing inventory GUIs can adopt `GuiComponent` incrementally.

The foundation is code-complete only after the relevant source review. The implementation does not claim a Gradle build, automated execution, live server playtest or resource-pack visual validation until those are run separately.

<!-- icesmp-ux-foundation-doc -->
