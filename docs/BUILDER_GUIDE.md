# IceSMP builder és content handoff guide

<!-- DOC-AUTHORITY: CANONICAL_HUMAN_GUIDE -->

Ez a guide az építők, world designerek és content authorok közti fizikai szerződést írja le. Nem tart fenn teljes quest/item/config leltárt, és nem talál ki koordinátát vagy NPC-t pusztán azért, mert egy rendszer támogatná.

## 1. Alapelv: build csak valós content requirementből

Minden build hookhoz legyen:

- owner domain/feature;
- canonical config/content rekord vagy jóváhagyott roadmap item;
- world és koordináta/anchor authority;
- interaction típusa;
- spawn/placement/protection feltétel;
- acceptance és rollback;
- felelős builder/content owner.

Ne fabricálj arénát, NPC-t, markerhelyet vagy dungeont olyan questhez, amelynek nincs fizikai world dependencyje.

## 2. Handoff sablon

```text
Feature/content ID:
Owner domain:
World:
Anchor/region:
Required footprint:
Required blocks/entities/NPCs:
Protection rules:
Spawn/interaction surface:
Failure/cleanup behavior:
Config/content source:
Staging acceptance:
Rollback:
```

A handoff ID stabil, a koordináta pedig explicit. Screenshot vagy chatüzenet nem canonical coordinate authority.

## 3. NPC és quest hook

NPC csak akkor canonical quest giver/target, ha:

- a content rekord és NPC binding ugyanarra a stabil identityre mutat;
- a runtime dependency (FancyNpcs) elérhető;
- interaction és dialogue route validált;
- builder world/anchor létezik és védett;
- quest objective nem feltételez nem implementált navigációt.

Quest marker nem automatikus waypoint. A builder guide nem állíthatja, hogy marker valahová vezet, ha a runtime csak státuszt vagy vizuális jelzést ad.

## 4. Faction városok és szolgáltatások

A négy fő faction és a guest/menedék út fizikai identitása különüljön el, de minden service hookot a tényleges manager/config/content igény vezéreljen.

Tipikus hookok:

- spawn és onboarding route;
- bank/market/exchange/profession állomás;
- faction/NPC/quest service;
- territory/protection boundary;
- campaign vagy event anchor;
- lore/chronicle/campfire tér;
- admin-only maintenance surface, ha valóban szükséges.

Ne építs rejtett faction/mechanic információt kötelező, nyilvános signage-dzsé, ha a design szerint discovery része.

## 5. Prologue és campaign

A Prologue fizikai buildjei csak a runtime által ténylegesen használt breach/objective/finale/portal hookokra épüljenek.

Kötelező ellenőrzés:

- world és chunk elérhető;
- region/protection engedi a scripted interactiont;
- restart után az anchor továbbra is azonosítható;
- pause/resume és failure cleanup nem hagy entityt vagy block state-et;
- objective/finale tér támogatja a tényleges player countot és counterplayt;
- Nether/portal unlock nem pusztán dekoráció, hanem explicit campaign statehez kötött.

A Prologue időtartama nem fix „első hét”; release gate alapján akár hosszabb open-beta szakasz lehet.

## 6. Event placement

Event build/anchor esetén külön kezeld:

- **admission:** futhat-e most az event;
- **placement:** hol lehet biztonságosan elhelyezni;
- **resource:** mit spawnol/módosít és ki takarítja;
- **settlement:** mikor tekinthető lezártnak;
- **recovery:** restart/timeout/abort után mi történik.

WorldGuard/personal claim, chunk, biome, surface/underwater, player proximity és más major event mind lehet placement gate. A builder ne hardcode-olja Java behavior helyett ezeket dekorációba.

## 7. Boss és encounter tér

Az authored PvE runtime birtokolja a creature stat/ability authorityt. A tér feladata a counterplay támogatása:

- olvasható telegraph és line-of-sight;
- mozgástér zone/projectile/slam ellen;
- add prioritás és spawnhely;
- region border/owner-scheduler szempont;
- bounded leash/cleanup;
- spectator/admin hozzáférés, ha approved;
- contribution és reward settlementhez nem akadályozó layout.

Ne építs második phase/combat logikát command blockból vagy datapackből a plugin mellett.

## 8. Corruption és terrain event

A terrain journal/recovery a runtime authority. Builder szempontból:

- jelöld az immutable/protected területeket;
- kerüld a recoveryt ellehetetlenítő, nem reprodukálható kézi módosítást aktív event közben;
- tesztelj surface, cave, water és chunk-border helyzetet;
- abort/rollback után vizsgáld a maradék blokk/entity state-et;
- nagy buildet csak backup és explicit event pause után módosíts.

## 9. Lore és vizuális identitás

A [`LORE.md`](LORE.md) canonical. A [`LORE_REFERENCE.md`](LORE_REFERENCE.md) runtime megfeleltetés, nem új kánon.

- Város, monument, szimbólum és environmental storytelling canonical anchorhoz kötődjön.
- Faction-perspective lehet torzított, de ne mondjon ellent kötelező világfaktumnak.
- Rejtett lore-t ne tegyél kötelező tutorial táblára.
- Teaser/kampány vizuál nem automatikusan production build requirement.

## 10. Resource-pack handoff

Új world/item/UI assethez add át:

- stable render ID;
- use surface (inventory, worn, entity, GUI, font, HUD);
- logical/runtime méret és UV;
- fallback;
- canonical source art és hash, ha authored pipeline része;
- generated/committed asset ownership;
- validator és client staging acceptance.

Inventory sprite nem használható automatikusan worn UV textúraként. Assetnév nem gameplay authority.

## 11. Builder acceptance

Minimum:

- clean server startup;
- world/chunk betöltés;
- interaction mindkét releváns oldalról;
- protection pozitív és negatív kontroll;
- relog/restart;
- multi-player és region-border próba;
- abort/timeout/cleanup;
- no orphan entity/block/task;
- screenshot/videó és pontos config/content ID;
- rollback leírás.

## 12. Mit ne tegyél

- ne találj ki koordinátát vagy NPC-t a hiányzó content helyett;
- ne másold a teljes generated quest/item listát a guide-ba;
- ne használj command blockot canonical state írására;
- ne építs gameplayt csak teaser vagy régi PR-szöveg alapján;
- ne módosíts aktív journal által birtokolt területet recovery terv nélkül;
- ne nevezd késznek a buildet élő server acceptance nélkül.

Nyitott build hookok és staging gate-ek: [`ROADMAP.md`](../ROADMAP.md).  
Quest authoring részlet: [`QUESTS.md`](QUESTS.md).  
Content workflow: [`CONTENT_AUTHORING.md`](CONTENT_AUTHORING.md).
