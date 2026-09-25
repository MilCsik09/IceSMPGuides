# IceSMP dokumentációs authority

<!-- DOC-AUTHORITY: CANONICAL_DOCUMENT_INDEX -->

**Review baseline:** `staging` @ `121c3b9cccca15c3e828f2bd363e8ddb17016b73`  
**Review dátuma:** 2026. szeptember 25.  
**Gépi authority:** [`config/documentation-authority.json`](../config/documentation-authority.json)  
**Deterministic gate:** `python3 scripts/check_documentation.py`

Ez a fájl a dokumentáció szerepeit és az authority-ütközések feloldási sorrendjét írja le. Nem tartalmaz kézzel karbantartott class-, command-, permission-, config- vagy fájlszámlálót; az exact inventory a gate által generált `build/reports/architecture/` riportban él.

## Authority-sorrend

1. **Tényleges forráskód, config/content, build és CI** — executable truth.
2. **`AGENTS.md`** — fejlesztési, agent-, branch-, lifecycle-, safety- és dokumentációs workflow.
3. **`docs/ARCHITECTURE.md`** — implementált current state.
4. **`docs/ARCHITECTURE_FOUNDATION_PLAN.md`** — még nem implementált cél és migráció.
5. **`ROADMAP.md`** — egyetlen nyitott finding/backlog authority.
6. **Kanonikus human guide-ok** — felhasználási nézőpont, belső spekuláció nélkül.
7. **Supporting reference-ek** — szűk technikai vagy authoring szerződések.
8. **Generated reportok és historical dokumentumok** — bizonyíték, nem current authority.

Ha dokumentum és executable truth eltér, a dokumentum hibás. Ha current-state és future-plan eltér, a current-state leírása nem írható át úgy, mintha a terv már elkészült volna.

## Szerepek

| Szerep | Jelentés |
|---|---|
| `CANONICAL_CURRENT_STATE` | Ténylegesen implementált rendszer vagy kánon elsődleges leírása. |
| `CANONICAL_WORKFLOW` | Kötelező fejlesztési, authoring vagy üzemeltetési workflow. |
| `CANONICAL_HUMAN_GUIDE` | Player/builder/admin/team nézőpontú, tartós használati útmutató. |
| `SUPPORTING_REFERENCE` | Szűk, nem versengő technikai vagy tartalmi referencia. |
| `FUTURE_PLAN` | Nem implementált cél, terv vagy nyitott program. |
| `GENERATED_REPORT` | Gépből származó, újragenerálható leltár/evidence. |
| `HISTORICAL` | Lezárt változtatás vagy release-evidence; nem current authority. |
| `DUPLICATE_OR_STALE` | Megtartott átirányító vagy megszüntetendő párhuzamos authority. |

## Kanonikus current-state és workflow

- `README.md` — repository belépő és authority index;
- `AGENTS.md` — elsődleges engineering/workflow authority;
- `CLAUDE.md` — kompatibilitási shim az `AGENTS.md`-hez;
- `docs/ARCHITECTURE.md` — current implementation;
- `docs/CONTENT_AUTHORING.md` — content authoring workflow;
- `docs/LORE.md` — lore-kánon;
- `resource-pack/README.md` — resource-pack build/render contract;
- `docs/DOCUMENTATION_AUTHORITY.md` — dokumentációs szerepek és mirror-policy.

## Öt kanonikus human guide

1. `docs/FEATURES.md`
2. `docs/LATEST_CHANGES.md`
3. `docs/PLAYER_GUIDE.md`
4. `docs/BUILDER_GUIDE.md`
5. `docs/ADMIN_GUIDE.md`

Az exact command-, permission-, quest-, item-, class-, config- és assetlisták nem ezekben élnek kézzel, hanem a repository inventoryban vagy a canonical config/content forrásban.

## Future-plan authority

- `ROADMAP.md` — nyitott findingek és prioritás;
- `docs/ARCHITECTURE_FOUNDATION_PLAN.md` — az 1.0 architekturális program normatív terve;
- `docs/development/LONG_TERM_EQUIPMENT_ECONOMY.md` — domain-specifikus jövőbeli authoring contract.

## Teljes DOC-00 besorolás

| Fájl | Szerep | Döntés |
|---|---|---|
| `AGENTS.md` | CANONICAL_WORKFLOW | Teljesen újraírva; jelenlegi kompatibilitási szabály és célarchitektúra külön. |
| `CLAUDE.md` | SUPPORTING_REFERENCE | Rövid shim; nincs önálló szabályrendszer. |
| `README.md` | CANONICAL_CURRENT_STATE | Repository- és dokumentációs belépő. |
| `ROADMAP.md` | FUTURE_PLAN | Egyetlen nyitott finding/backlog authority. |
| `docs/ADMIN_GUIDE.md` | CANONICAL_HUMAN_GUIDE | Tartós live-ops/recovery útmutató; exact leltár generált. |
| `docs/ARCHITECTURE.md` | CANONICAL_CURRENT_STATE | Kizárólag implementált állapot; foundation elemek nincsenek késznek állítva. |
| `docs/ARCHITECTURE_FOUNDATION_PLAN.md` | FUTURE_PLAN | Normatív 1.0 cél- és migrációs terv. |
| `docs/BUILDER_GUIDE.md` | CANONICAL_HUMAN_GUIDE | Fizikai világkötések és content handoff. |
| `docs/CONTENT_AUTHORING.md` | CANONICAL_WORKFLOW | Canonical authored-content workflow. |
| `docs/DOCUMENTATION_AUTHORITY.md` | CANONICAL_WORKFLOW | Dokumentumszerepek, mirror és update contract. |
| `docs/FACTION_REWORK.md` | DUPLICATE_OR_STALE | Átirányító; current authority a kanonikus guide-okban és architecture-ben. |
| `docs/FEATURES.md` | CANONICAL_HUMAN_GUIDE | Tartós capability-katalógus, release napló nélkül. |
| `docs/LATEST_CHANGES.md` | CANONICAL_HUMAN_GUIDE | Rövid, dátumozott változáslista; nem PR-handoff. |
| `docs/LORE.md` | CANONICAL_CURRENT_STATE | Narrative canon. |
| `docs/LORE_REFERENCE.md` | SUPPORTING_REFERENCE | Lore↔runtime megfeleltetés; nem kánonpótló. |
| `docs/PLAYER_GUIDE.md` | CANONICAL_HUMAN_GUIDE | Publikus player journey; rejtett staff/dev mechanika nélkül. |
| `docs/PLAYER_GUIDE_EQUIPMENT_ECONOMY.md` | DUPLICATE_OR_STALE | Átirányító a fő player guide-ra és authoring contractra. |
| `docs/PROLOGUE.md` | SUPPORTING_REFERENCE | Rövid technikai/current-state boundary; roadmap nem itt él. |
| `docs/QUESTS.md` | SUPPORTING_REFERENCE | Quest authoring contract; exact katalógus generált. |
| `docs/RESOURCE_PACK_CMD.md` | SUPPORTING_REFERENCE | Item-model/equipment/HUD szerződés; exact assetlista generált. |
| `docs/TEASER.md` | SUPPORTING_REFERENCE | Kampány- és vizuális reference; nem launch schedule authority. |
| `docs/TEXTURE_WORKSHEET.md` | GENERATED_REPORT | Reprodukálható authoring record; számlálók nem canonicalak. |
| `docs/admin/CLASS_SPEC_REWORK_RUNBOOK.md` | SUPPORTING_REFERENCE | Class/spec recovery runbook; legacy technikai ID-k kompatibilitási kivételek. |
| `docs/development/AUTHORED_PVE_CREATURE_MODEL.md` | SUPPORTING_REFERENCE | Authored PvE authority boundary. |
| `docs/development/CLASS_SPEC_REWORK_1_21_11_TO_26_2.md` | HISTORICAL | Valódi verziók közti port contract. |
| `docs/development/CLASS_SPEC_REWORK_ARCHITECTURE.md` | SUPPORTING_REFERENCE | Class/spec current implementation részletes reference. |
| `docs/development/CLASS_SPEC_REWORK_GAP_ANALYSIS.md` | HISTORICAL | Lezárt vertical-slice evidence; nyitott pontok a roadmapban. |
| `docs/development/CLASS_SPEC_REWORK_MIGRATION.md` | HISTORICAL | Greenfield migration-döntés rövid recordja. |
| `docs/development/CLASS_SPEC_REWORK_TEST_PLAN.md` | SUPPORTING_REFERENCE | Regressziós task pointer. |
| `docs/development/LONG_TERM_EQUIPMENT_ECONOMY.md` | FUTURE_PLAN | Tartós future authoring contract. |
| `docs/development/SPECIAL_ITEM_TOOLTIP_PROFILES.md` | SUPPORTING_REFERENCE | Belső tooltip acceptance; nem mirrorolandó. |
| `docs/development/equipment-rp2-art-bible.md` | SUPPORTING_REFERENCE | Technikai asset-ID kompatibilitási reference. |
| `docs/development/equipment-rp2-asset-foundation.md` | HISTORICAL | Asset foundation closure/evidence; exact leltár gépi. |
| `docs/development/equipment-rp2-imagegen-prompts.md` | GENERATED_REPORT | Reprodukálható art-source record. |
| `docs/development/player-facing-messaging-integrity-hardening.md` | HISTORICAL | Lezárt hardening PR-evidence. |
| `resource-pack/README.md` | CANONICAL_WORKFLOW | Resource-pack authoring, validation és publish contract. |

## Megszüntetett authority-ütközések

- A régi `AGENTS.md` `master`-baseline-ja, kézi fájlszámai és az `IceSMPCore` jövőbeli bővítését előíró mintája megszűnt.
- A `CLAUDE.md` nem tart fenn külön build-, architecture- vagy delegálási szabályokat.
- A current architecture nem állít jövőbeli runtime/module/DataPlatform elemet implementáltnak.
- A `ROADMAP.md` nem changelog; lezárt PR-handoff nem marad benne authorityként.
- A human guide-ok nem tartanak fenn teljes kézi command/permission/config/class/file leltárt.
- A faction, prologue és equipment/economy párhuzamos guide-ok nem versenyeznek a kanonikus guide-okkal.
- Rejtett developer artifactok nem kerülnek public guide-ba vagy mirrorba.

## Naming és kompatibilitási kivételek

Tiltott az új architektúra- vagy feature-brandingként használt `V2`, `v2`, `2.0` és `next-gen`.

Megőrizhető, ha valódi kompatibilitási azonosító:

- Minecraft/Paper/API/release verzió, például `1.21.11` vagy `26.2`;
- schema/protocol/migration verzió;
- már létező Java class, config key, Gradle task, workflow, fájl- vagy assetazonosító, például `GameplayV2ClassPolicy` vagy `equipment-rp2-*`;
- történeti dokumentumban idézett lezárt programnév.

A kivétel nem engedély új, hasonló nevű elem létrehozására. A gépi manifest path- és regex-szinten dokumentálja a megengedett legacy tokeneket.

## Mirror-policy

Az `IceSMPGuides` nem teljes source mirror. Csak a `config/documentation-authority.json` fájlban `mirror: true` jelölésű, team/public dokumentumokat tartja meg azonos relatív útvonalon és byte-pontosan.

Nem mirrorolandó:

- `AGENTS.md`, `CLAUDE.md` és belső agent workflow;
- rejtett developer-artifact vagy security-sensitive acceptance;
- generated evidence és PR-specifikus history;
- nagy, belső implementation handoff, ha a public/team guide nem igényli.

A mirror drift gate a forrás- és mirror-fájl SHA-256 tartalmát hasonlítja össze.

## Módosítási workflow

1. Határozd meg a dokumentum szerepét.
2. Ellenőrizd az állítást a tényleges forrás/config/content/build/CI alapján.
3. Current-state változás ugyanabban a PR-ben frissíti az architecture/human guide authorityt.
4. Nyitott finding kizárólag a `ROADMAP.md`-ba kerül.
5. Exact inventoryt a generator frissít, nem kézi szöveg.
6. Workflow-szabály változás együtt módosítja az `AGENTS.md`/`CLAUDE.md` párost.
7. Mirrorolt fájl ugyanabban a munkacsomagban byte-pontosan szinkronizálódik.
8. Futtasd:

```bash
python3 scripts/check_documentation.py
python3 scripts/check_markdown_links.py --root .
python3 scripts/check_consistency.py
./gradlew build --console=plain --no-daemon
```

A `build/reports/architecture/markdown-review.md` és `.json` determinisztikus, nem kanonikus riport; commitolni nem kell.
