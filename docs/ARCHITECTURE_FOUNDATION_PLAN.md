# IceSMP 1.0 Architecture Foundation

<!-- DOC-AUTHORITY: FUTURE_PLAN_AUTHORITY -->
## Teljes architekturális rework-, dokumentációs és adatplatform-előkészítési terv

**Állapot:** normatív tervezési dokumentum, implementáció még nem történt  
**Készült:** 2026. szeptember 25.  
**Repository:** `MilCsik09/IceSMP`  
**Tervezési baseline:** `staging` @ `121c3b9cccca15c3e828f2bd363e8ddb17016b73`  
**Program neve:** IceSMP 1.0 Architecture Foundation  
**Elsődleges integrációs ág:** `architecture/1.0-integration`  
**Első dokumentációs ág:** `architecture/1.0-docs-authority`  
**Első runtime ág:** `architecture/1.0-runtime-foundation`

> A repository tényleges állapota az implementáció megkezdésekor elsőbbséget élvez a fenti pillanatképpel szemben. A munkakezdéskor kötelező újra lekérni és dokumentálni a `staging` pontos HEAD-jét. A tervezés jelenlegi baseline-ja a fenti commit.

---

## 0. Elnevezési, dokumentációs és instrukciós authority

### 0.1. Az új architektúra az 1.0 alapja

A program neve **IceSMP 1.0 Architecture Foundation**. Nem egy már kiadott, stabil architektúra „második verzióját” építjük, hanem azt az alapot, amelyre az IceSMP első valódi 1.0 kiadása épül.

Kötelező névadási szabályok:

- új branch, package, class, dokumentum, modul, milestone és user-facing megnevezés nem kaphat `V2`, `v2`, `2.0`, `next-gen` vagy hasonló párhuzamos-platform brandinget;
- a jelenlegi dokumentációban szereplő olyan elnevezéseket, mint „Profile v2”, „Quest Framework v2” vagy „Architecture 2.0”, forrásellenőrzés után semleges, tartós névre kell cserélni;
- az új rendszer nem „legacy mellett élő új verzió”, hanem a legacy kód fokozatosan átvitt, majd kiváltott 1.0-s célarchitektúrája;
- technikai sémaverziók, protokollverziók, adatbázis-migrációs sorszámok, Minecraft/Paper verziók és valódi release-verziók továbbra is használhatnak számot. Ezek nem marketing- vagy architektúrabrandingek, ezért nem szabad őket pusztán névadási okból átírni;
- persisted kulcsot, schema ID-t vagy publikus protokollazonosítót csak külön kompatibilitási/migrációs tervvel lehet átnevezni.

### 0.2. DOC-00 — teljes Markdown-review minden kódmódosítás előtt

A kivitelezés első tényleges fázisa nem Java-refaktor, hanem a repository **összes tracked Markdown fájljának teljes, elejétől végéig történő reviewja**.

Kötelező scope:

- `AGENTS.md` és `CLAUDE.md`;
- `README.md` és `ROADMAP.md`;
- az öt kanonikus human-facing guide;
- `docs/ARCHITECTURE.md`, `docs/QUESTS.md`, content/lore/resource-pack és minden egyéb supporting dokumentum;
- `.github/`, agent-, tooling- és bármely más tracked alkönyvtár Markdown fájljai;
- az `IceSMPGuides` mirrorban megtartandó dokumentumok;
- minden olyan MD, amely jelenlegi állapotot, authorityt, workflow-t, commandot, konfigurációt, fájlszámot, branch-nevet vagy implementációs tényt állít.

A review során minden dokumentumot be kell sorolni:

```text
CANONICAL_CURRENT_STATE
CANONICAL_WORKFLOW
CANONICAL_HUMAN_GUIDE
SUPPORTING_REFERENCE
FUTURE_PLAN
GENERATED_REPORT
HISTORICAL
DUPLICATE_OR_STALE
```

Minden állítást össze kell vetni legalább a releváns forráskóddal, config/content fájlokkal, command- és permission-regisztrációval, valamint a repository aktuális branch/CI állapotával. Külön ellenőrizendő:

- elavult manager-, listener-, class- és package-számok;
- nem létező vagy átnevezett parancsok, permissionök, configkulcsok és fájlok;
- implementáltnak leírt, de csak tervezett funkciók;
- egymással versengő authority-dokumentumok;
- hibás lokális linkek és mirror-drift;
- `V2`/`2.0` branding;
- a jelenlegi monolit Core-t végleges célként bemutató, elavult architektúraszabályok;
- adat-, persistence-, Folia-, shutdown- és failure-path állítások pontossága.

A review eredménye:

1. minden megtartott dokumentum javítása a saját szerepe szerint;
2. duplikált/stale dokumentumok összevonási vagy eltávolítási döntése, indoklással;
3. a jövőbeli terv és a tényleges jelenlegi állapot világos szétválasztása;
4. determinisztikus, nem kanonikus inventory riport a `build/reports/architecture/` alatt;
5. dokumentációs consistency gate a lokális linkekre, authoritylistára, tiltott brandingre és mirror-szinkronra;
6. a `ROADMAP.md` aktualizálása a foundation program és a migrációs hullámok állapotával.

### 0.3. Az `AGENTS.md` teljes felülvizsgálata és újraírása

Az `AGENTS.md` nem egyszerűen néhány új pontot kap. A teljes fájlt forrás alapján újra kell értékelni, majd szükség szerint újra kell írni.

A cél-`AGENTS.md` kötelező témái:

- a jelenlegi repository és build valós, ellenőrzött snapshotja;
- az IceSMP 1.0 Architecture Foundation célja és nem céljai;
- module-, package- és dependency-határok;
- canonical authorityk és tiltott dual-write;
- Folia owner-thread szabályok;
- lifecycle-, task-, listener-, command-, session- és persistence-ownership;
- config/content/data authority és reload szabályok;
- adatbázisra kész repository/adapter határok;
- dokumentációs authority és mirror-policy;
- branch-, PR-, worktree-, multi-agent és file-ownership workflow;
- build, regresszió, consistency, architecture gate és Definition of Done;
- tiltott minták: új Core reflection, új statikus service locator, új legacy gyűjtőpackage, rejtett authority, háttérben maradó task/listener;
- hogyan kell új modult, eventet, persistent store-t, configot, commandot és client-protocol változást hozzáadni.

Az új `AGENTS.md`:

- ne tartson kézzel karbantartott, gyorsan avuló fájlszámokat, ha azok generálhatók;
- ne nevezze a régi `IceSMPCore`-mintát kötelező jövőbeli mintának;
- különítse el a **jelenlegi kompatibilitási szabályt** a **célarchitektúra szabályától**;
- őrizze meg az összes továbbra is helyes Folia-, persistence-, build-, docs- és safety-szerződést;
- hivatkozzon generált inventorykra ott, ahol exact listákra van szükség.

A jelenlegi `AGENTS.md` szabályai addig érvényesek, amíg az új változat reviewzva és commitolva nincs. A `CLAUDE.md` ugyanabban a commitban kapjon kompatibilitási frissítést; ne legyen önálló, versengő authority.

### 0.4. DOC-00 kilépési feltétele

Kódrefaktor csak akkor kezdődhet, ha:

- minden tracked Markdown fájl reviewja elkészült;
- a dokumentumok authority-szerepe dokumentált;
- az `AGENTS.md` frissített vagy újraírt változata elfogadott;
- a `CLAUDE.md` kompatibilitási tartalma szinkronban van;
- nincs új architektúrabranding `V2`/`2.0` néven;
- a plan repositorybeli neve `docs/ARCHITECTURE_FOUNDATION_PLAN.md`;
- a dokumentációs consistency gate zöld;
- a mirrorolandó fájlok az `IceSMPGuides` repositoryban azonos tartalommal elérhetők;
- a review során talált, de nem azonnal javított pontok egyetlen helyen, a `ROADMAP.md`-ban szerepelnek.

---

## 1. Vezetői döntés

Az IceSMP-t nem kell microservice-ekre vagy több pluginra bontani, és nem szabad egyszerre újraírni. A cél egy **moduláris monolit**:

- egyetlen Paper/Folia plugin;
- egyetlen build és kiadási egység;
- több, explicit lifecycle- és authority-határral rendelkező belső modul;
- modulonként publikus API, saját bootstrap, persistence, task-, listener- és session-ownership;
- fokozatos migráció a jelenlegi működés megtartásával.

A rework fő célja nem a Java-fájlok számának mesterséges csökkentése. Átmenetileg több, kisebb és pontosabb szerepű osztály is létrejöhet. A cél:

1. kevesebb globális manager;
2. kevesebb kézi wiring;
3. egyértelmű durable authority;
4. reprodukálható startup/shutdown;
5. típusos config reload;
6. vékony Bukkit/Folia adapterek;
7. immutable query/view határ a UI felé;
8. egy új feature ne igényeljen módosítást tíz központi registryben;
9. az Event Platform legyen az első teljes referencia-migráció;
10. a meglévő PlayerProfile-, WAL-, PvE- és Folia-biztonsági alapok megőrzése.

---

## 2. Miért kell külön branch?

Ez a munka hosszú ideig érintheti:

- a plugin belépési pontját;
- az `IceSMPCore` lifecycle-ját;
- command-, listener-, scheduler- és store-regisztrációt;
- config reloadot;
- session cleanupot;
- UI queryket;
- eventek lifecycle-ját;
- package-struktúrát.

A `staging` közvetlen módosítása túl nagy kockázatot jelentene. Az architekturális rework idején a normál gameplay-fejlesztéseknek és hibajavításoknak továbbra is biztonságosan kell haladniuk.

### 2.1. Ágstratégia

```text
staging
  └── architecture/1.0-integration
        ├── architecture/1.0-docs-authority
        ├── architecture/1.0-runtime-foundation
        ├── architecture/1.0-resource-registries
        ├── architecture/1.0-config-data-foundation
        ├── architecture/1.0-session-foundation
        ├── architecture/1.0-presentation-boundary
        ├── architecture/1.0-event-facade
        └── ...
```

Az integrációs ág és a fáziságak neve szándékosan nem tartalmaz `V2`/`2.0` brandinget. A `1.0` azt jelöli, hogy ez a program hozza létre az első stabil IceSMP-architektúra alapját.

Az első két PR sorrendje kötött:

1. `architecture/1.0-docs-authority` → teljes Markdown-review, `AGENTS.md`/`CLAUDE.md`, terv és guardrail;
2. `architecture/1.0-runtime-foundation` → csak a dokumentációs PR integrálása után induló runtime foundation.

### 2.2. Kötelező szabályok

- Közvetlen commit a `staging` ágra tilos.
- Közvetlen feature-fejlesztés az integrációs ágon is kerülendő; oda phase PR-ek kerülnek.
- Minden phase branch az aktuális integrációs ágról indul. A runtime branch csak a dokumentációs authority PR integrálása után nyitható.
- A phase PR célága az `architecture/1.0-integration`.
- A `staging` változásait csak ellenőrzött kapuknál merge-eljük az integrációs ágba.
- Publikált, review alatt álló ágat nem force-pusholunk.
- Minden mérföldkő végén teljes build, consistency, regresszió és célzott Folia-próba szükséges.
- Az integrációs ágat csak milestone-onként érdemes visszamerge-elni `staging`re.
- A baseline commitot és minden későbbi `staging`-szinkront a ROADMAP-ban/a PR leírásában rögzíteni kell.
- Az integrációs ágon ajánlott branch protection és kötelező CI check.

### 2.3. Merge-mérföldkövek

Az egész többhónapos reworköt nem egyetlen PR-ban kell `staging`re tenni.

Javasolt milestone-ok:

- **Milestone A:** teljes Markdown/AGENTS authority review + Runtime Kernel + registry foundation;
- **Milestone B:** config reload + session lifecycle + health;
- **Milestone C:** query/presentation boundary + Event façade;
- **Milestone D:** natív Event Platform és első parity eventek;
- **Milestone E:** Quest/Narrative és Faction/Economy szétválasztás;
- **Milestone F:** class/progression, package-konszolidáció és legacy Core eltávolítása.

Mindegyik milestone önállóan buildelhető, visszagörgethető és működő rendszer legyen.

---

## 3. Nem célok

Az IceSMP 1.0 Architecture Foundation nem jelentheti:

- az egész plugin egyszerre történő újraírását;
- gameplay-mechanikák indokolatlan módosítását a foundation fázisokban;
- YAML workflow- vagy programozási nyelv építését;
- saját általános DI-framework létrehozását;
- minden listenert egyetlen univerzális listenerbe olvasztani;
- minden schedulert egyetlen tickbe kényszeríteni;
- minden persistence-megoldást egyetlen YAML-fájlba tenni;
- a PlayerProfile authority megkerülését;
- második mob scaling-, ability-, loot- vagy combat engine építését;
- a Corruption terrain WAL generic event-node-okká alakítását;
- package move-okat valódi authority- és dependency-refaktor nélkül;
- kompatibilitási dual-write authorityt;
- reflectionalapú automatikus module discoveryt;
- adatbázis-driver, ORM vagy production database backend bevezetését a foundation első hullámaiban;
- YAML/fájl authority azonnali lecserélését vagy rejtett dual-write-ot;
- minden adat generikus CRUD repositoryba kényszerítését;
- technikai schema-/protocol-verziók átnevezését pusztán a branding eltávolítása miatt.

---

## 4. Architektúraalapelvek

### 4.1. Egy állapot – egy authority

Minden tartós vagy aktív állapotnak pontosan egy gazdája van.

Példák:

```text
PlayerProfile aggregate      → Profile Module
wallet                       → Economy/Profile section authority
faction membership           → Profile faction section + Faction application facade
item identity                → Itemization Module
authored creature profile    → Combat/PvE Module
event lifecycle              → Event Platform
event terrain                → instance-owned terrain resource/journal
HUD view                     → projection, soha nem authority
```

Projection, cache, PDC és YAML nem válhat párhuzamos authorityvé.

### 4.2. Moduláris monolit

A modulok ugyanabban a pluginban futnak, de:

- csak publikus API-kon kommunikálnak;
- belső implementation package-eket más modul nem importál;
- saját bootstrapjuk van;
- saját lifecycle- és resource-ownershipjük van;
- a top-level runtime csak module API-kat ismer.

### 4.3. Explicit dependencyk

Tiltott új kódban:

```java
SomeManager.current();
engine.getManager(...);
Bukkit.getPluginManager().getPlugin(...) mint belső DI;
reflection az IceSMPCore mezőire;
kötelező setter-injection;
```

Kötelező dependency konstruktorból, module dependencyből vagy szűk porton érkezik.

### 4.4. Domain és Bukkit szétválasztása

A domain package:

- nem importál Bukkit-, Paper- vagy Folia-típusokat;
- immutable value objectokkal dolgozik;
- dependency-free regressziós tesztben futtatható.

A Bukkit/Folia adapter:

- eventet fogad;
- region-local validációt végez;
- application commandot vagy immutable signalt küld;
- nem birtokol tartós authorityt.

### 4.5. Folia ownership

- Player/entity művelet: entity scheduler.
- Block/location művelet: region scheduler.
- Globális koordináció: global region scheduler vagy explicit single-writer.
- IO: Bukkit-objektumtól mentes executor.
- Cross-region kommunikáció: immutable value/signal.
- Stale callback: generation/revision fencing.
- Nincs globális entity- vagy blockscan.

### 4.6. Code-first gameplay

A komplex event, quest, class és combat mechanika Javában él.

YAML feladata:

- authored content;
- roster;
- template;
- loot;
- szöveg;
- operator tuning.

YAML nem vezérel általános control flow-t.

### 4.7. Strangler migration

Az új platform fokozatosan veszi körbe és váltja ki a jelenlegi rendszert:

```text
legacy implementation
→ facade/adapter
→ engine-orchestrated shell
→ native module
→ legacy authority eltávolítása
```

Köztes állapotban mindig pontosan meg kell nevezni a canonical authorityt.


### 4.8. Storage-backend függetlenség

A domain és application réteg nem tudhatja, hogy az adat jelenleg YAML-, JSON-, fájl-, PlayerProfile-, journal- vagy később adatbázis-backed.

- A modul saját, domain-specifikus repository/port szerződést publikál.
- A YAML/fájl/DB implementáció a `persistence` vagy `integration` adapterben él.
- A domain nem importál `YamlConfiguration`, JDBC-, ORM- vagy database-driver típust.
- A tartós write-ok operation ID-val, revisionnel és dokumentált tranzakciós boundaryval rendelkeznek, ahol erre szükség van.
- A repository API-k I/O-szempontból aszinkron-kompatibilisek; region/entity thread nem blokkolhat adatbázis- vagy fájl-I/O-ra.
- A későbbi database backend bootstrapkor választható adapter lesz, nem `if (databaseEnabled)` ágak tömege a gameplaykódban.

### 4.9. Dokumentáció mint architekturális authority

A dokumentáció szerepe explicit:

- az `AGENTS.md` a fejlesztési és agent workflow authority;
- a `docs/ARCHITECTURE.md` kizárólag a ténylegesen implementált current state-et írja;
- a `docs/ARCHITECTURE_FOUNDATION_PLAN.md` a még folyamatban lévő cél- és migrációs terv;
- a `ROADMAP.md` az egyetlen nyitott finding/backlog authority;
- a human-facing guide-ok nem tartalmazhatnak belső, spekulatív architektúraígéreteket;
- exact inventoryk generált riportok, nem kézzel karbantartott narratív listák.

---

## 5. Célarchitektúra

```text
IceSMP (JavaPlugin)
└── IceSmpRuntime
    ├── PlatformKernel
    ├── DataPlatform
    ├── ProfileModule
    ├── ProgressionModule
    ├── ItemizationModule
    ├── ProfessionModule
    ├── EconomyModule
    ├── FactionModule
    ├── WorldModule
    ├── SeasonModule
    ├── CombatPveModule
    ├── EventPlatform
    ├── QuestNarrativeModule
    ├── TrashModule
    ├── SocialModerationModule
    └── UiClientModule
```

### 5.1. Platform Kernel

```text
platform/
├── runtime/
├── config/
├── persistence/
├── execution/
├── session/
├── commands/
├── messaging/
├── observability/
└── integration/
```

Felelőssége:

- module graph;
- lifecycle;
- command gate;
- listener ownership;
- task ownership;
- persistent-store coordination;
- config snapshot és reload;
- player session lifecycle;
- module health;
- committed domain event publication;
- platform integration backendek;
- backend-, migration- és data-health koordináció a `DataPlatform` felé.

Nem tartalmaz gameplay domainlogikát és nem birtokol domainadatot.

---

## 6. Module lifecycle

### 6.1. Szerződések

```java
public interface IceSmpModule {
    ModuleDescriptor descriptor();
    ModuleRuntime create(ModuleContext context);
}
```

```java
public interface ModuleRuntime {
    void load();
    void start();
    void quiesce();
    CompletionStage<Void> checkpoint();
    void stop();
}
```

```java
public record ModuleDescriptor(
        ModuleId id,
        Set<ModuleId> requiredModules,
        PersistenceCriticality persistence,
        ReloadCapability reloadCapability
) {}
```

### 6.2. Állapotgép

```text
NEW
→ CREATED
→ LOADING
→ LOADED
→ STARTING
→ READY
→ QUIESCING
→ CHECKPOINTING
→ STOPPING
→ STOPPED

hiba:
→ DEGRADED
→ RECOVERY_REQUIRED
→ FAILED
```

### 6.3. ModuleGraph

A runtime:

- validálja a hiányzó dependencyket;
- tiltja a ciklust;
- topologikus sorrendben indít;
- fordított sorrendben állít le;
- modulonként health state-et tart;
- startuphiba esetén fail-closed módon takarítja a már elindult modulokat.

---

## 7. Közös resource registryk

### 7.1. ListenerRegistry

```java
ListenerHandle register(ModuleId owner, Listener listener);
```

Tulajdonságok:

- ownerhez kötött;
- duplicate registration diagnosztika;
- modul stopkor automatikus unregister;
- health/inspect felület.

### 7.2. ManagedTaskRegistry

```java
TaskHandle register(
    ModuleId owner,
    TaskId id,
    ExecutionTarget target,
    TaskSchedule schedule,
    Runnable work
);
```

Tulajdonságok:

- explicit owner;
- quiesce;
- cancel;
- reschedule;
- last-run és failure state;
- duplicate ID tiltás;
- exception isolation;
- bounded retry policy.

### 7.3. StoreRegistry

A meglévő `PersistentStoreCoordinator` fölött:

- modulonként store regisztráció;
- startup order;
- criticality;
- autosave eligibility;
- final checkpoint;
- health.

### 7.4. CommandRegistry

A meglévő `CommandLifecycle` fölött:

- module owner;
- alias és permission metadata;
- automatikus command drain;
- module stopkor delegate retirement.

### 7.5. PlayerSessionRegistry

- join;
- profile-ready;
- quit;
- kick;
- disable;
- session generation;
- idempotens cleanup.

### 7.6. ReloadParticipantRegistry

- typed candidate config;
- prepare/validate/apply;
- rollback vagy previous snapshot megtartása;
- scheduler reschedule hivatalos handle-en.

### 7.7. HealthContributorRegistry

Minden modul és fontos resource állapotot publikál:

```text
READY
DEGRADED
PAUSED
RECOVERY_REQUIRED
STOPPING
FAILED
```

---

## 8. Konfigurációs célrendszer

A jelenlegi immutable `ConfigSnapshot` és authority/reload policy megmarad.

### 8.1. Új komponensek

```text
ConfigRepository
ConfigSchemaRegistry
ConfigPublisher
ConfigReloadCoordinator
ModuleConfig<T>
ReloadParticipant<T>
```

### 8.2. Reload protokoll

```text
1. candidate fájlok betöltése
2. schema és authority validáció
3. érintett modulok prepare
4. minden prepare sikeres
5. snapshot publish
6. apply
7. taskok szabályos reschedule-je
8. hiba esetén korábbi snapshot megtartása vagy modul DEGRADED állapot
```

### 8.3. Kötelező eredmény

A célállapotban:

- nincs reflection az `IceSMPCore`-ra;
- nincs private scheduler-method meghívás;
- nincs private timestamp field átírás;
- listener reload module lifecycle API-n keresztül történik.

---

## 9. Adat- és adatbázis-előkészítés

Az adatbázis **nem része az első implementation milestone-nak**, de a teljes architektúrát már most úgy kell felépíteni, hogy később fájl- vagy YAML-authority helyett adatbázis-adaptert lehessen bekötni a domain és gameplay újraírása nélkül.

### 9.1. Négy külön adatfajta

| Adatfajta | Jelenlegi tipikus forrás | Jövőbeli backend lehetőség | Szabály |
|---|---|---|---|
| Authored canonical content | packaged YAML Gitben | versioned DB/content service | immutable, verziózott snapshot |
| Operator config/tuning | `config/*.yml`, override | DB-backed operator settings | authority- és provenance-rétegek megmaradnak |
| Durable domain/runtime state | PlayerProfile, YAML store, journal | relációs vagy más transactional DB | domain-specifikus repository és explicit transaction |
| Nagy/streaming world state | terrain WAL, recovery fájl | specializált DB/object/file store | nem kényszerítendő generikus relációs modellbe |

A négy kategóriát nem szabad egyetlen „mindent az adatbázisba” absztrakcióba összemosni.

### 9.2. DataPlatform szerepe

```text
DataPlatform
├── BackendRegistry
├── DataHealthRegistry
├── MigrationCoordinator
├── TransactionContextFactory
├── ContentSourceRegistry
├── ConfigSourceRegistry
└── RepositoryAdapterRegistry
```

A `DataPlatform`:

- kapcsolatot, backendéletciklust, migration futtatást és health-et koordinál;
- nem definiál általános gameplay CRUD API-t;
- nem birtokol faction-, quest-, event- vagy wallet-adatot;
- nem enged nyers connectiont vagy SQL-t a domain/application rétegbe;
- modulonként regisztrált repository adaptereket és migration participantokat kezel.

### 9.3. Domain-specifikus repository portok

Javasolt minta:

```java
public interface QuestProgressRepository {
    CompletionStage<QuestProgressSnapshot> load(UUID playerId);
    CompletionStage<CommitResult> commit(QuestProgressMutation mutation);
}
```

Nem javasolt:

```java
GenericRepository<Object, Object>
```

A port:

- domainnyelvet használ;
- nem szivárogtat `YamlConfiguration`, JDBC vagy ORM típust;
- revisiont és idempotens operation ID-t használ, ahol szükséges;
- később ugyanazzal az API-val kaphat YAML-, PlayerProfile- vagy database adaptert.

### 9.4. Config- és content-source absztrakció

A konfigurációs rétegek tartós sorrendje:

```text
packaged canonical content/defaults
→ persistent operator settings
→ explicit runtime overrides
```

A jövőben a persistent operator settings és akár az authored content forrása lehet adatbázis, de a source provenance nem veszhet el.

Szükséges típusok:

```text
VersionedContentSnapshot<T>
TypedConfigSnapshot<T>
ContentRevision
ConfigGeneration
SourceProvenance
```

A `YamlConfiguration` kizárólag a fájladapterben jelenhet meg. A modulok typed snapshotot kapnak.

### 9.5. Spell-, class- és balance-adatok

A spellértékeket külön kell választani:

```text
SpellCatalog                 → struktúra, identity, iskola, képességdefiníció
SpellTuningSnapshot          → cooldown, cost, damage, radius, skálázás
SpellContentRevision         → authored content verzió
SpellTuningGeneration        → operator/live tuning generáció
```

A runtime cast közben továbbra is az aktuális, atomikusan publikált tuning snapshotot olvashatja, így a jelenlegi live-tuning viselkedés megőrizhető. Később ugyanaz a snapshot DB-ből is felépíthető.

Kötelező cél:

- a spell és class gameplay ne olvasson szétszórtan nyers config pathokat;
- a path parsing egy adapter/catalog builder felelőssége;
- aktív, tartós folyamat szükség esetén pinelheti a content/tuning revisiont;
- invalid database vagy file candidate nem írhatja felül az utolsó jó snapshotot.

### 9.6. Aszinkron és Folia szerződés

- adatbázis- és fájl-I/O nem futhat entity- vagy region threaden blokkolva;
- repository write `CompletionStage` vagy explicit operation/outbox boundary mögött történik;
- callback visszatéréskor session/module/instance generation ellenőrzés szükséges;
- shutdownkor a modul quiesce után draineli vagy checkpointolja a pending write-okat;
- connection pool és DB health a DataPlatform lifecycle tulajdona.

### 9.7. Jövőbeli database cutover

A tényleges database bevezetés külön, későbbi program:

```text
DATA-DB-00  backend kiválasztás és production requirements
DATA-DB-01  schema + migration tooling
DATA-DB-02  export/import és checksum verification
DATA-DB-03  non-authoritative shadow verification
DATA-DB-04  explicit authority cutover
DATA-DB-05  file backend retirement csak stabil release után
```

Szabályok:

- nincs tartós dual-write authority;
- shadow write/read csak diagnosztikai, nem authoritative;
- cutover visszaállítható és auditált;
- minden rekordhoz schema/revision/provenance tartozik;
- vendor döntés — például PostgreSQL, SQLite vagy más backend — csak requirements és terhelési mérés után történik;
- a terrain és más nagy world journal maradhat specializált tárolóban akkor is, ha a többi domain adatbázisba kerül.

### 9.8. Amit most kell megtenni, database nélkül

- teljes data-authority inventory;
- közvetlen YAML/config-path olvasások feltérképezése;
- új domain/application kódban storage adapter leakage tiltása;
- typed config/content snapshot szabály;
- module-owned repository/migration registration seam;
- architecture gate a domainből importált YAML/JDBC/ORM típusokra;
- persistence- és migration-health helyének kialakítása;
- minden új durable statehez documented repository authority.

Az első foundation ág **nem ad hozzá database drivert, ORM-et, connection poolt, SQL-sémát vagy production database configot**.

---

## 10. Session Platform

```java
public interface PlayerSessionParticipant {
    default void onJoin(PlayerSessionContext context) {}
    default void onProfileReady(PlayerSessionContext context) {}
    default void onQuit(UUID playerId) {}
    default void onKick(UUID playerId) {}
    default void onDisable(UUID playerId) {}
}
```

A `PlayerSessionCoordinator`:

- egyetlen Bukkit adaptert tart;
- regisztrált participantokat futtat;
- kezeli a Profile ready állapotot;
- garantálja az idempotens cleanupot;
- session generationt biztosít stale callback ellen;
- timeout és health diagnosztikát ad.

A `PlayerSessionCleanupListener` többtucat-paraméteres konstruktora megszűnik.

---

## 11. Query és presentation határ

### 11.1. Modul query API-k

```java
FactionView factionOf(UUID playerId);
WalletView walletOf(UUID playerId);
ProgressionView progressionOf(UUID playerId);
EventView activeEventFor(UUID playerId);
QuestView questsOf(UUID playerId);
```

### 11.2. HUD felosztása

```text
HudProjectionService
HudSnapshotRepository
HudRenderer
HudSessionService
HudClientRouteAdapter
```

A projection a játékos owner threadjén készít immutable `PlayerHudView`-t. A renderer nem kap domain managerreferenciát.

### 11.3. GUI és command

```text
GUI adapter ─┐
             ├→ application use case
Command ─────┘
```

A GUI nem `player.performCommand()` útján használja az üzleti logikát.

---

## 12. Domain eventek és portok

### 12.1. Közvetlen port

Kritikus query vagy command esetén:

```java
FactionQueries
WalletOperations
SeasonQueries
QuestProgressPort
CreatureSpawnPort
WorldPlacementPort
```

### 12.2. Commit utáni domain event

```text
FactionMembershipCommitted
WalletCredited
QuestCompleted
EventOutcomeCommitted
ProfessionLevelChanged
PlayerClassChanged
```

Szabályok:

- csak durable commit után;
- consumer idempotens;
- nem helyettesít atomikus pénzügyi vagy inventory tranzakciót;
- nem rejtett request/response bus.

---

## 13. Event Platform

### 13.1. Publikus API

```java
interface EventControl {
    CompletionStage<EventStartResult> start(EventStartRequest request);
    CompletionStage<EventControlResult> pause(UUID instanceId);
    CompletionStage<EventControlResult> resume(UUID instanceId);
    CompletionStage<EventControlResult> stop(UUID instanceId, EventStopReason reason);
}

interface EventQueries {
    Optional<EventView> find(UUID instanceId);
    List<EventView> active();
    List<EventTypeView> catalog();
}

interface EventSignalSink {
    void publish(EventSignal signal);
}
```

### 13.2. Belső komponensek

```text
EventCatalog
EventModuleRegistry
EventInstanceCoordinator
EventAdmissionService
NaturalEventScheduler
EventOperationCoordinator
EventResourceRegistry
EventInstanceStore
EventViewRepository
EventHealthService
```

### 13.3. Code-first event module

```java
interface WorldEventModule<S extends EventModuleState> {
    EventDescriptor descriptor();
    EventStateCodec<S> stateCodec();
    S createInitialState(EventStartContext context, EventStartRequest request);
    EventDecision<S> onStart(EventModuleContext context, S state);
    EventDecision<S> onSignal(EventModuleContext context, S state, EventSignal signal);
    EventDecision<S> onWakeup(EventModuleContext context, S state, EventWakeup wakeup);
    EventDecision<S> onStop(EventModuleContext context, S state, EventStopReason reason);
    EventRecoveryDecision<S> recover(EventRecoveryContext context, S state);
    EventView createView(EventViewContext context, S state);
}
```

### 13.4. Type, variant, instance

```text
type: corruption
variant: ritual
instance: UUID
```

### 13.5. Instance-owned resource

```text
EntityGroupResource
BoundedTerrainResource
StreamingTerrainResource
RouteDriverResource
TrackedTargetResource
ModifierLeaseResource
EconomicTransactionResource
```

A resource lehet komplex és saját journallal rendelkező runtime, de nem lehet második event-lifecycle authority.

### 13.6. Legacy migration

```text
LEGACY_BRIDGE
→ ENGINE_SHELL
→ NATIVE_MODULE
```

### 13.7. Corruption

A végleges felosztás:

```text
CorruptionEventModule
StreamingTerrainResource
CorruptionMobResource
CorruptionEffectResource
DynamicSpatialLease
```

A jelenlegi terrain WAL megmarad és resource backenddé válik.

---

## 14. Modultérkép

| Modul | Canonical authority |
|---|---|
| Data Platform | backend lifecycle, config/content sources, migration coordination, data health |
| Profile | PlayerProfile aggregate, repository, transaction |
| Progression | class, specialization, spell unlock, talent, resource |
| Itemization | item identity, equipment, transformation, relic identity |
| Profession | profession state, recipe, craft transaction |
| Economy | wallet, market, exchange, treasury, shop, crate economy |
| Faction | membership, guild, sin/law, whisper, governance |
| World | territory, claim, protection, placement, anchors |
| Season | season timeline és modifiers |
| Combat/PvE | creature profile, scaling, abilities, pet/minion combat |
| Events | event lifecycle és event modules |
| Quest/Narrative | quest catalog/progress/reward, NPC, lore, chronicle |
| Trash | anomaly/trash runtime és saját state |
| Social/Moderation | party, chat, moderation, vanish, invsee |
| UI/Client | HUD, tablist, menus, resource pack, client bridge |

---

## 15. Dependencyirányok

```text
platform ← minden modul

data platform ← minden durable/config/content modult csak publikus adapteren keresztül szolgál ki

profile ← progression, economy, faction, quest

itemization ← economy, profession, combat, quest

progression ← combat, quest, ui

world ← combat, events, quest

combat ← events

economy/faction/season ← events és quest csak publikus porton

ui → kizárólag query és application API

domain → nem importál Bukkit/Paper/Folia típust
```

Tiltott:

- modul belső implementation package importja;
- UI → concrete manager;
- event → másik eventmanager;
- listener → több domain közvetlen durable mutationja;
- config bridge → private Core field;
- domain/application → `YamlConfiguration`, JDBC, ORM vagy backend-specifikus connection;
- gameplay service → nyers SQL vagy fájlútvonal.

---

## 16. Migrációs program

# Wave 0 – Dokumentációs és instrukciós authority

## DOC-00 – Teljes Markdown-review és naming cleanup

### Feladatok

- minden tracked Markdown teljes elolvasása és forrásellenőrzése;
- authority-besorolás és generated inventory;
- `AGENTS.md` teljes frissítése vagy újraírása;
- `CLAUDE.md` kompatibilitási szinkron;
- `README.md`, `ROADMAP.md`, `docs/ARCHITECTURE.md` és minden érintett guide javítása;
- `V2`/`2.0` branding eltávolítása, technikai verziók megőrzésével;
- `docs/ARCHITECTURE_FOUNDATION_PLAN.md` létrehozása;
- mirror és lokális link ellenőrzés.

### Gate

- minden tracked MD reviewzott;
- új `AGENTS.md` accepted authority;
- dokumentációs consistency zöld;
- runtime branch még nem indult el;
- nincs kód- vagy gameplayváltozás.

---

# Wave 1 – Baseline és guardrail

## AR-00 – Current-state baseline

### Feladatok

- aktuális `staging` HEAD rögzítése;
- teljes build és consistency baseline;
- manager/listener/task/store/current()/setter/reflection inventory;
- gépi dependency graph;
- ciklusok és tiltott importok listája;
- baseline architecture exceptions fájl;
- `scripts/check_architecture.py`.

### Gate

- a script reprodukálható;
- a jelenlegi adósság baseline-ként elfogadott;
- új adósságot diff alapján blokkol;
- nincs gameplayváltozás.

---

# Wave 2 – Runtime foundation

## AR-01 – IceSmpRuntime és LegacyCoreModule

### Feladatok

- `IceSmpRuntime`;
- `ModuleId`;
- `ModuleDescriptor`;
- `IceSmpModule`;
- `ModuleRuntime`;
- `ModuleGraph`;
- `ModuleHealth`;
- `LegacyCoreModule`;
- szükség esetén `LegacyPrologueModule`;
- entrypoint átállítása runtime delegációra.

### Fontos

- az `IceSMPCore` belsejét még nem bontjuk;
- startup/shutdown sorrend byte-for-byte szemantikáját megtartjuk;
- Prologue, ResourcePack, TransientEntities és command drain viselkedése nem romolhat;
- nincs gameplayváltozás;
- nincs package move.

### Gate

- build és regression zöld;
- startup/shutdown lifecycle teszt;
- module graph cycle/missing dependency teszt;
- failure cleanup teszt;
- élő Folia smoke test;
- runtime nélkül nem indítható a Core.

---

## AR-02 – Resource registryk

### Feladatok

- ListenerRegistry;
- ManagedTaskRegistry;
- StoreRegistry;
- CommandRegistry;
- SessionRegistry;
- ReloadRegistry;
- HealthRegistry.

### Migráció

Elsőként wrapper/adapter, nem tömeges refaktor.

### Gate

- modul stopkor owned resource cleanup;
- duplicate ID tiltás;
- inspect és health;
- nincs változás a gameplayben.

---

## AR-03 – Config reload rework

### Feladatok

- typed ReloadParticipant;
- explicit reschedule API;
- reflection bridge célpontok migrációja;
- `ConfigRuntimeReloadBridge` és `AdvancedConfigRuntimeBridge` eltávolítása.

### Gate

- nulla Core reflection;
- invalid candidate nem publikus;
- previous config marad aktív hiba esetén;
- scheduler reload és listener toggle tesztelve.

---

## AR-04 – Session lifecycle

### Feladatok

- PlayerSessionCoordinator;
- PlayerSessionParticipant;
- Profile-ready fázis;
- session generation;
- GameMode/Position snapshot service;
- jelenlegi cleanup ownerök regisztrációja.

### Gate

- többtucat-paraméteres cleanup listener eltűnik;
- új participant központi lista módosítása nélkül regisztrálható;
- join/quit/kick/disable parity.

---

# Wave 3 – Data- és backend-előkészítés

## DATA-00 – Data authority inventory és storage leakage gate

- minden durable/config/content authority katalogizálása;
- közvetlen `YamlConfiguration`, fájlútvonal, ConfigManager path és repository-hozzáférés inventory;
- module-owned repository és migration participant szerződések;
- architecture gate új domain/application storage leakage-re;
- nincs adatbázis-backend és nincs adatmozgatás.

## DATA-01 – Typed config/content snapshotok

- modulonként typed config adapter;
- versioned content snapshot;
- source provenance;
- spell/class tuning catalog boundary;
- direct path readek fokozatos kiváltása compatibility facade mögött.

## DATA-02 – Backend és migration SPI

- DataPlatform lifecycle;
- backend registry;
- migration coordinator;
- data health;
- file/YAML adapterek elsődleges implementációként;
- későbbi DB adapter számára stabil seam, DB driver nélkül.

### Gate

- domain/application réteg storage-backend független;
- file/YAML viselkedés parity;
- nincs dual authority;
- nincs új nyers config path az új modulokban;
- database cutover továbbra is külön, későbbi program.

---

# Wave 4 – Query és UI

## AR-05 – Query API és immutable view-k

### Feladatok

- domain query portok;
- PlayerOverviewView;
- HudProjectionService;
- HudRenderer split;
- menu query contextok leváltása;
- command/GUI közös application use case-ek.

### Gate

- HudManager nem függ eventmanagerektől;
- UI nem kap concrete manager bundle-t;
- Placeholder/client bridge csak immutable snapshotot olvas;
- UI parity teszt.

---

# Wave 5 – Event Platform

## EV-00 – Event façade és catalog

- EventControl;
- EventQueries;
- EventCatalog;
- LegacyEventController;
- `/event`, HUD és admin GUI átállítása;
- nincs mechanikaváltozás.

## EV-01 – Scheduling és admission

- MajorEventGate kiváltása;
- common natural schedule;
- placement és admission különválasztása;
- EventSpawnGuard port mögött marad.

## EV-02 – Native instance/resource foundation

- instance ID;
- lifecycle;
- persistence;
- operation outbox;
- entity/resource routing;
- contribution/reward;
- read model.

## EV-03 – Pilotok

1. Server Challenge;
2. Gathering Buff;
3. Abundance;
4. Blood Moon.

## EV-04 – Parity migráció

- Treasure jelenlegi formában;
- Archeology jelenlegi formában;
- Stranger;
- Caravan;
- Ambient.

## EV-05 – Upgrade-ek

- Archeology teljes rework;
- Treasure teljes rework;
- Invasion;
- Wild Hunt;
- Escort;
- Cultists;
- World Boss;
- Meteor.

## EV-06 – Corruption

- legacy bridge;
- engine shell;
- terrain resource kivonása;
- lifecycle cutover;
- régi authority eltávolítása.

## EV-07 – Campaignök

- Season Finale;
- Prologue;
- crisis-event láncok.

---

# Wave 6 – Nagy domainek

## QR-00 – Quest/Narrative

A `QuestManager` felosztása:

```text
QuestCatalog
QuestProgressService
QuestAssignmentService
QuestRewardCoordinator
CustomQuestRepository
QuestDialoguePort
QuestMarkerProjection
```

## FE-00 – Faction/Economy

```text
FactionMembershipService
FactionProjection
FactionSwitchPolicy
GuildService
WalletService
WalletQueries
CurrencyTokenService
TreasuryService
MarketService
```

## PR-00 – Progression/class slices

- class runtime application/bukkit/presentation split;
- közös catalog;
- session registry;
- HUD projection;
- pet/minion portok.

## PKG-00 – Package consolidation

- legacy `managers/`, `listeners/`, `commands/`, `utils/` fokozatos kiürítése;
- package move csak valódi modulrefaktorral együtt.

---

## 17. Teszt- és CI-stratégia

### 17.1. Minden phase előtt

```bash
./gradlew build --console=plain --no-daemon
python3 scripts/check_consistency.py
```

Ha a dependency repository nem érhető el, a cache-es `javac` csak preflight; nem helyettesíti a valódi buildet.

### 17.2. Új architecture gate

A `scripts/check_architecture.py` ellenőrizze:

- új osztály a legacy `managers/`/`listeners/` gyűjtőcsomagban;
- új Core reflection;
- új statikus `current()` bridge;
- domain → Bukkit import;
- modul-internal cross import;
- UI → concrete manager;
- event hardcoded switch;
- lifecycle nélküli repeating task;
- unowned listener/store/task;
- dependency cycle.

Kezdetben baseline exception fájl engedi a meglévő adósságot, de új adósságot nem.

### 17.3. Runtime lifecycle regressziók

- module dependency order;
- cycle rejection;
- partial startup rollback;
- quiesce;
- checkpoint;
- stop reverse order;
- double stop idempotencia;
- reload failure;
- task exception isolation;
- listener unregister;
- command drain;
- store final checkpoint;
- session cleanup.

### 17.4. Folia acceptance

- region/entity scheduler ownership;
- stale callback;
- player logout pending callback mellett;
- plugin disable pending IO mellett;
- reload task reschedule;
- entity chunk border;
- event/resource cleanup távoli chunkon.

### 17.5. Performance

- startup idő;
- heap;
- task count;
- per-tick allocation;
- signal queue;
- 50–60 játékos;
- corruption terrain load;
- HUD snapshot cost.

---

## 18. Dokumentáció

A dokumentációs munka nem mellékfeladat, hanem a program legelső gate-je.

Repositoryban javasolt normatív terv:

```text
docs/ARCHITECTURE_FOUNDATION_PLAN.md
```

Kötelező:

- a DOC-00 fázisban minden tracked Markdown teljes reviewja;
- az `AGENTS.md` teljes forrásalapú frissítése vagy újraírása;
- a `CLAUDE.md` kompatibilitási szinkronja ugyanabban a commitban;
- azonos relatív útvonalon mirror az `IceSMPGuides` repóba minden megtartott dokumentumnál;
- `ROADMAP.md` fázis-, finding- és állapotfrissítés;
- `docs/ARCHITECTURE.md` csak az implementált current state-et írja;
- a foundation terv a jövőbeli célállapot authorityja, de nem állíthat még nem implementált részt késznek;
- workflow-szabály változásakor az agent dokumentumok és consistency gate együtt változnak;
- minden phase PR-ben migration note, authority delta és dokumentációs delta;
- nincs `V2`/`2.0` branding, kivéve valódi technikai schema/protocol/release verziót;
- exact class/file inventory generált riportból származik, nem kézi számlálásból.

A foundation terv implementációjának végén a tartalma fokozatosan beolvad a `docs/ARCHITECTURE.md` current-state leírásába; a plan csak addig marad külön normatív dokumentum, amíg valódi, nyitott migrációs programot ír le.

---

## 19. Multi-agent munkaszabályok

### 19.1. Fájlok kizárólagos szerkesztése és integráció

- Egy adott fájlt egyszerre csak egy agent módosíthat.
- A fő agent előre ossza ki és tartsa nyilván a szerkesztési felelősségeket.
- Az ugyanazt a fájlt érintő feladatokat egy agent kapja.
- Ha egy alügynök másnak kiosztott fájl módosítását igényli, jelezze a fő agentnek; ne szerkessze önállóan.
- A felelősség átadása csak az előző szerkesztő munkájának lezárása után történhet.
- Párhuzamos olvasás és review megengedett.
- Párhuzamos szerkesztés ugyanazon a fájlon tilos.
- A fő agent végzi az integrációt és a konfliktusfeloldást.
- Nagy shared fájlok — `IceSMP.java`, `IceSMPCore.java`, `build.gradle.kts`, `AGENTS.md`, `CLAUDE.md`, `ROADMAP.md`, `docs/ARCHITECTURE.md` — egy kijelölt integration agent tulajdonában legyenek.

### 19.2. Worktree-stratégia

- Alügynökönként külön worktree és branch.
- A fő agent nem cherry-pickel vakon; minden változást reviewz.
- Shared API módosítás előbb merge-elendő, csak utána épülhet rá másik agent.
- Minden alügynök jelentse:
  - módosított fájlok;
  - authority-változás;
  - lifecycle-változás;
  - tesztek;
  - nyitott kockázat.

---

## 20. Rollback és biztonság

Minden phase tartalmazzon:

- compatibility facade-ot;
- explicit cutover pontot;
- no-dual-authority tesztet;
- rollback leírást;
- config/persistence migrationt csak verziózott sémával;
- régi adat törlését csak sikeres cutover és legalább egy release után.

A foundation fázisokban nincs adatformatum-változás.

---

## 21. Definition of Done

Az IceSMP 1.0 architekturális alapja akkor tekinthető teljesnek, ha:

1. az `IceSMP` entrypoint csak az `IceSmpRuntime`-ot kezeli;
2. az `IceSMPCore` megszűnik vagy minimális compatibility facade marad;
3. nincs reflection a Core privát mezőire;
4. nincs új statikus service-locator híd;
5. minden listener/task/store/session state module-owned;
6. a module graph gépileg validált;
7. a UI csak query/view API-kat fogyaszt;
8. command és GUI közös use case-t használ;
9. az Event Platformban minden event katalógusból érhető el;
10. új event nem igényel Core/HUD/command/restart/shutdown switch módosítást;
11. a Corruption natív event module + streaming terrain resource;
12. a QuestManager és nagy manager facade-k fel vannak bontva;
13. a legacy `managers/` és `listeners/` csomag már nem kap új feature-t;
14. minden durable state-nek dokumentált canonical authorityja van;
15. build, consistency, documentation gate, architecture gate és Folia acceptance zöld;
16. az `AGENTS.md` a célarchitektúrát és a tényleges workflow-t pontosan írja, a `CLAUDE.md` nem versengő authority;
17. nincs architektúra- vagy feature-branding `V2`/`2.0` néven, kivéve valódi technikai verziókat;
18. az új domain/application kód nem függ YAML-, JDBC-, ORM- vagy backend-specifikus típustól;
19. a config, authored content, durable state és streaming journal authorityk külön vannak választva;
20. a spell/class balance typed, versioned snapshoton keresztül elérhető;
21. a későbbi database backend module bootstrapnál adapterként beköthető, gameplay újraírás és dual-write nélkül.

---

## 22. Első implementációs mérföldkő – pontos scope

Az első munkamenet **nem** hajtja végre a teljes reworköt.

A scope kizárólag:

### DOC-00

- minden tracked Markdown teljes reviewja;
- `AGENTS.md` teljes frissítése/újraírása és `CLAUDE.md` szinkron;
- 1.0 foundation naming átvezetése;
- dokumentációs authority és mirror gate.

### AR-00

- baseline;
- architecture és data-authority inventory/gate;
- plan dokumentálása;
- branch foundation;
- közvetlen YAML/config-path/storage coupling feltérképezése;
- database-ready szabályok rögzítése backend implementáció nélkül.

### AR-01

- runtime/module lifecycle;
- LegacyCoreModule;
- változatlan startup/shutdown szemantika;
- tesztek.

Kifejezetten tilos az első mérföldkőben:

- eventmechanika átírása;
- manager package tömeges mozgatása;
- QuestManager felbontása;
- Corruption átírása;
- HUD redesign;
- config bridge kiváltása;
- player session cleanup átírása;
- persistence schema változtatása;
- database driver, ORM, SQL schema, connection pool vagy production database backend bevezetése;
- file/YAML authority lecserélése vagy shadow dual-write.

Az első milestone célja, hogy az új architektúrának legyen biztonságos gerince, amely mögött a teljes jelenlegi rendszer változatlanul fut.
