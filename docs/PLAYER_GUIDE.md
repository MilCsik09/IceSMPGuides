# IceSMP játékos útmutató

<!-- DOC-AUTHORITY: CANONICAL_HUMAN_GUIDE -->

Ez a guide a normál játékos számára szükséges, publikus működést írja le. Nem fed fel hidden developer artifactot, staff recoveryt vagy szándékosan rejtett narrative mechanikát.

## 1. Első belépés

Belépés után kövesd az onboarding/profile felületet. A szerver a karaktered tartós állapotát PlayerProfile-ban kezeli; a GUI, HUD, class artifact és item lore ennek megjelenítése vagy rebuildable mirrorja.

A Prologue/campaign aktuális állapota meghatározhatja, mely világterületek, questek vagy capabilityk érhetők el. A pontos release-kapu mindig a szerveren látható állapot és staff announcement.

## 2. Profil és menük

A két biztos belépő:

- `/profile` — karakter, progression és kapcsolódó felületek;
- `/menu` — központi navigáció, ahol engedélyezett.

A commandok exact listája változhat, ezért a beépített helpet és GUI-t használd; ne régi screenshotból vagy kézi listából indulj ki.

## 3. Class, specializáció és spell

- Válassz classt a szerver által felkínált profile/class felületen.
- A class közös progressiont, resource-t és saját combat loopot ad.
- A specializáció loadout-specifikus doctrine/mastery/capstone állapotot tarthat.
- A második loadout csak a tényleges unlock és safety gate után használható.
- Combat, közeli ellenség vagy pending operation blokkolhat váltást.
- A váltás nem heal, resource- vagy cooldown-reset kerülőút.
- A személyes class artifact a spellbook/catalyst felülete; idegen vagy duplikált fizikai példány nem ad jogosultságot.
- A spellbook/favorites aktív képességkészletet és leírást mutat; a szerver canonical unlock/provenance állapota dönt.

A HUD rövid állapotfokozatokat és használható combat információt mutat; nem tartós authority.

## 4. Faction és világ

A világ négy fő faction és egy guest/menedék belépőút köré szerveződik. A faction meghatározhat:

- kezdőhelyet és városi szolgáltatásokat;
- relationt, territoryt és protectiont;
- law/sin/bounty következményeket;
- bizonyos quest-, profession- vagy narrative perspektívát;
- vizuális és HUD presentationt.

A kánon forrása a Kódex. Pletyka, campfire story vagy faction-nézőpont szándékosan torzíthat, de nem írhatja át a canonical anchorokat.

## 5. Pénz, bank és kereskedelem

- Egyes valuták account/projection, mások fizikai item formában is megjelenhetnek.
- Banki ki- és befizetés csak a támogatott útvonalon történik.
- Market/exchange/shop műveletnél a receipt a tényleges committed eredményt írja le.
- Ne tekints item lore-t vagy GUI-previewt wallet authoritynak.
- Pending delivery/reward esetén szabadíts fel inventoryhelyet, és kövesd a rendszer recovery útmutatását.
- Kézi file/PDC/item-másolás nem érvényes pénzügyi művelet.

## 6. Profession és crafting

- A profession saját XP/level és recipe unlock állapotot használ.
- Blueprint megtaníthat receptet, de a fizikai blueprint nem válik recipe authorityvá.
- A recipe megkövetelhet professiont, szintet, materialt, állomást vagy world feltételt.
- Crafted gear előnye a targetálható recept és quality-control; dropped gear encounter/source identitást adhat.
- Az armor family és class/spec requirementet a server ellenőrzi. A tooltip zöld/piros viewer projectionje tájékoztató, a canonical item nem változik játékosonként.
- Masterwork, set, Signature vagy Ascension csak authored itemnél és a tényleges runtime consumerrel számít.

## 7. Equipment és itemek

Az IceSMP item lehet template/instance alapú, rarityvel, affixszel, source taggel és checksum/identity szerződéssel.

- Ne próbáld vanilla craftinggal megkerülni a canonical recipe útvonalat.
- Wrong-family, invalid, suppressed vagy duplikált physical identity nem biztos, hogy active equipmentnek számít.
- A tooltip rövid mechanikai és requirement információt mutat; a display/lore nem írja felül a szerver state-et.
- A resource pack inventory és worn megjelenése két külön renderidentitás lehet.

## 8. Questek és történetek

- Questet NPC, board, profile/menu vagy scripted event adhat.
- Objective csak a ténylegesen elfogadott/aktív questhez számít.
- Rewardnál inventoryhely vagy más gate szükséges lehet; durable pending reward nem vész el pusztán a full inventory miatt.
- A `/lore` vagy a kapcsolódó narrative felület a már hallott/megismert tartalmat mutathatja.
- Campfire story és faction perspective nem feltétlenül objektív igazság.
- Quest marker csak ott jelenik meg, ahol a runtime valóban támogatja; a guide nem ígér automatikus navigációt.

## 9. PvE, event és boss

- A creature level/rank/ability authored profile-ból és world contextből épül.
- Telegraph, cast time, recovery és counterplay figyelhető; ne csak a névből következtess.
- World eventhez lehet admission, safe placement, contribution és timeout.
- World-boss reward contribution- vagy settlement-gated lehet; a kill önmagában nem garantál minden jutalmat.
- Event restart után folytatódhat vagy recovery állapotba kerülhet; kövesd az event status/announcement felületet.
- Corruption és más terrain event látható world state-et módosíthat, de saját rollback/recovery útvonallal rendelkezik.

## 10. Pet, minion és companion

- A durable roster és a live entity két külön dolog.
- Capture/summon/release/stance csak a támogatott command/GUI útvonalon történjen.
- A pet fejlődése és aktuális HP/state a domain szabályát követi; ne a mob névtagjából következtess.
- Saját pet/minion megölése nem legitim lootfarm.
- Combat targetet az owner interaction és a companion AI együtt választja.
- Logout, death, spec switch vagy plugin cleanup eltávolíthatja a live entityt a durable roster elvesztése nélkül.

## 11. HUD, actionbar és resource pack

A first-party pack szükséges a teljes HUD-, tooltip-, icon- és equipment presentationhöz. Ha nem töltődött be:

1. ellenőrizd, hogy a kliens engedélyezi a server resource packet;
2. használd a támogatott resend/reconnect útvonalat;
3. jelezd a staffnak a kliensverziót és a hiba pontos képét;
4. missing/magenta assetnél ne próbáld item újragenerálással javítani a problémát.

A fallback copy olvasható kell maradjon egyedi font nélkül is.

## 12. Biztonságos hibajelentés

Hasznos report:

- pontos idő és world/location;
- reprodukciós lépések;
- command/GUI/event neve;
- képernyőkép és teljes releváns console/player üzenet;
- relog/restart után megmarad-e;
- item esetén canonical név és acquisition path;
- pet/event esetén owner/instance context.

Ne küldj érzékeny admin audit ID-t vagy profile/store fájlt nyilvános csatornába.

## 13. Mi nem player-facing authority

- GitHub branch/commit vagy régi PR-handoff;
- developer artifact neve és owner-only interaction;
- raw enum, UUID, internal template ID vagy file path;
- staff recovery command;
- future architecture terv;
- kézzel szerkesztett PDC/YAML vagy másolt ItemStack.

Feature-áttekintés: [`FEATURES.md`](FEATURES.md).  
Kánon: [`LORE.md`](LORE.md).  
Legutóbbi változások: [`LATEST_CHANGES.md`](LATEST_CHANGES.md).
