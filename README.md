# IceSMP — Játékos Kézikönyv 📖

Üdv a fagyott királyságok földjén! Ez a kézikönyv **mindent** elmagyaráz a szerverről, úgy,
hogy bárki megértse — még akkor is, ha most ülsz le először Minecraftozni. Minden rendszernek
külön oldala van, hogy könnyen megtaláld, amit keresel.

> **Pár szó, amit jó tudni előre (ne ijedj meg tőlük):**
> - **XP / tapasztalat:** a zöld csík a képernyő alján, meg a melletti szám (a szinted). Sok
>   varázslat ebből „fizet".
> - **Éhség:** a sonkacomb 🍗 ikonok. Néhány varázslat ebből „fizet". 1 sonkacomb = 2 éhségpont.
> - **Cooldown / várakozási idő:** miután használtál egy képességet, várnod kell egy kicsit,
>   mire újra tudod használni.
> - **Mob:** szörny vagy állat (zombi, csontváz, tehén...).
> - **Blokk:** egy kocka a világban (1 blokk ≈ 1 méter).

> 🖱️ **Tipp:** írd be a **`/menu`** parancsot, és egy kattintós főmenü nyílik meg, ahonnan
> minden rendszer egy gombnyomásra elérhető — nem kell parancsokat gépelned!

> **Jelölések:** ✅ = kész és kipróbálható • 🚧 = részben kész • ⏳ = még nincs kész •
> 🔜 = **új, még NEM él** (a következő plugin-frissítéssel jön).
> A számok (szintek, %-ok, idők) a szerver beállításától függhetnek — itt az alapértékek vannak.

> 🧪 **Tesztelőknek:** ez a kézikönyv már a **készülő frissítést** is leírja, hogy előre lásd, mi
> jön. A **🔜 jelölésű** részek a szerveren **MÉG NEM aktívak** — minden más a **jelenlegi
> működés**. A készülő újdonságok összefoglalója: [🔜 Mi új a következő frissítésben](#-mi-új-a-következő-frissítésben-még-nem-él).

---

## 📚 Tartalom

1. [Kezdő lépések](01-kezdes.md) — mit csinálj az első 10 percben
2. [Frakciók](02-frakciok.md) — a négy oldal és a bónuszaik
3. [Valuta és gazdaság](03-valuta-gazdasag.md) — pénz, bank, piac, árfolyam
4. [Kasztok](04-kasztok.md) — a 13 hős-típus, szintezés, Lélekkapocs, Erő-csík
5. [Képességek (varázslatok)](05-kepessegek.md) — **minden** spell: mit tud, mennyibe kerül
6. [Specializációk](06-specializaciok.md) — a kaszt „kiteljesedése" 25. szinttől
7. [Talentek (talent-fa)](07-talentek.md) — passzív erősítések pontokból
8. [Szakmák](08-szakmak.md) — **minden** szakma: mit ad XP-t és mennyit
9. [Relikviák és rituálék](09-relikviak.md) — legendás tárgyak
10. [Világesemények](10-vilagesemenyek.md) — vérhold, világboss, karaván, meteor, kincs, kihívás…
11. [Királyság, raid és háború](11-raid-haboru.md) — királyok, ostrom, reputáció
12. [Küldetések](12-kuldetesek.md) — mit hol vegyél fel, mit kapsz
13. [Frakcióterületek és saját birtok](13-teruletek.md) — fővárosok, claim, építésvédelem
14. [Parancsok listája](14-parancsok.md) — minden parancs egy helyen
15. [Party (csapat)](15-csapat.md) — csapatalakítás, közös XP, party-HUD

---

## 🔜 Mi új a következő frissítésben (még nem él)

Ezek **dokumentálva vannak**, de a szerveren **MÉG NEM aktívak** — a tesztelők előre láthatják, mi
jön a következő plugin-frissítéssel. (Ami itt nincs felsorolva, az a **jelenlegi működés**.)

**Látvány / effektek**
- 🔜 **Spell-VFX** — formázott spell-effektek (sugár/gyűrű/hélix/kúp) + spec-témájú színek → [Képességek](05-kepessegek.md)
- 🔜 **Claim-fényfal** a `/claim show`-nál (izzó, per-nézős határfal) → [Területek](13-teruletek.md)
- 🔜 **3D crate-feltárás** a láda fölött (pörgő nyeremény-ikon) → [Parancsok](14-parancsok.md)
- 🔜 **Aurora fény-fátyol** + **boss-AoE padló-telegraph** → [Világesemények](10-vilagesemenyek.md)

**Szakmák / craft**
- 🔜 **Craft-korlátok bővítése** — 5 új szakma-kapu (bányász, favágó, alkimista, enchanter, séf) → [Szakmák](08-szakmak.md)

**Felület / kényelem**
- 🔜 **AFK-rendszer** — `/afk`, AFK-zóna jutalom + anti-farm → [Parancsok](14-parancsok.md)
- 🔜 **Dinamikus HUD + natív tablista** — harc-fókusz, forgó infósor, célpont-sor, relációs háború-színek → [Kezdés](01-kezdes.md)
- 🔜 **Lebegő sebzés-számok** (alapból csak a saját sebzésed) → [Kezdés](01-kezdes.md)

> A fenti listán túl a kézikönyv több apró pontosítást is tartalmaz. Kétség esetén: **ami a jelenleg
> élő szerveren nem így működik, az 🔜 (hamarosan).**

## 📜 Világ és történet (Lore)

A szerver kanonikus háttértörténete — a Teremtéstől a Káoszkorig, a négy frakció eredete, a
fővárosok, és a hozzájuk illő egyedi tárgyak katalógusa.

- [**Az Élet Fája és a Káosz Kora**](lore/LORE.md) — a teljes világ-történet + frakció-lore

## 💡 Fejlesztési ötletek (brainstorm)

> **Figyelem:** ez **nem** játékos-tartalom, hanem a fejlesztés ötlettára — *nincs elköteleződés*,
> nem minden valósul meg. Ideiglenesen került ide, betekintésre.

- [**Ötlettár — index**](ideas/README.md) — 356 kidolgozott ötlet, 11 kategóriában

| Kategória | Miről szól |
|---|---|
| [A) Polish](ideas/A-polish.md) | GUI/HUD/spell-QoL, kényelmi funkciók, effektek |
| [B) Új mechanika](ideas/B-mechanika.md) | Progresszió, céhek, dungeonök, claim/territórium |
| [C) Admin / infra](ideas/C-infra.md) | Balansz-adat, moderáció, üzemeltetés |
| [D) Világ, hangulat](ideas/D-vilag.md) | Immerzió, fővárosi élet, közösségi események |
| [E) Kaszt és spec](ideas/E-kaszt.md) | Kaszt-identitás, spec-szinergia, signature-pillanatok |
| [F) Gazdaság](ideas/F-gazdasag.md) | Árfolyam, adó, szerződések, luxus-sinkek |
| [G) PvP, háború](ideas/G-pvp.md) | Raid-variánsok, rivalizálás, taktikai réteg |
| [H) PvE, végjáték](ideas/H-pve.md) | Esemény-típusok, boss-mélyítés, ko-op |
| [I) Szakmák](ideas/I-szakmak.md) | Mestermű, érc-események, recept-láncok |
| [J) Questek, story](ideas/J-quest.md) | Objektíva-típusok, döntés-flagek, NPC-emlékezet |
| [K) Lore-integráció](ideas/K-lore.md) | A lore beépítése: itemek, Nether-zóna, Suttogók, feketepiac |

---

*Jó kalandozást! ❄️ Ha valamit nem értesz, kérdezz egy admintól vagy egy tapasztaltabb játékostól.*
