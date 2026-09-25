# IceSMP feature-katalógus

<!-- DOC-AUTHORITY: CANONICAL_HUMAN_GUIDE -->

Ez a guide a játék és a csapat számára látható **tartós capabilityket** foglalja össze. Nem changelog, nem teljes class/config/command inventory és nem future-plan. Nyitott vagy még csak tervezett elem a [`ROADMAP.md`](../ROADMAP.md)-ban szerepel.

## 1. Belépés és identitás

- saját onboarding és Prologue-flow;
- PlayerProfile-alapú tartós karakterállapot;
- class, specializáció, loadout, doctrine/mastery/capstone progression;
- személyes class artifact és spellbook/favorites UX;
- profile/menu felületek, amelyek projectiont mutatnak, nem külön authorityt.

## 2. Class és combat

A jelenlegi class/spec katalógus 13 classt és 35 specializációt fed le. A classok konkrét, egymástól eltérő gameplay loopokat használnak; nincs általános, minden mechanikát vezérlő DSL.

- resource, cooldown, mastery és doctrine progression;
- specenkénti transient combat state explicit lifecycle cleanup-pal;
- DARK specializációk közös gate/seal/recovery policyval;
- player-vezérelt, olvasható kockázat/jutalom mechanikák;
- PvE/PvP clampok és közös spell power pipeline;
- immutable UI projection a class/spec állapotról.

A végső balance és valós party/TTK érzés staging gate.

## 3. Faction, law és world identity

- négy fő faction és guest/menedék útvonal;
- membership, faction relation és territory/protection integráció;
- law/sin/bounty és faction-specifikus presentation;
- world/lore identitás a canonical Kódexhez kötve;
- rejtett narratív réteg csak ott publikus, ahol a játék ténylegesen felfedi.

## 4. Gazdaság

- több fizikai és account/projection szinten kezelt valuta;
- banki be- és kivételi útvonalak;
- market, exchange, shop és treasury rendszerek;
- durable-first műveletek, receipt/WAL/recovery ott, ahol kritikus;
- blueprint, material, crafted result, loot és item identity integráció;
- player-facing receipt csak a tényleges committed outcome-ról.

## 5. Professions, crafting és equipment

- profession level/XP és recipe learning;
- canonical recipe/content katalógus;
- blueprint és material source/sink metadata;
- item template/instance identity, rarity, affix és checksum szerződések;
- armor family, requirement és active-equipment authority;
- set/signature/masterwork/ascension jellegű, authored equipment capabilityk;
- build-aware, bounded loot personalization;
- resource-pack inventory és wearable presentation külön render-identitással.

A hosszú távú equipment/economy authoring contract supporting future-plan, nem automatikus live content.

## 6. Quest, NPC és narrative

- authored quest catalog és durable player progress;
- assignment, objective, reward és dialogue utak;
- FancyNpcs-alapú canonical NPC interakció;
- quest builder/admin tooling explicit permissionnel;
- lore, chronicle, campfire story és achievement integráció;
- faction-perspective storytelling a canonical lore anchorain belül;
- visszaolvasható narrative felületek ott, ahol implementáltak.

Az exact quest/NPC/dialogue lista generált inventory.

## 7. PvE creature platform

- authored creature template, level, rank és stat projection;
- common ability runtime telegraph/cooldown/recovery/interrupt szerződéssel;
- event/boss/Prologue spawn ugyanazon creature authorityn keresztül;
- bounded summon/add lifecycle;
- pet/minion combat és owner attribution;
- reward owner különválasztás generic/event/none útvonalra;
- daylight, context, resistance/weakness és authored counterplay.

## 8. Eventek és seasonök

A jelenlegi rendszer több managerből álló world-event runtime:

- world boss, invasion, blood moon, treasure, caravan, gathering/abundance, server challenge, escort, meteor, wild hunt és ambient jellegű események;
- safe placement és spawn guard;
- major-event admission/gate;
- contribution és settlement;
- restart/resume vagy domain-specifikus recovery;
- season modifier és campaign/Prologue integráció;
- Corruption saját terrain journal/recovery útvonallal.

Egységes natív Event Platform még future architecture.

## 9. Pets, minions és companions

- durable companion roster és runtime entity projection;
- capture/summon/release/stance életciklus;
- XP/level/evolution vagy class-specifikus roster state;
- owner combat target követés;
- no-loot/no-XP policy plugin-owned minionnál/petnél, ahol a canonical combat authority így definiálja;
- death, despawn, logout és spec switch cleanup.

## 10. Social és moderation

- party és közös gameplay projection;
- natív chat/prefix integráció;
- moderation, vanish, inspect/invsee és admin tooling;
- donation chest, leaderboard, parkour és közösségi célok;
- permission- és auditkapuk high-risk actionökhöz.

A teljes command/permission lista a generált repository inventoryban él.

## 11. HUD, GUI és client

- first-party HUD és class/mechanic state;
- profile, spellbook, skill/talent, profession, market, quest és admin felületek;
- MiniMessage-alapú player-facing copy;
- resource-pack tooltip style, item category accent és wearable assets;
- immutable resource-pack URL/hash, resend és client route;
- viewer-specifikus requirement presentation csak client projectionként.

## 12. Reliability és safety

- Folia entity/region/global scheduler ownership;
- stale callback fencing;
- command admission close/drain;
- persistent store coordinator és final checkpoint;
- atomic file replace, revision/CAS, quarantine és recovery több kritikus domainben;
- deterministic generators/auditok és dependency-light regressziók;
- fail-closed startup vagy operation ott, ahol authority/integrity nem bizonyítható.

## Readiness értelmezése

- **Implementált:** executable source/config/content létezik és automated gate fedi.
- **Gated:** automated gate mellett kötelező élő Folia/client/human acceptance hiányzik.
- **Planned:** csak a roadmap/foundation plan írja; nem tekinthető feature-nek.

Operátori részletek: [`ADMIN_GUIDE.md`](ADMIN_GUIDE.md).  
Játékos használat: [`PLAYER_GUIDE.md`](PLAYER_GUIDE.md).  
Világ- és content hookok: [`BUILDER_GUIDE.md`](BUILDER_GUIDE.md).
