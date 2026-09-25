# Legutóbbi változások

<!-- DOC-AUTHORITY: CANONICAL_HUMAN_GUIDE -->

Ez a fájl rövid, emberi változáslista. Nem PR-handoff, nem teljes tesztmátrix és nem architecture roadmap. Részletes diff/evidence a GitHub PR-ekben és CI artifactokban, nyitott tételek a [`ROADMAP.md`](../ROADMAP.md)-ban élnek.

## 2026. szeptember 25. — Dokumentációs és architecture authority

- elindult az IceSMP 1.0 Architecture Foundation külön branch-stackben;
- rögzítésre került a DOC-00 baseline: `staging` @ `121c3b9cccca15c3e828f2bd363e8ddb17016b73`;
- minden tracked Markdown fájl teljes authority-reviewt kapott;
- az `AGENTS.md` source-based engineering authority lett;
- a `CLAUDE.md` rövid, nem versengő shim;
- különvált a current architecture, a future foundation plan és az egyetlen roadmap authority;
- az öt human guide tartós, szerepalapú formát kapott;
- bevezetésre került a dokumentációs szerep-, naming-, local-link- és mirror-gate;
- a mirror scope explicit allowlist és byte-pontos tartalomellenőrzés lett;
- ebben a fázisban nem történt Java runtime vagy gameplay módosítás.

## 2026. szeptember 24. — Event restart és Corruption

A DOC-00 indulási `staging` commit az event restart/resume és underwater Corruption spread javításait tartalmazza. A részletes runtime behavior és fennmaradó élő tesztkapuk a source/current architecture és a roadmap alapján értelmezendők.

## Tartós változásnapló-szabály

Új bejegyzés csak akkor kerül ide, ha:

- merge-elt vagy reviewzható, felhasználó/csapat szempontból értelmezhető változást ír le;
- nem állít automated bizonyíték nélkül live readiness-t;
- nem másolja be a teljes PR leírást, commitlistát vagy exact inventoryt;
- külön jelzi a kötelező human/staging gate-et;
- nem használ tiltott párhuzamos-platform brandinget.

Régebbi, részletes release-handoffok a Git historyban és a kapcsolódó PR-ekben maradnak visszakereshetők.
