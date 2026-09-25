# IceSMP current architecture

<!-- DOC-AUTHORITY: CURRENT_STATE_ONLY -->

Ez a dokumentum kizárólag a ténylegesen implementált állapotot írja le a DOC-00 indulási baseline-on (`staging` @ `121c3b9cccca15c3e828f2bd363e8ddb17016b73`). A célarchitektúra külön dokumentum: [`ARCHITECTURE_FOUNDATION_PLAN.md`](ARCHITECTURE_FOUNDATION_PLAN.md).

## 1. Runtime topológia

Az IceSMP jelenleg egyetlen Paper/Folia plugin.

```text
Paper/Folia
└── IceSMP (JavaPlugin)
    ├── CommandLifecycle
    ├── ResourcePackListener
    ├── TransientEntities
    ├── IceSMPCore
    ├── PrologueRuntime
    └── PrologueRuntimeConfigOverlay
```

Az entrypoint `onEnable()` sorrendje:

1. default config mentése;
2. resource-pack listener létrehozása és regisztrációja;
3. transient entity runtime install;
4. `IceSMPCore` létrehozása és `enable()`;
5. Prologue runtime és config overlay install;
6. WorldGuard bridge health diagnosztika;
7. command admission megnyitása;
8. resource-pack resend;
9. runtime probe-ok.

Az `onDisable()` bezárja a command admissiont, leállítja a Prologue réteget, meghívja a Core disable útvonalát, lezárja a resource-pack listenert, eltakarítja a transient entity state-et és ellenőrzi a statikus facade teardownját.

A fail-closed `requestDisable` külön command drain, presentation cleanup, `prepareDisable`, player shutdown completion és bounded deadline lépéseket használ.

## 2. Jelenlegi Core

Az `IceSMPCore` ma központi bootstrap és wiring authority. Nagyszámú manager-, listener-, command-, config-, store- és presentation dependencyt konstruál és regisztrál.

Ez **jelenlegi kompatibilitási tény**, nem jövőbeli kötelező minta. Új moduláris runtime, ModuleGraph vagy LegacyCoreModule még nincs implementálva.

A jelenlegi Core felelősségei többek között:

- config és message bootstrap;
- domain manager/service wiring;
- programmatic command registration;
- Bukkit listener registration;
- persistent store koordináció;
- scheduler/task indítás;
- player session cleanup összeállítása;
- HUD/GUI/client bridge wiring;
- event manager lifecycle;
- shutdown és final persistence útvonalak.

Exact manager/listener/task/store lista a generált repository inventoryban él.

## 3. Command surface

A plugin Paper Brigadier `BasicCommand` objektumokat regisztrál programmatikusan. Az `IceSMP.registerCommand` a commandokat a `CommandLifecycle` wrapperen vezeti át, így admission close és drain támogatott.

A root commandok többsége az `IceSMPCore` kézi regisztrációjából, a Prologue command a `PrologueRuntime` installból érkezik. A command-, alias-, subcommand- és permission-truth a tényleges regisztráció és command implementáció; a human guide nem tart fenn teljes kézi listát.

## 4. Listener és task ownership

A jelenlegi listener- és taskownership vegyes:

- sok listener közvetlenül a Core-ban regisztrált;
- egyes domain runtime-ok saját listener/task lifecycle-t tartanak;
- Folia entity/region/global scheduler használata több specializált wrapperen és manageren keresztül történik;
- közös, module-owned ListenerRegistry vagy ManagedTaskRegistry még nincs.

A jelenlegi cleanup szerződéseket meg kell őrizni. Új background resource explicit owner nélkül nem adható hozzá.

## 5. Folia contract

A plugin executable metadata szerint Folia-kompatibilis. A runtime szabály:

- player/entity mutation entity scheduleren;
- block/location/chunk mutation region scheduleren;
- globális koordináció global region scheduleren;
- cross-region hatás a target saját schedulerére hopol;
- file I/O nem blokkolhat entity/region threadet;
- delayed callbacknek stale session/instance ellen védettnek kell lennie.

A repository regressziós suite-jai source- és behavior-contractokat ellenőriznek; valódi multi-region staging továbbra is release gate.

## 6. Config és authored content

Jelenlegi forrásrétegek:

```text
src/main/resources/config.yml
src/main/resources/config/**/*.yml
src/main/resources/content/**/*.yml
src/main/resources/messages.yml
src/main/resources/messages/**/*.yml
src/main/resources/datapack/**
resource-pack/**
```

A `ConfigManager` és domain-specifikus catalog/validator osztályok packaged defaults, telepített config és runtime snapshotok között közvetítenek. Több domain már immutable/typed snapshotot használ, de a repositoryban közvetlen patholvasás és legacy bridge is létezik.

Általános `DataPlatform`, backend registry vagy typed config platform még nincs. Invalid candidate nem válhat automatikusan authoritative állapottá; a meglévő domain reload/validation szerződés az irányadó.

## 7. Adat- és persistence-authority

A jelenlegi rendszer több, domainhez illeszkedő tárolási modellt használ:

- **PlayerProfile aggregate/sectionök** — player progression és több durable domain state;
- **YAML/file store-ok** — domain-specifikus runtime state és operator adatok;
- **WAL/receipt/operation ID protokollok** — gazdasági és kritikus durable műveletek;
- **specializált journalok** — például nagy world/terrain recovery;
- **packaged authored content** — Gitben verziózott canonical content;
- **runtime projection/cache/PDC** — csak akkor authoritative, ha a domain szerződés kifejezetten így definiálja; általánosan nem válhat shadow authorityvé.

Production database backend, JDBC/ORM, SQL schema vagy connection pool nincs a foundation részeként bevezetve.

## 8. Fő domainek

A kód jelenleg részben package-ekre és manager/service-ekre tagolt, de még nem teljes module boundarykkal.

- Profile, class/spec, progression, spell, talent és resource;
- itemization, equipment, relic, crafting és professions;
- currency, bank, market, exchange és treasury;
- factions, guild/law/sin/territory és protection;
- quests, NPC binding, dialogue, lore és achievements;
- PvE creature profiles, scaling, abilities, pets/minions és loot;
- world events, seasons, raids, Prologue és Corruption;
- party, chat, moderation és admin tooling;
- HUD, menus, Placeholder/client bridge és resource pack.

A határok nem mindenhol explicit API-k. UI és event wiring több helyen concrete manager dependencyt használ; ez célzott migrációs adósság.

## 9. PlayerProfile és class/spec

A class/spec durable state a PlayerProfile strukturált sectionjében él. A fizikai class artifact, HUD, GUI és PDC csak presentation vagy rebuildable mirror lehet; nem hozhat létre második progression authorityt.

A támogatott class/spec gameplay concrete service-ekkel és explicit transient cleanup-pal működik. A több classból közös generikus mechanic DSL szándékosan nincs. A történeti Java/config azonosítókban előforduló régi rollout-token kompatibilitási ID lehet, nem új architektúrabranding.

## 10. Authored PvE

A canonical creature spawn/stat/ability útvonal authored `MobTemplate`/profile, közös level/rank projection és `MobAbilityRuntime` attachment köré épül. World/encounter manager timingot, placementet, roster/wave/narrative flow-t és settlementet birtokol; nem építhet második combat engine-t.

Summon/add lifecycle bounded ownerrel, lifespan-nal és cleanup-pal működik. Reward ownership külön van választva generic, event és none útvonalakra.

## 11. Eventek és Prologue

A world-event rendszer jelenleg több legacy managerből, közös spawn/admission segédekből és domain-specifikus persistence/restart útvonalakból áll. Egységes natív Event Platform, catalog és instance/resource registry még nincs.

A Prologue külön runtime-ként installálódik, saját event gate-, encounter-, objective- és finale-flow-val. A current Prologue contractot a [`PROLOGUE.md`](PROLOGUE.md), az operátori lépéseket az [`ADMIN_GUIDE.md`](ADMIN_GUIDE.md), a fizikai világkötéseket a [`BUILDER_GUIDE.md`](BUILDER_GUIDE.md) foglalja össze.

## 12. UI és client boundary

A HUD/GUI/client réteg több immutable projectiont használ, de nem minden felület választott le concrete managerekről. A resource pack:

- first-party HUD/font/sprite/assets forrás;
- item `ITEM_MODEL` és wearable `EQUIPPABLE.assetId` szerződés;
- deterministic ZIP/publish tooling;
- immutable hash-es URL és runtime resend;
- tooltip style és client-only viewer projection.

Resource-pack asset vagy lore nem gameplay authority.

## 13. Build és validation

A Gradle build Java 21 toolchaint és Paper/Folia 1.21.11 API-t használ. A `check` task mellett számos domain regression és Python audit/generator fut. A repository docs inventory és link/consistency tooling külön CI-workflowban él.

Minimum:

```bash
./gradlew build --console=plain --no-daemon
python3 scripts/check_consistency.py
python3 scripts/check_markdown_links.py --root .
python3 scripts/check_documentation.py
```

## 14. Ismert current-state adósság

- központi, nagy `IceSMPCore` wiring;
- kézi command/listener/task/store listák;
- vegyes lifecycle ownership;
- egyes config reload bridge-ek és közvetlen patholvasások;
- UI→concrete manager dependencyk;
- eventenként eltérő lifecycle/persistence modellek;
- legacy gyűjtőpackage-ek;
- exact leltárak korábbi kézi dokumentálása.

Ezek nyitott migrációs témák, nem felhatalmazás új hasonló adósságra. Prioritás és phase: [`ROADMAP.md`](../ROADMAP.md).

## 15. Kifejezetten nem implementált foundation elemek

A következők a DOC-00 végén még **nem léteznek executable runtime-ként**:

- `IceSmpRuntime` és ModuleGraph;
- `LegacyCoreModule`;
- közös listener/task/store/command/session/reload/health registryk;
- általános DataPlatform/backend registry/migration coordinator;
- teljes typed config platform;
- PlayerSessionCoordinator;
- egységes query/presentation boundary;
- natív Event Platform és catalog;
- production database backend.

Ezeket a future plan írja le; current-state dokumentumba csak az integrálásuk után kerülhetnek kész komponensként.
