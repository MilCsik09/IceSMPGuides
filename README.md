# IceSMP

<!-- DOC-AUTHORITY: REPOSITORY_ENTRY -->

Az IceSMP egyetlen Java 21 Paper/Folia pluginból, first-party resource packből, authored config/content rétegből és determinisztikus build/validation toolingból álló Minecraft SMP projekt.

## Aktuális executable snapshot

- repository: `MilCsik09/IceSMP`;
- elsődleges fejlesztési ág: `staging`;
- DOC-00 indulási commit: `121c3b9cccca15c3e828f2bd363e8ddb17016b73`;
- Minecraft/Paper API: `1.21.11`;
- Java toolchain: 21;
- Folia metadata: támogatott;
- plugin entrypoint: `hu.taliann.icesmp.IceSMP`;
- jelenlegi runtime: `IceSMP` közvetlenül birtokolja az `IceSMPCore` kompatibilitási runtime-ot, valamint a resource-pack, Prologue és transient-entity lifecycle elemeket.

A modular runtime, module graph, DataPlatform és natív Event Platform még célarchitektúra; nem része a jelenlegi executable állapotnak.

## Build

```bash
./gradlew build --console=plain --no-daemon
python3 scripts/check_consistency.py
python3 scripts/check_markdown_links.py --root .
python3 scripts/check_documentation.py
```

A teljes Gradle build a mérvadó. Részleges task vagy cache-es fordítás csak preflight.

## Dokumentációs authority

| Kérdés | Elsődleges forrás |
|---|---|
| Engineering és agent workflow | [`AGENTS.md`](AGENTS.md) |
| Dokumentumszerepek és mirror | [`docs/DOCUMENTATION_AUTHORITY.md`](docs/DOCUMENTATION_AUTHORITY.md) |
| Implementált architektúra | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| 1.0 célarchitektúra és migráció | [`docs/ARCHITECTURE_FOUNDATION_PLAN.md`](docs/ARCHITECTURE_FOUNDATION_PLAN.md) |
| Nyitott findingek/backlog | [`ROADMAP.md`](ROADMAP.md) |
| Content authoring | [`docs/CONTENT_AUTHORING.md`](docs/CONTENT_AUTHORING.md) |
| Narrative canon | [`docs/LORE.md`](docs/LORE.md) |
| Resource-pack authoring | [`resource-pack/README.md`](resource-pack/README.md) |

### Öt kanonikus human guide

- [`docs/FEATURES.md`](docs/FEATURES.md) — capability-katalógus;
- [`docs/LATEST_CHANGES.md`](docs/LATEST_CHANGES.md) — rövid változáslista;
- [`docs/PLAYER_GUIDE.md`](docs/PLAYER_GUIDE.md) — játékos útmutató;
- [`docs/BUILDER_GUIDE.md`](docs/BUILDER_GUIDE.md) — építő/content handoff;
- [`docs/ADMIN_GUIDE.md`](docs/ADMIN_GUIDE.md) — live-ops és recovery.

Az exact command-, permission-, class-, config-, quest-, item- és assetinventory gépi riportból származik. Ne használj régi dokumentumban szereplő kézi számlálót executable truthként.

## Repository-felépítés

```text
src/main/java/          plugin runtime és domain/application/adapters
src/main/resources/     packaged config, content, messages, datapack és metadata
src/regression/         dependency-light regressziós suite-ok
resource-pack/          kicsomagolt first-party pack forrás
scripts/                deterministic audit/generator/consistency tooling
docs/                   current, workflow, supporting, future és historical dokumentáció
.github/workflows/      CI, build, audit és publish kapuk
```

## Fejlesztési szabály röviden

- Ne commitolj közvetlenül `staging`re.
- Egy state-nek egy canonical authorityja legyen.
- Ne vezess be új Core reflectiont, statikus service locatort vagy hidden dual-write-ot.
- Folia owner-thread szabályt minden entity/location mutationnél tartsd be.
- Java/gameplay változás ugyanabban a PR-ben frissíti a releváns current-state dokumentációt.
- Nyitott finding kizárólag a `ROADMAP.md`-ba kerül.
- Új architektúra- vagy feature-branding nem használhat `V2`, `v2`, `2.0` vagy `next-gen` megnevezést; valódi technikai verzió és kompatibilitási ID kivétel lehet.

A részletes szabályokért mindig az `AGENTS.md` az authority.
