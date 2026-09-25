# IceSMP admin és live-ops guide

<!-- DOC-AUTHORITY: CANONICAL_HUMAN_GUIDE -->

Ez a guide a tartós üzemeltetési, recovery és staging eljárásokat írja le. Nem teljes command/permission/config inventory; az exact surface a repository generált riportjából és a tényleges regisztrációból származik.

## 1. Indítás előtti ellenőrzés

- Java 21;
- megfelelő Paper/Folia `1.21.11` runtime;
- kötelező dependencyk, különösen FancyNpcs;
- writable plugin data könyvtár;
- config/content/message schema és authority validáció;
- resource-pack URL/hash elérhetőség;
- backup a durable store-okról és journalokról;
- nincs félbehagyott deploy vagy kézzel módosított JAR/config keverék.

A production runtime nem tölt le automatikusan plugint. A Gradle run task provisioning csak fejlesztői környezet.

## 2. Startup sorrend és health

A current entrypoint sorrendje: resource-pack listener, transient runtime, Core enable, Prologue install, command open, resend és runtime probe-ok.

Startup után ellenőrizd:

- nincs fail-closed dependency/preflight hiba;
- config/content validatorok zöldek;
- persistent store recovery nem jelzett quarantine/reconciliation állapotot;
- command admission nyitott;
- resource pack current hash-sel küldhető;
- Prologue/event restart state konzisztens;
- runtime probe-ok nem jeleznek authority vagy shutdown szivárgást.

## 3. Command és permission truth

A root commandok programmatikusan regisztrált Paper commandok. Ne tarts kézi listát ebből a guide-ból production authorityként.

Használd:

```bash
python3 scripts/generate_repository_inventory.py --root . --output build/repository-inventory --mode report
python3 scripts/check_documentation_coverage.py --root . --inventory build/repository-inventory/repository-inventory.json --output build/repository-inventory --mode report
```

High-risk commandnál kötelező:

- explicit permission;
- actor/target/context audit;
- confirmation vagy holder-bound generation, ha destructive;
- no raw UUID/internal enum player-facing output;
- durable-first commit;
- idempotens recovery;
- staff és normal-player projection különválasztása.

## 4. Reload policy

Általános in-process `/reload` nem támogatott recovery stratégia.

Config/content változásnál:

1. készíts backupot;
2. futtasd a schema/authority validatorokat;
3. only-supported reload command/API;
4. invalid candidate esetén az előző jó snapshot maradjon aktív;
5. scheduler/listener toggle csak hivatalos lifecycle handle-en;
6. figyeld a health/degraded state-et;
7. ha a domain nem reloadable, controlled restart.

Ne írj private Core fieldet, timestampet vagy scheduler-metódust reflectionnel.

## 5. Backup egységek

A backup ne egyetlen véletlen YAML legyen. Konzisztens egységként kezeld:

- PlayerProfile és section state;
- gazdasági wallet/store és WAL/receipt;
- respec/critical operation journal;
- event/Prologue restart state;
- Corruption/terrain journal és recovery evidence;
- authored/operator config snapshot;
- resource-pack manifest/properties;
- quarantine/recovery evidence.

Restore előtt állítsd le a write admissiont. Egy tranzakcióhoz tartozó store-okból ne állíts vissza csak egyet.

## 6. PlayerProfile és class/spec recovery

Normál player state, `REVIEW`, `QUARANTINED`, sealed loadout és reconciliation-required külön eset.

Általános eljárás:

1. állítsd meg az ismételt login/mutation próbát;
2. rögzíts owner UUID-t, evidence/audit ID-t és teljes backupot;
3. ellenőrizd, mely store/operation authority érintett;
4. használd a domain dokumentált recovery commandját;
5. ne másolj másik player profile-t és ne szerkeszd csak az egyik store-t;
6. reconnect és runtime rebuild;
7. pozitív és negatív kontroll;
8. evidence megőrzése.

A class artifact vagy ItemStack nem progression authority; elveszett fizikai mirror a canonical profile-ból építhető újra a támogatott útvonalon.

## 7. Gazdasági incidens

- Ne refundolj kézzel, amíg a receipt/WAL/operation állapotot nem ellenőrizted.
- Ismételt committed operation ID no-op kell legyen.
- Pending market/quest/item deliverynél előbb állapítsd meg, durable commit történt-e.
- Wallet, physical currency és GUI preview külön réteg lehet.
- File permission/ENOSPC hiba után preserve log/store, állítsd le admissiont és controlled recoveryt használj.

## 8. Event live-ops

Event indítás/leállítás előtt ellenőrizd:

- admission és más major event;
- world/chunk/placement;
- WorldGuard/claim és spawn guard;
- participant és reward policy;
- instance/restart state;
- entity/task/terrain resource cleanup;
- timeout/abort/recovery.

Restart után használd az event status/diagnosztikai felületet. Ne indíts új példányt csak azért, mert a broadcast eltűnt; előbb ellenőrizd az authoritative state-et.

World-boss reward contribution-gated lehet. A generic boss-band loot és az event settlement külön authority.

## 9. Corruption és terrain recovery

- aktív event alatt ne szerkeszd kézzel a journal által birtokolt területet;
- pause/abort/rollback előtt backup;
- ellenőrizd surface, water, cave, chunk-border és protected region esetet;
- restart után journal replay/recovery health;
- távoli chunk entity és effect cleanup;
- rollback után maradék block/entity/task audit;
- nagy terhelésnél bounded batch és server health figyelés.

A terrain journal nem generikus event state, és nem törölhető csak azért, mert az event lifecycle lezárultnak látszik.

## 10. Prologue

A Prologue külön runtime és campaign gate.

Live-ops előtt:

- canonical phase/state;
- encounter/objective/finale activity;
- pause/resume epoch;
- world anchorok;
- participant és reward policy;
- Nether/portal unlock gate;
- restart/recovery evidence.

A Prologue nem fix naptári hét. A továbblépés boss/finale és release readiness döntéshez kötött.

## 11. Resource pack

- a plugin immutable hash-es URL-t használ;
- `latest.zip` emberi alias, nem runtime authority;
- validate-only és preflight nem production publish;
- publish előtt DNS/custom domain, SHA-1 és bucket hozzáférés;
- hiányzó asset vagy invalid JSON/PNG/UV publikálási hiba;
- resend után kliens elfogadás és hash ellenőrzés;
- rollback korábbi immutable objektumra.

Tooltip/wearable visual change valós kliens staginget igényel. A validator nem bizonyítja önmagában az olvashatóságot.

## 12. Invsee és high-risk moderation

A tényleges command parser és permission a source authority. Régi `read|edit <main|ender>` vagy más történeti syntax nem használható ellenőrzés nélkül.

Kötelező:

- actor/target audit;
- online/offline és inventory authority tisztázása;
- edit lease/generation, ha a runtime ezt használja;
- disconnect/timeout cleanup;
- ender/main inventory megkülönböztetés csak tényleges támogatásnál;
- destructive action confirmation és rollback/evidence.

## 13. Staging matrix

### Startup/shutdown

- clean start;
- dependency missing/incompatible negative control;
- config invalid negative control;
- command open/close/drain;
- clean disable;
- pending I/O disable;
- repeated enable/disable, ha a harness támogatja;
- no task/listener/static leak.

### Player lifecycle

- first join/profile ready;
- relog, kick, death és disable;
- pending callback melletti logout;
- class/spec switch combatban és safety radiusban;
- companion/entity cleanup;
- resource-pack resend.

### Persistence

- atomic replace;
- stale revision/CAS;
- repeated operation ID;
- partial WAL/receipt recovery;
- quarantine és explicit recovery;
- permission denied/ENOSPC;
- final checkpoint sorrend.

### Event/PvE

- manual/natural trigger;
- placement/admission negative control;
- region border;
- restart/resume;
- timeout/abort;
- contribution/reward;
- owner death/add cleanup;
- terrain rollback.

### Client/UI

- missing pack/fallback;
- tooltip readability több GUI scale-en;
- viewer requirement projection;
- HUD/actionbar burst;
- GUI holder-bound click routing;
- no raw enum/UUID/internal ID normál játékosnak.

## 14. Build és release gate

```bash
./gradlew build --console=plain --no-daemon
python3 scripts/check_consistency.py
python3 scripts/check_markdown_links.py --root .
python3 scripts/check_documentation.py
```

Futtasd a módosított domain célzott regression/audit taskjait és a valódi Folia/client staginget. A CI `steps=null` vagy runner/credit/infrastructure hiba nem code success és nem code failure bizonyíték; ténylegesen végrehajtott gate szükséges.

## 15. Incident report

Rögzítsd:

- exact commit/JAR/hash;
- runtime és dependency verzió;
- idő, actor, target, world/location;
- teljes releváns log és stack trace;
- domain authority/store/operation ID;
- reprodukció;
- restart/relog hatás;
- mitigation;
- preserved evidence és backup;
- follow-up roadmap ID.

Nyitott live gate-ek: [`ROADMAP.md`](../ROADMAP.md).  
Current architecture: [`ARCHITECTURE.md`](ARCHITECTURE.md).  
Class/spec részletes recovery reference: [`admin/CLASS_SPEC_REWORK_RUNBOOK.md`](admin/CLASS_SPEC_REWORK_RUNBOOK.md).
